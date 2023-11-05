pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "lolam/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh "npm install"
                // sh './gradlew build --no-daemon'
                // archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            // when {
            //     branch 'master'
            // }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    //app.inside {
                    //    sh 'echo Hello, World!'
                    //}
                }
            }
        }
        stage('Push Docker Image latest') {
            //when {
            //    branch 'master'
            //}
            steps {
                script {
                    try {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                            echo "Pushing Docker Image: ${DOCKER_IMAGE_NAME}:latest"
                            docker.push("${DOCKER_IMAGE_NAME}:latest")
                        }
                    }catch (Exception e) {
                        echo "Error: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error "Failed to push Docker image"
                        }
                }
            }
        }
        stage('Push Docker Image tag') {
            //when {
            //    branch 'master'
            //}
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        docker.push("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
