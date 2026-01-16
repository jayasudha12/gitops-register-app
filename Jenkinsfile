pipeline {
    agent { label "jenkins-Agent" }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag passed from CI pipeline')
    }

    environment {
        APP_NAME = "register-app"
    }

    stages {

        stage("Cleanup Workspace") {
            steps { cleanWs() }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/jayasudha12/gitops-register-app.git'
            }
        }

        stage("Update Deployment Image Tag") {
            steps {
                sh """
                    echo "✅ IMAGE_TAG received: ${IMAGE_TAG}"

                    echo "Before update:"
                    cat deployment.yaml

                    sed -i "s|image:.*|image: puvisha007/${APP_NAME}:${IMAGE_TAG}|g" deployment.yaml

                    echo "After update:"
                    cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                    git config --global user.name "jayasudha12"
                    git config --global user.email "puvishapa@gmail.com"

                    git add deployment.yaml
                    git commit -m "Updated image tag to ${IMAGE_TAG}" || echo "No changes to commit"
                """

                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/jayasudha12/gitops-register-app.git main"
                }
            }
        }

        stage("Deploy to EKS") {
            steps {
                sh """
                    echo "Applying deployment to EKS..."
                    kubectl apply -f deployment.yaml

                    echo "Restarting rollout..."
                    kubectl rollout restart deployment/${APP_NAME}

                    echo "Waiting for rollout..."
                    kubectl rollout status deployment/${APP_NAME} --timeout=180s

                    echo "✅ Running image now:"
                    kubectl describe deploy ${APP_NAME} | grep -i image
                """
            }
        }
    }
}
