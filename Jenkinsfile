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
        stage('Retrieve Image Tag') {
            steps {
            copyArtifacts(projectName: 'appjob', filter: 'image_tag.txt', target: '', flatten: true)
            }
        }
        stage("Update the Deployment Tags") {
            steps {
                sh """
                   cat deployment.yaml
                   sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                   cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                   git config --global user.name "rizgh"
                   git config --global user.email "riz2490@hotmail.com"
                   git add deployment.yaml
                   git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                  sh "git push https://github.com/rizgh/gitops-register-app main"
                }
            }
        }
      
    }
}
