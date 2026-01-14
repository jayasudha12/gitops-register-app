pipeline {
    agent { label "jenkins-Agent" }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag to deploy')
    }

    environment {
        APP_NAME    = "register-app"
        DOCKER_USER = "puvisha007"
        IMAGE_NAME  = "${DOCKER_USER}/${APP_NAME}"
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

        stage("Update the Deployment Image Tag") {
            steps {
                script {
                    sh """
                        echo "✅ Before update:"
                        cat deployment.yaml

                        # ✅ Update ONLY image field (Safe fix)
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml

                        echo "✅ After update:"
                        cat deployment.yaml
                    """
                }
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                script {
                    sh """
                        git config --global user.name "jayasudha12"
                        git config --global user.email "puvishapa@gmail.com"

                        git add deployment.yaml

                        # ✅ Avoid failing if no changes
                        git diff --cached --quiet || git commit -m "Updated Deployment Image Tag to ${IMAGE_TAG}"
                    """

                    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                        sh "git push https://github.com/jayasudha12/gitops-register-app.git main"
                    }
                }
            }
        }

    }
}
