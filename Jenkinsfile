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
                    // Ensure the copyArtifacts plugin is correctly set up to retrieve the file
                    def artifacts = copyArtifacts(
                        projectName: 'CI-apps', 
                        filter: 'image_tag.txt', 
                        target: '', 
                        flatten: true
                    )
                    // Read the image tag from the file
                    def imageTagContent = readFile('image_tag.txt').trim()
                    env.IMAGE_TAG = imageTagContent // Set the IMAGE_TAG environment variable
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
