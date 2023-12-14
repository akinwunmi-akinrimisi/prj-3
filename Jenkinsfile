pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
        KUBE_CONFIG = credentials('kube-config-credentials-id')
        GITHUB_REPO_URL = 'https://github.com/yourusername/your-repo.git'
        DOCKERFILE_PATH = 'path/to/Dockerfile'
        IMAGE_NAME = sh(script: "grep -oP '(?<=^\\s*FROM\\s+)[^\\s]+' ${DOCKERFILE_PATH}", returnStdout: true).trim()
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: "${env.GITHUB_REPO_URL}"]]])
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def dockerImage = docker.build("${env.IMAGE_NAME}:latest", "${env.DOCKERFILE_PATH}")
                    docker.withRegistry('https://registry.hub.docker.com', "${env.DOCKER_HUB_CREDENTIALS}") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: "${env.KUBE_CONFIG}", variable: 'KUBE_CONFIG_FILE')]) {
                        sh "kubectl --kubeconfig=${KUBE_CONFIG_FILE} apply -f path/to/service_volume_statefulset.yml"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Cleanup steps, if any
            }
        }
    }
}
