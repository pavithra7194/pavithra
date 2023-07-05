import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=be71113f-0d94-46d6-ac5f-638f80f5585b',
        'AZURE_TENANT_ID=32de2f9d-f504-425a-9cd9-00c61749d137']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'pavi'
      def webAppName = 'java987'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'javaazureapp', passwordVariable: 'pU_8Q~amqAh3wxSit6x8QJxJGHNafcMRJXx2zabF', usernameVariable: '1f0bb3bd-bf11-44c2-9c77-d5a350796729')]) {
       sh '''
          az login --service-principal -u 1f0bb3bd-bf11-44c2-9c77-d5a350796729 -p pU_8Q~amqAh3wxSit6x8QJxJGHNafcMRJXx2zabF -t 32de2f9d-f504-425a-9cd9-00c61749d137
          az account set -s be71113f-0d94-46d6-ac5f-638f80f5585b
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
