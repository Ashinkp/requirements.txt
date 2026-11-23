pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'ashin2002/requirements.txt'
        SONAR_SERVER_NAME     = 'sonarqube-env'
        SONAR_SCANNER_TOOL    = 'sonarqube-scanner'
        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_BRANCH       = 'main'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo "passed"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"
                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=requirements.txt-key \
                            -Dsonar.projectName='requirements.txt-project' \
                            -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Build & Post Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

                            docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} -t ${DOCKER_IMAGE}:latest -f Dockerfile .

                            docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN')]) {
                    sh """
                        git config user.name "Ashinkp"
                        git config user.email "ashinkp33@gmail.com"
                        git remote set-url origin https://\${GH_USER}:\${GH_TOKEN}@github.com/Ashinkp/requirements.txt.git

                        git fetch origin ${MANIFEST_BRANCH}
                        git checkout -B ${MANIFEST_BRANCH} origin/${MANIFEST_BRANCH}

                        sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|' manifests/deployment.yaml

                        git diff --quiet || {
                            git commit -am "chore(manifest): update maker image tag to ${env.BUILD_NUMBER} [skip ci]"
                            git push origin ${MANIFEST_BRANCH}
                        }
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! Manifest repo updated with new image tag.'
        }
        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }
    }
}
