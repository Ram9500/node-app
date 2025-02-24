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
               sshagent(['kops-machine']){
                   sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@3.6.126.155:/home/ec2-user/"
                   script{
                       try{
                           sh "ssh ec2-user@3.6.126.155 kubectl apply -f ."
                       }catch(error){
                           sh "ssh ec2-user@3.6.126.155 kubectl create -f ."
                       }
                   }
               }
            }
        }        
    }
}
def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
