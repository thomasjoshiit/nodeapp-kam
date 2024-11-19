pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
        NEXUS_URL  = "172.31.34.232:8080"
        IMAGE_URL_WITH_TAG = "${NEXUS_URL}/node-app:${DOCKER_TAG}"
    }
    stages{
        stage('Build Docker Image'){
            steps{
                // sh "docker build . -t ${IMAGE_URL_WITH_TAG}"
                sh "docker build . -t cfusionjo/nodeapp:${DOCKER_TAG}"

            }
        }
        stage('Nexus Push'){
            steps{
                // withCredentials([string(credentialsId: 'nexus-pwd', variable: 'nexusPwd')]) {
                //     sh "docker login -u cfusionjo -p ${nexusPwd} ${NEXUS_URL}"
                //     sh "docker push ${IMAGE_URL_WITH_TAG}"
                // }
                withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerHubPWD')]) {
                    sh "docker login -u cfusionjo -p ${dockerHubPWD}"
                    sh "docker push cfusionjo/nodeapp:${DOCKER_TAG}"
                }

            }
        }

        stage('Deploy to Kubernetes'){
            steps{
               sh "chmod +x changeTag.sh"
               sh "./changeTag.sh ${DOCKER_TAG}"
               sshagent(['kopsMachineKey']) {
                   sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ubuntu@18.208.229.22:/home/ubuntu"
                    script{
                        try{
                            sh "ssh ubuntu@18.208.229.22 kubectl apply -f ." 
                        }
                        catch(error) 
                            {
                            sh "ssh ubuntu@18.208.229.22 kubectl create -f ."
                            }
                    }
                }
            }
        }



        // stage('Docker Deploy Dev'){
        //     steps{
        //         sshagent(['tomcat-dev']) {
        //             withCredentials([string(credentialsId: 'nexus-pwd', variable: 'nexusPwd')]) {
        //                 sh "ssh ec2-user@172.31.0.38 docker login -u admin -p ${nexusPwd} ${NEXUS_URL}"
        //             }
		// 			// Remove existing container, if container name does not exists still proceed with the build
		// 			sh script: "ssh ec2-user@172.31.0.38 docker rm -f nodeapp",  returnStatus: true
                    
        //             sh "ssh ec2-user@172.31.0.38 docker run -d -p 8080:8080 --name nodeapp ${IMAGE_URL_WITH_TAG}"
        //         }
        //     }
        // }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
