pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "ldfoskey007/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
          }        
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                     docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([sshUserPrivateKey(credentialsId: 'webserver_login', keyFileVariable: 'MYSSHKEY', usernameVariable: 'USERNAME')]) {
                    sh ''' 
                     envsubst < ./train-schedule-kube.yml > /var/lib/jenkins/workspace/train-schedule_master@tmp\train-schedule-kube.yml 
                    'scp -i $MYSSHKEY train-schedule-kube.yml $USERNAME@$control_ip:/tmp'
                    'ssh -i $MYSSHKEY $USERNAME@$control_ip "kubectl apply -f /tmp/train-schedule-kube.yml && rm /tmp/train-schedule-kube.yml" '
                    
                    '''
                          }
                      }
                 }
    }
}
