pipeline {
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-token')
    }
    agent any
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage("Compile") {
            steps {
                sh "./gradlew compileJava"
            }
        }
        stage("Build") {
           steps {
               sh "./gradlew build"
               sh "cp ./build/libs/MiniBoard-0.0.1-SNAPSHOT.jar ./docker/smboard/"
           } 
        }
        stage("Docker Login") {
           steps {
               sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin" 
           }
        }
        stage("Docker Image Build") {
           steps {
               sh "docker build -t johbeomju/apache2_smboard:${BUILD_NUMBER} ./docker/apache2/"
               sh "docker build -t johbeomju/smboard_smboard:${BUILD_NUMBER} ./docker/smboard/"
               sh "docker build -t johbeomju/mariadb_smboard:${BUILD_NUMBER} ./docker/mariadb/"
           }
        }
        stage("Docker Image Push") {
           steps {
               sh "docker push johbeomju/apache2_smboard:${BUILD_NUMBER}"
               sh "docker push johbeomju/smboard_smboard:${BUILD_NUMBER}" 
               sh "docker push johbeomju/mariadb_smboard:${BUILD_NUMBER}" 
           } 
        }
        stage("Docker Image Clean up") {
           steps {
               sh "docker image rm johbeomju/apache2_smboard:${BUILD_NUMBER}" 
               sh "docker image rm johbeomju/smboard_smboard:${BUILD_NUMBER}" 
               sh "docker image rm johbeomju/mariadb_smboard:${BUILD_NUMBER}" 
           }
        }
        stage("Minikube start") {
           steps {
               sh "minikube start --driver=docker --cni=calico"
           }
        }
        stage("Deploy") {
           steps {
               sh "sed -i 's/{{VERSION}}/${BUILD_NUMBER}/g' ./kubernetes/apache2.yml"
               sh "sed -i 's/{{VERSION}}/${BUILD_NUMBER}/g' ./kubernetes/smboard.yml"
               sh "sed -i 's/{{VERSION}}/${BUILD_NUMBER}/g' ./kubernetes/mariadb.yml"
               sh "kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission"
               sh "kubectl apply -f ./kubernetes/mariadb.yml"
               sh "kubectl apply -f ./kubernetes/smboard.yml"
               sh "kubectl apply -f ./kubernetes/apache2.yml"
               sh "kubectl apply -f ./kubernetes/ingress.yml"
           } 
           post {
                success {
                    slackSend (
                        channel: "#배포",
                        color: "#2C953C",
                        message: "Miniboard 배포가 성공하였습니다."
                    )
                    echo "Completed Server Deploy"
                }
                failure {
                    slackSend (
                        channel: "#배포",
                        color: "#FF3232",
                        message: "Miniboard 배포가 실패하였습니다."
                    )
                    echo "Fail Server Deploy"
                }
          }
       }  
    }
}
