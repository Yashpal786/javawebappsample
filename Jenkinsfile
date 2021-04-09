import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=89453064-5455-4683-94e8-f6a5e3f0b82a',
        'AZURE_TENANT_ID=0adb040b-ca22-4ca6-9447-ab7b049a22ff']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'Quickstartjenkins-rg'
      def webAppName = 'mywebapppppp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'd~fmvTRlP~n_i1xi2GosZwH2Wi708JV~4P', usernameVariable: '03457a0a-e707-4ce4-b4e2-2e071baa7583')]) {
       sh '''
          az login --service-principal -u 03457a0a-e707-4ce4-b4e2-2e071baa7583 -p d~fmvTRlP~n_i1xi2GosZwH2Wi708JV~4P -t 0adb040b-ca22-4ca6-9447-ab7b049a22ff
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
