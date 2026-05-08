@Library('Shared') _
pipeline {
    agent none

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker image tag')
    }
    
    stages {
        stage("Validate Parameters") {
            agent any
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
            agent any
            steps {
                cleanWs()
            }
        }

        stage("Git Checkout") {
            agent any
            steps {
                script {
                    clone("https://github.com/sachitrrazdan1710/java-pipeline.git", "devops")
                }
                stash name: 'source-code', includes: '**/*'
            }
        }

        stage("Trivy: Filesystem Scan") {
            agent { label 'docker-agent' }
            steps {
                unstash 'source-code'
                script {
                    trivy_scan()
                }
            }
        }

        stage("Maven Build") {
            agent any
            steps {
                unstash 'source-code'
                sh 'mvn clean package'
                stash name: 'build-artifact', includes: 'webapp/target/webapp.war'
            }
        }

        stage("Docker Build") {
            agent { label 'docker-agent' }
            steps {
                unstash 'source-code'
                unstash 'build-artifact'
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
            agent { label 'docker-agent' }
            steps {
                script {
                    sh "trivy image --severity HIGH,CRITICAL sachitrrazdan1710/regapp:${params.IMAGE_TAG}"
                }
            }
        }

        stage("Docker: Push to DockerHub") {
            agent { label 'docker-agent' }
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
            echo "CI Pipeline Failed. ServiceNow integration is currently disabled for this branch."
        }
    }
}
