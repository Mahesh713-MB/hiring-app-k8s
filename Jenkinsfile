
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKERHUB_CREDENTIALS = 'docker-hub'
        DOCKERHUB_USERNAME    = 'mahesh0504docker'
        IMAGE_NAME            = 'my-nginx'

        // Replace with your Argo CD manifests repository
        GITOPS_REPO            = 'https://github.com/Mahesh713-MB/hiring-app-k8s.git'
        GITOPS_BRANCH          = 'main'
        GITOPS_CREDENTIALS     = 'Github_server'

        // Change if your deployment file has another location
        DEPLOYMENT_FILE        = 'deployment.yaml'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh '''
                    echo "Application source checked out"
                    git rev-parse --short HEAD
                    test -f Dockerfile
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    echo "Checking Docker build configuration"
                    docker --version
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Login and Push') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: env.DOCKERHUB_CREDENTIALS,
                            usernameVariable: 'DH_USER',
                            passwordVariable: 'DH_TOKEN'
                        )
                    ]) {
                        sh '''
                            set +x
                            echo "$DH_TOKEN" | docker login \
                                -u "$DH_USER" --password-stdin

                            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                                ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}

                            docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                                ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest

                            docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
                            docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Update GitOps Repository') {
            steps {
                dir('gitops') {
                    deleteDir()

                    git branch: "${GITOPS_BRANCH}",
                        credentialsId: "${GITOPS_CREDENTIALS}",
                        url: "${GITOPS_REPO}"

                    sh '''
                        set -eu

                        test -f "${DEPLOYMENT_FILE}"

                        sed -i "s|image:.*|image: ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}|" \
                            "${DEPLOYMENT_FILE}"

                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@example.com"

                        git add "${DEPLOYMENT_FILE}"

                        if ! git diff --cached --quiet; then
                            git commit -m "Deploy hiring-app build ${BUILD_NUMBER}"
                            git push origin "${GITOPS_BRANCH}"
                        else
                            echo "No manifest changes to commit"
                        fi
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the failed stage in Jenkins Console Output.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
