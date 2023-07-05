import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

pipeline {
  agent any
  
   environment {
    AZURE_SUBSCRIPTION_ID = '9f72d118-d6c7-4392-83cb-fcaf495b9855'
    AZURE_TENANT_ID = '02ab6863-c3b7-458c-a98e-8e28a4106622'
  }
  
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
    
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('deploy') {
      steps {
        script {
           def resourceGroup = 'gnew'
          def webAppName = 'workshop74'
          
          // login Azure
          withCredentials([azureServicePrincipal('suuuuuuuuucccccchiiiiiiiiiiiii')]) { 
            sh '''
              az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
              az account set -s $AZURE_SUBSCRIPTION_ID
            '''
          }
          
          // get publish settings
          def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
          def ftpProfile = getFtpPublishProfile(pubProfilesJson)
          
          // upload package
          sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
          
          // log out
          sh 'az logout'
        }
      }
    }
  }
}
