pipeline {
    agent none

    

    environment {
        DOCKERHUB_USER = "lakshmanan1996"
        GIT_REPO = "https://github.com/Lakshmanan1996/Trend.git"
        SERVICE_NAME = "Trend"
        IMAGE_NAME = "Trend-v1.0"
    }

    stages {

        /* ===================== CHECKOUT ===================== */
        stage('Checkout Code') {
            
            steps {
                cleanWs()
                git branch: 'main', url: env.GIT_REPO

                stash name: 'source',
                      includes: '**/*',
                      excludes: '**/.git/**,**/target/**'
            }
        }

        /* ===================== SONARQUBE ===================== */
        stage('SonarQube Analysis') {
            agent {label 'workernode1'}
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('sonarqube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=Trend-v1.0 \
                          -Dsonar.projectName=Trend-v1.0 \
                          -Dsonar.sources=dist 
                        """
                    }
                }
            }
        }

        /* ===================== QUALITY GATE ===================== */
        stage('Quality Gate') {
            label {'workernode1'}
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        /* ===================== DOCKER BUILD ===================== */
        stage('Docker Build') {
            
            steps {
                unstash 'source'
                sh '''
                  docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} .
                  docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} \
                             ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                '''
            }
        }

        /* ===================== TRIVY ===================== */
        stage('Trivy Scan') {
            
            steps {
                sh '''
                  trivy image --exit-code 0 --severity HIGH,CRITICAL \
                    ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                '''
                echo "✅ Trivy scan completed."
            }
        }

        /* ===================== PUSH TO DOCKER HUB ===================== */
        stage('Push Image') {
            
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }

                sh '''
                  docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                  docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Internet Banking CI Pipeline SUCCESS"
        }
        failure {
            echo "❌ Internet Banking CI Pipeline FAILED"
        }
    }
}

