pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'omarabdo4'
        IMAGE_NAME      = 'butcher-cars-frontend'
        IMAGE_TAG       = "v1.0.${BUILD_NUMBER}"
        GITOPS_REPO     = 'github.com/OmarAbdo74-B1/gitops-manifests.git'
        DOCKER_CREDS_ID = 'dockerhub-credentials'
        GITHUB_CREDS_ID = 'github-creds'
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh """
                        echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
                        docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Promote to Staging in GitOps') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GITHUB_CREDS_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh """
                        rm -rf gitops-temp
                        git clone https://\${GIT_TOKEN}@${GITOPS_REPO} gitops-temp
                        cd gitops-temp/frontend/overlays/staging

                        sed -i 's/newTag:.*/newTag: "${IMAGE_TAG}"/' kustomization.yaml

                        git config user.email "jenkins-ci@enterprise.local"
                        git config user.name "Jenkins CI/CD"
                        git add kustomization.yaml
                        git commit -m "ci: bump frontend image to ${IMAGE_TAG} [skip ci]" || echo "No changes detected"
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        always {
            sh "docker logout || true"
            sh "rm -rf gitops-temp"
        }
        success {
            echo "Successfully built, pushed and promoted Frontend version: ${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed! Please check logs."
        }
    }
}