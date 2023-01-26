properties([pipelineTriggers([githubPush()])])
pipeline{
    agent{
        node {
            label 'slave1'
        }
    }
    environment {
        repo_creds=credentials('cred_name')
    } 
    tools {
        maven 'My-Maven'
    }
    stages{
        stage("Cloning Repo"){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'git@github.com:musatee/helmfile_example.git']]])
            }
        }
        stage("Building Artifact"){
            steps{
                sh 'mvn clean install'
            }
            post {
                    failure {
                        slackSend channel: 'channel_name', message: "Building artifact is failed with build ID: ${env.BUILD_ID}!!"
                     }
                }
        }
        stage("Building Image & committing to dockerHub"){
            steps{
                script{
                    docker.withRegistry("https://index.docker.io/v1/","repo_creds") {
                        def app_image = docker.build("musatee11/springboot:${env.BUILD_ID}")
                        app_image.push()
                        }
                }
                
                
            }
            post {
                    always {
                        sh 'docker rmi -f $(docker images -aq)'
                     }
                    failure {
                        slackSend channel: 'channel_name', message: "Building image & commit to dockerHub is failed with build ID: ${env.BUILD_ID}!!"
                     }
                }

        } 
        stage("deploy to Cluster"){
           steps{
            sh "helmfile sync ./my-helmchart/ --set deployment.image.tag="${env.BUILD_ID}""
           }
           post {
                    failure {
                        slackSend channel: 'channel_name', message: "helmfile sync operation is failed with build ID: ${env.BUILD_ID}!!"
                     }
                } 
        }
    }
   post{
        
        always {
            cleanWs()
        }
        success {
            slackSend channel: 'channel_name', message: "Deployment to EKS is successful with build ID: ${env.BUILD_ID}!!"
        }
        failure {
           slackSend channel: 'channel_name', message: "Deployment to EKS is failed on build ID: ${env.BUILD_ID}" 
        } 
       
    } 
}
