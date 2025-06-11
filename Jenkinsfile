pipeline {
    agent any

    environment {
        SLACK_CHANNEL = '#devops'
        SLACK_TOKEN = credentials('SLACK_BOT_TOKEN')
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/umair615/django-notes-app.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the Docker image'
                sh 'docker build -t notes-app:latest .'
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                echo 'Pushing the image to DockerHub'
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                                  usernameVariable: 'dockerhubUser',
                                                  passwordVariable: 'dockerhubPass')]) {
                    sh 'docker login -u $dockerhubUser -p $dockerhubPass'
                    sh 'docker image tag notes-app:latest alyeeumair26/notes-app:latest'
                    sh 'docker push alyeeumair26/notes-app:latest'
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the container'
                sh 'docker pull alyeeumair26/notes-app:latest'
                sh 'docker run -d -p 8001:8001 alyeeumair26/notes-app:latest'
            }
        }
    }

    post {
        success {
            script {
                sendSlack("✅ *SUCCESS*: Job `${env.JOB_NAME}` #${env.BUILD_NUMBER} completed successfully.")
            }
        }
        failure {
            script {
                sendSlack("❌ *FAILURE*: Job `${env.JOB_NAME}` #${env.BUILD_NUMBER} failed. Check Jenkins logs.")
            }
        }
    }
}

def sendSlack(String message) {
    sh """
    curl -X POST https://slack.com/api/chat.postMessage \\
      -H "Authorization: Bearer ${env.SLACK_TOKEN}" \\
      -H "Content-type: application/json" \\
      --data '{
        "channel": "${env.SLACK_CHANNEL}",
        "text": "${message}"
      }'
    """
}
