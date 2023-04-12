pipeline {
    agent any
    environment { 
                    current_image= "karthikhari95/my-nginx-server" + ":$BUILD_NUMBER"
                }
    stages {
        stage('Getting build') {
            steps {
                script {
                    git 'https://github.com/mightycolony/helm-jen.git'
                }
            }
        }
        stage('Build') {
            steps {
                script {
                   println(current_image)
                   dockerImage = docker.build (current_image)
                }
            }
        }
        stage('Upload') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '230dd889-a7e6-43e6-adfa-abf2ea46e9f0', url: 'https://index.docker.io/v1/') {
                    dockerImage.push("$BUILD_NUMBER")
                        }
                }
            }
        }
        stage('Deploying  to Kubernetes') {
            steps {
               script {
                   kubeconfig(credentialsId: '162d1e41-7bcf-4e8b-a719-d2bd5be3d5e8', serverUrl: 'https://192.168.0.195:6443') {
                        checker = sh(returnStdout: true, script: """kubectl get deployments -n deployment -o jsonpath='{.items[*].metadata.name}'""").trim()
                        checker_service = sh(returnStdout: true, script: """kubectl get services -n deployment -o jsonpath='{.items[*].metadata.name}'""").trim()
                        current_image_number= sh(returnStdout: true, script:"""kubectl get deployments -n deployment -o 'jsonpath={.items[*].spec.template.spec.containers[*].image}'""").trim()

                        if ( "nginx-server" in checker) {
                            println( "$checker" + " " +  "Server Available")
                        }else if (current_image_number.isEmpty()){
                            sh 'kubectl create deployment nginx-server --image=${current_image} --replicas=2 -n deployment'
                        }
                        
                        if (current_image != current_image_number){
                            sh 'kubectl set image deployment/nginx-server my-nginx-server=${current_image} -n deployment'
                            println("image update!!")
                        }
                        
                        if ( "nginx-service" in checker_service ) {
                            println( "$checker_service" + " " + "Service Available")
                        }else{
                            sh "kubectl expose deployment nginx-server --port=80 --target-port=80 --selector=app=nginx-server --name=nginx-service --type=NodePort -n deployment"
                        }
                    
                    }     
                   
                }
            }
        }
        stage ('Cleaning Local Image') {
            steps {
                script {
                    sh 'docker rmi ${current_image}'
                }
            }
        }
    }
}


