pipeline {
  agent any
  stages {
    stage ('New Build Notifications') {
       parallel {
          stage ('Email notification') {
            steps {
                    echo '>>> Send New application build notification'
                    //mail bcc: '', body: 'Thanks', cc: '', from: '', replyTo: '', subject: "New Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME]", to: 'khaled.amrosy.fci@gmail.com'
                    mail bcc: '', body: """Dears,
                    A new build has been started for the Job [$JOB_NAME].
                    Below are the Build details:
                    Build Number : $BUILD_NUMBER
                    Build URL is : $BUILD_URL
                    Build Tag : $BUILD_TAG
                    Node Name : $NODE_NAME
                    Executor Number : $EXECUTOR_NUMBER
                    Workspace : $WORKSPACE
                    Thanks. """, cc: '', from: '', replyTo: '', subject: "New Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME]", to: "$emailRecipientIDs"
            }
         }
         stage ('Slack Notification') {
            steps {
              echo '>>> Send New Build Slack notifcation'
              slackSend baseUrl: 'https://hooks.slack.com/services/', channel: "$slackChannelName", color: 'warning', message: "New Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME]", teamDomain: "$slackTeamDomainName", tokenCredentialId: 'pipeline-demo-slackID'
            }
          }
        }
    }
    
    stage('SCM Checkout') {
      steps {
        echo '>>> Start getting SCM code'
        git 'https://github.com/khalednoh/demo1.git'
        echo '>>> End getting SCM code'
      }
    }

    stage('Build JAR') {
      steps {
        echo '>>> Start building using Maven'
        sh 'mvn -Dmaven.test.skip=TRUE install'
        echo '>>> End building using Maven'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo '>>> start clearing old docker images'
        sh label: '', script: '''if docker images -a | grep "pipeline-demo*" | awk \'{print $1":"$2}\' | xargs docker rmi -f; then
         printf \'Clearing old images succeeded\\n\'
        else
          printf \'Clearing old images failed\\n\'
        fi'''
        echo '>>> End clearing old docker images'
        echo '>>> Start building App docker image'
        sh "docker build -t $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER --pull=true $WORKSPACE"
        echo '>>> End building App docker image'
      }
    }

    stage('Upload Docker Image') {
      steps {
        echo '>>> Login to Docker hub'
        withCredentials([string(credentialsId: 'DockHubSecret', variable: 'DockerHubIDSecret')]) {
            sh "docker login -u $MyDockerAccountName -p $DockerHubIDSecret"
        }
        echo '>>> Start uploading the docker image'
        sh "docker push $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER"
        echo '>>> End uploading the docker image'
      }
    }

    stage('Deploy') {
      steps {
        echo 'Start Killing and Removing old continaer'
        
        sh label: '', script: '''if docker ps -a | grep "Jenkins-pipeline-Demo*" | awk \'{print $1}\' | xargs docker rm -f; then
            printf \'Clearing old containers done succeeded\\n\'
        else
            printf \'Clearing old containers failed\\n\'
        fi'''

        echo 'End Killing and Removing old continaer'
        echo 'Start running a new container of the application'
        sh "docker run -d -p 8090:8090 --name=$MyTagName$BUILD_NUMBER $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER"
        echo 'End running a new container of the application'
      }
    }

  }
  environment {
    MyDockerAccountName = 'khalednoh'
    MyDockerReposioryName = 'pipeline-demo'
    MyTagName = 'Jenkins-pipeline-Demo'
    emailRecipientIDs='khaled.amrosy.fci@gmail.com'
    slackChannelName = '#pipeline-demo'
    slackTeamDomainName = 'pipeline-demo'
  }
  post { 
        always { 
            echo 'I will always run, and I can clean up the workspace here......'
        }
        success {
            echo 'I will run in case of failure, and I will send an email in case of failure.'
            mail bcc: '', body: """Dears,
            Congratulations the build # $BUILD_NUMBER for the Job [$JOB_NAME] has successded.
            Below are the Build details:
            Build Number : $BUILD_NUMBER
            Build URL is : $BUILD_URL
            Build Tag : $BUILD_TAG
            Node Name : $NODE_NAME
            Executor Number : $EXECUTOR_NUMBER
            Workspace : $WORKSPACE
            Thanks. """, cc: '', from: '', replyTo: '', subject: "Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME] >> Success", to: "$emailRecipientIDs"

            echo 'I will send a success notification on the slack channel'
            slackSend baseUrl: 'https://hooks.slack.com/services/', channel: "$slackChannelName", color: 'good', message: "Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME] >> Success", teamDomain: "$slackTeamDomainName", tokenCredentialId: 'pipeline-demo-slackID'
        }
        failure { 
            echo 'I will run in case of success, and I will send an email in case of success'
            mail bcc: '', body: """Dears,
            Unfortunately the build # $BUILD_NUMBER for the Job [$JOB_NAME] has failed.
            Below are the Build details:
            Build Number : $BUILD_NUMBER
            Build URL is : $BUILD_URL
            Build Tag : $BUILD_TAG
            Node Name : $NODE_NAME
            Executor Number : $EXECUTOR_NUMBER
            Workspace : $WORKSPACE
            Thanks. """, cc: '', from: '', replyTo: '', subject: "Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME] >> Failed", to: "$emailRecipientIDs"

            echo 'I will send a success notification on the slack channel'
            slackSend baseUrl: 'https://hooks.slack.com/services/', channel: "$slackChannelName", color: 'danger', message: "Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME] >> Failed", teamDomain: "$slackTeamDomainName", tokenCredentialId: 'pipeline-demo-slackID'
        }
  }
}
