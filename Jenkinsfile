pipeline{
  agent { label "agentfarm" }
    stages {
      stage('Delete the workspace'){
        steps{
         cleanWs()
        }

      }
      stage('Installing Java and Maven') {
        steps {
          script {
            def maven_exists = fileExists '/usr/share/maven'
            if (maven_exists == true) {
              echo "Skipping Maven install - already installed"
            } else {
              sh 'sudo apt-get update -y && sudo apt-get upgrade -y'
              sh 'sudo apt install -y wget tree unzip openjdk-11-jdk maven'
              sh 'mvn -version'
            }
         }
       }
     }
      stage('Download Java Code'){
        steps {
          git branch: 'main', credentialsId: 'git-repo-creds', url: 'git@github.com:kshyamsunder/-java-springboot-sample-app.git'
        }
      }

     stage('Compiling and Running Test Cases') {
       steps {
         sh 'mvn clean'
         sh 'mvn compile'
         sh 'mvn test'
       }
     }


      stage('Send Slack Notification') {
        steps {
          slackSend color: 'warning', message: "Mr. Karthik: Please approve ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.JOB_URL} | Open>)"
        }
      }
      stage('Request Input') {
        steps {
          input 'Please approve or deny this build'
        }
      }
    
     stage('Creating package') {
       steps {
         sh 'mvn package'
       }
     }
   }

   stage('Deploying Application') {
     steps {
       script{
         withEnv(['JENKINS_NODE_COOKIE=dontkill']) {
         sh 'nohup java -jar ./target/jenkin-java-training-0.0.1-SNAPSHOT.jar &'
         }
       }
     }
   }

    post {
      success {
        slackSend color: 'warning', message: "Build ${env.JOB_NAME} ${env.BUILD_NUMBER} was successful ! :)"
      }
      failure {
          slackSend color: 'warning', message: "Build ${env.JOB_NAME} ${env.BUILD_NUMBER} failed :("
      }
    }


}
