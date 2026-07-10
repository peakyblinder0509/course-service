pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'SonarScanner'
        IMAGE_NAME = "my-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        HARBOR_URL = "harbor-node1.com"
        PROJECT    = "crm-adminpanel"
        HARBOR     = "robot$test-project1+harbor-1"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-crd',
                    url: 'https://github.com/peakyblinder0509/course-service.git'
            }
        }

        stage('Check Files') {
            steps {
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('NPM Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=CRM-ADMINPANEL \
                        -Dsonar.projectName=CRM-ADMINPANEL \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/,dist/ \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    """
                }
            }
        }
 
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${HARBOR_URL}/${PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }

        stage('Docker Login Harbor') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'test-harbor',
                    usernameVariable: 'HARBOR_USER',
                    passwordVariable: 'HARBOR_PASS'
                )]) {
                    sh 'echo "$HARBOR_PASS" | docker login harbor-node1.com -u "$HARBOR_USER" --password-stdin'
                }
            }
        }

        stage('Push To Harbor') {
            steps {
                sh 'docker push ${HARBOR_URL}/${PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}'
            }
        }
    }

    post {
        always {
            sh """
            echo "Removing old Docker images..."
            docker images ${IMAGE_NAME} --format "{{.Tag}}" | \
            grep -v "${BUILD_NUMBER}" | while read tag; do
                echo "Removing ${IMAGE_NAME}:\$tag"
                docker rmi ${IMAGE_NAME}:\$tag || true
            done
            """
        }
    }
}
