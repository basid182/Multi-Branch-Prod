pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "alie12361/multibranch-flask-app"
        GIT_USER   = "basid182"
        GIT_EMAIL  = "basid.tibco@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when { branch 'main' }
            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                                    }
            }
        }

        stage('Update K8s Manifest') {
            when { branch 'main' }
            steps {
                script {

                    
                    withCredentials([usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                                            )]) {
                        sh """
                        set -e
                        git config user.name "$GIT_USER"
                        git config user.mail "$GIT_EMAIL"
                        
                        git fetch origin
                        git checkout main
                        git reset --hard origin/main
                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml
                        git add k8s/deployment.yml
                        git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}@github.com/basid182/Multi-Branch-Prod.git main
                        """
                    }
                }
            }
        }
    }
}
