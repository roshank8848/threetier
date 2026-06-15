pipeline {
    agent any
    environment {
      APP_SERVER = "10.0.1.182"
      }
    parameters {
        string(name: 'BACKEND_VERSION', defaultValue: 'latest', description: 'Backend version/tag to deploy')
        string(name: 'FRONTEND_VERSION', defaultValue: 'latest', description: 'Frontend version/tag to deploy')
    }
    stages {
        stage('Trigger Downstream Builds') {
            parallel {
                stage('Build Backend') {
                    steps {
                        // Triggers the separate backend job and passes parameters
                        build job: 'backend-pipeline-job', 
                              parameters: [string(name: 'VERSION', value: params.BACKEND_VERSION)],
                              wait: true
                    }
                }
                stage('Build Frontend') {
                    steps {
                        // Triggers the separate frontend job
                        build job: 'frontend-pipeline-job', 
                              parameters: [string(name: 'VERSION', value: params.FRONTEND_VERSION)],
                              wait: true
                    }
                }
            }
        }
        stage('Deploy System') {
            steps {
                echo "Deploying Backend (${params.BACKEND_VERSION}) and Frontend (${params.FRONTEND_VERSION})..."
                script {
                  withCredentials([sshUserPrivateKey(
                    credentialsId: 'app-server-ssh',     // ← Your credential ID
                    keyFileVariable: 'SSH_KEY'
                )]) {
                      sh '''
                         echo "copying dokcer-compose file to deploymenet server"
                         scp -i "$SSH_KEY" -o StrictHostKeyChecking=no docker-compose.yaml ubuntu@"${APP_SERVER}":~/docker-compose.yaml
                         ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ubuntu@"${APP_SERVER}" "
                            docker compose down
                            sleep 10 
                            docker compose up -d
                          "
                      '''
                    }
                  }
            }
        }
    }
}

