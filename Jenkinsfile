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
                    app.tag("latest")
                    app.tag("${env.BUILD_NUMBER}")
                    //app.inside {
                    //    sh 'echo Hello, World!'
                    //}
                }
            }
        }
        stage('Push Docker Image latest') {
        steps {
            script {
                withCredentials([usernamePassword(credentialsId: "docker_hub_login", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                try {
                        sh "docker login -u \$DOCKER_USERNAME -p \$DOCKER_PASSWORD https://registry.hub.docker.com"
                        sh "docker push $DOCKER_IMAGE_NAME:latest"
                        sh "docker push $DOCKER_IMAGE_NAME:${env.BUILD_NUMBER}"
                    } catch (Exception e) {
                        echo "Error: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error "Failed to push Docker image"
                        }
                    }
                }
            }
        }

        stage('CanaryDeploy') {
            // when {
            //     branch 'master'
            // }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                sh 'kubectl apply -f train-schedule-kube-canary.yml'
                //kubernetesDeploy(
                //    kubeconfigId: 'kubeconfig',
                //    configs: 'train-schedule-kube-canary.yml',
                //    enableConfigSubstitution: true
                //)
            }
        }
        stage('DeployToProduction') {
            // when {
            //    branch 'master'
            // }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh 'kubectl apply -f train-schedule-kube.yml'
                // kubernetesDeploy(
                //     kubeconfigId: 'kubeconfig',
                //     configs: 'train-schedule-kube-canary.yml',
                //     enableConfigSubstitution: true
                // )
                // kubernetesDeploy(
                //     kubeconfigId: 'kubeconfig',
                //     configs: 'train-schedule-kube.yml',
                //     enableConfigSubstitution: true
                // )
            }
        }
    }
}
