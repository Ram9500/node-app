pipeline{
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh"docker build . -t ramgram/nodeapp:${DOCKER_TAG}"
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                  sh "docker login -u ramgram -p ${dockerHubPwd}"
                  sh "docker push ramgram/nodeapp:${DOCKER_TAG}"
               }
            }
        }
        stage('Deploy to k8s'){
            steps{
               sh "chmod +x changeTag.sh"
               sh "./changeTag.sh ${DOCKER_TAG}"
               sshagent(['kops']) {
                   sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@54.157.146.244:/home/ec2-user/"                   
                   }
               }
            }
        }        
    }
def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
