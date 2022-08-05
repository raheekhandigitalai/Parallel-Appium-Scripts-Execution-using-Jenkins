# Parallel Appium Execution using Jenkins

In this example we will look at how we can create an automated CICD pipeline to execute Appium Scripts against iOS and Devices against Digital.ai's Continuous Testing Platform.

```
def getJsonObjectFromResponseOutput(response, jsonObject) {
        def jsonContent = new groovy.json.JsonSlurperClassic().parseText(response)
            jsonObject = jsonContent.data[jsonObject].toString()
            return jsonObject
}

def getJsonObjectFromResponseOutputID(response, JsonObject) {
        def jsonContent = new groovy.json.JsonSlurper().parseText(response)
        jsonObject = jsonContent.id
        return jsonObject
}

def getJsonObjectFromResponseOutputName(response, JsonObject) {
        def jsonContent = new groovy.json.JsonSlurper().parseText(response)
        jsonObject = jsonContent.name
        return jsonObject
}

node {
    
    String CLOUD_URL = env.getProperty('CLOUD_URL')
    String ACCESS_KEY = env.getProperty('ACCESS_KEY')
    String JENKINS_BUILD_NUMBER = env.getProperty('BUILD_NUMBER')
    
    String triggerResponseCreateTestView = ""
    String triggerResponseGetTestViewResults = ""
    String triggerResponseDeleteTestView = ""
    String passedCount = ""
    String failedCount = ""
    String incompleteCount = ""
    String skippedCount = ""
    String totalCount = ""
    
    String testViewId = ""
    String TestViewName = ""
    
    def mvnHome
    
    // stage('Preparation') { 
    //     git 'https://github.com/raheekhandigitalai/ExperiBankDemoApplication.git'
    //     mvnHome = tool 'M3'
    // }
    // stage('Build') {
    //     withEnv(["MVN_HOME=$mvnHome"]) {
    //         if (isUnix()) {
    //             sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean install'
    //         } else {
    //             bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean install/)
    //         }
    //     }
    // }
    stage('Report Summary') {
        triggerResponseCreateTestView = sh(script: '''curl --location --request POST \'''' + CLOUD_URL + '''/reporter/api/testView\' --header \'Content-Type: application/json\' --header \'Authorization: Bearer ''' + ACCESS_KEY + '''\' --data \'{"name": "TestViewFromJenkins", "byKey": "date", "groupByKey1": "device.os", "groupByKey2": "device.version"}\'''', returnStdout: true).trim()
        echo "Response from Create View: ${triggerResponseCreateTestView}"
        
        testViewId = getJsonObjectFromResponseOutputID(triggerResponseCreateTestView, "id")
        testViewName = getJsonObjectFromResponseOutputName(triggerResponseCreateTestView, "name")
        
        echo "testViewId: ${testViewId}"
        echo "testViewName: ${testViewName}"
        echo "jenkinsbuildnr: ${JENKINS_BUILD_NUMBER}"
        
        triggerResponseGetTestViewResults = sh(script: '''curl --location --request GET \'''' + CLOUD_URL + '''/reporter/api/testView/''' + testViewId + '''/summary?filter=%7B%22Jenkins_Build_Number%22%3A%225%22%7D\' --header \'Authorization: Bearer ''' + ACCESS_KEY + '''\'''', returnStdout: true).trim()
        // triggerResponseGetTestViewResults = sh(script: '''curl --location --request GET \'''' + CLOUD_URL + '''/reporter/api/testView/''' + testViewId + '''/summary?filter=%7B%22Jenkins_Build_Number%22%3A%22''' + JENKINS_BUILD_NUMBER + '''%22%7D\' --header \'Authorization: Bearer ''' + ACCESS_KEY + '''\'''', returnStdout: true).trim()
        echo triggerResponseGetTestViewResults
        echo "Response from Get Test View Results: ${triggerResponseGetTestViewResults}"
    
        passedCount = getJsonObjectFromResponseOutput(triggerResponseGetTestViewResults, "passedCount")
        echo "Passed Count: ${passedCount}"
        
        failedCount = getJsonObjectFromResponseOutput(triggerResponseGetTestViewResults, "failedCount")
        echo "Failed Count: ${failedCount}"
        
        incompleteCount = getJsonObjectFromResponseOutput(triggerResponseGetTestViewResults, "incompleteCount")
        echo "Incomplete Count: ${incompleteCount}"
        
        skippedCount = getJsonObjectFromResponseOutput(triggerResponseGetTestViewResults, "skippedCount")
        echo "Skipped Count: ${skippedCount}"
        
        totalCount = getJsonObjectFromResponseOutput(triggerResponseGetTestViewResults, "_count_")
        echo "Total Count: ${totalCount}"
    }
    stage('Tear Down') {
        triggerResponseDeleteTestView = sh(script: '''curl --location --request DELETE \'''' + CLOUD_URL + '''/reporter/api/testView/''' + testViewId + '''\' --header \'Content-Type: application/json\' --header \'Authorization: Bearer ''' + ACCESS_KEY + '''\'''', returnStdout: true).trim()
        echo triggerResponseDeleteTestView
    }
}

```
