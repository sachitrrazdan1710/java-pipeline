@Library('Shared') _
pipeline {
    agent { label 'docker-agent' } // Defined once for the whole pipeline

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'v1.0.0', description: 'Docker image tag')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (!params.IMAGE_TAG?.trim()) {
                        error("IMAGE_TAG must be provided.")
                    }
                    echo "Image tag: ${params.IMAGE_TAG}"
                }
            }
        }

        stage("Workspace Cleanup") {
            steps {
                cleanWs()
            }
        }

        stage("Git Checkout") {
            steps {
                script {
                    // Bypass the buggy Git Plugin by using raw shell commands
                    sh "rm -rf *" 
                    sh "git clone -b main https://github.com/sachitrrazdan1710/java-pipeline.git ."
                }
            }
        }

        stage("Trivy: Filesystem Scan") {
            steps {
                script {
                    trivy_scan()
                }
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn clean package'
            }
        }

        stage("Docker Build") {
            steps {
                script {
                    sh 'cp webapp/target/webapp.war .'
                    docker_build(
                        "regapp",
                        "${params.IMAGE_TAG}",
                        "sachitrrazdan1710"
                    )
                }
            }
        }

        stage("Trivy: Image Scan") {
            steps {
                script {
                    sh "trivy image --severity HIGH,CRITICAL sachitrrazdan1710/regapp:${params.IMAGE_TAG}"
                }
            }
        }

        stage("Docker: Push to DockerHub") {
            steps {
                script {
                    docker_push(
                        imageName: "sachitrrazdan1710/regapp",
                        imageTag: "${params.IMAGE_TAG}",
                        credentials: "docker-hub-credentials"
                    )
                }
            }
        }
    }

    post {
        success {
            build job: "gitops-cd-pipeline", parameters: [
                string(name: 'IMAGE_TAG', value: "${params.IMAGE_TAG}")
            ]
        }
        failure {
            echo "CI Pipeline Failed."
        }
    }
}
