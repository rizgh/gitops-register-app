pipeline {
    agent { label "slave" }
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Image tag to deploy')
    }
    environment {
        APP_NAME = "register-app-pipeline"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/rizgh/gitops-register-app'
            }
        }
        stage("Retrieve Image Tag") {
            steps {
                script {
                def artifactFound = copyArtifacts(projectName: 'CI-apps', filter: 'image_tag.txt', target: '', flatten: true)
                    if (!artifactFound) {
                    error("Artifact image_tag.txt not found in CI-apps job.")
                        }
                }
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                script {
                    sh """
                    sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                    """
                }
            }
        }
        stage("Push the Changed Deployment File to Git") {
            steps {
                script {
                    sh """
                    git config --global user.name "rizgh"
                    git config --global user.email "riz2490@hotmail.com"
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest" || echo "No changes to commit"
                    """
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                        sh "git push https://${GIT_USER}:${GIT_PASS}@github.com/rizgh/gitops-register-app main"
                    }
                }
            }
        }
    }
}
