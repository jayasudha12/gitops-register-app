pipeline {
    agent { label "jenkins-Agent" }

    environment {
        APP_NAME = "register-app"
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
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
                    echo "Before update:"
                    cat deployment.yaml

                    # ✅ Replace only the image tag line
                    sed -i 's|image: ${APP_NAME}.*|image: puvisha007/${APP_NAME}:${IMAGE_TAG}|g' deployment.yaml

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
    }
}
