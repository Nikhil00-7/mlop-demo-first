pipeline {

    agent any
 
    tools{
        python "python"
    }
    environment {
        DOCKER_USER = "docdon0007"
        IMAGE = "model"
        TAG = "${env.BRANCH_NAME ?: 'latest'}"
    }

    options {
        timestamps()
        timeout(time: 30, unit: "MINUTES")
    }

    stages {

        stage("Matrix Build") {
            matrix {
                axes {
                    axis {
                        name 'PYTHON_VERSION'
                        values '3.10', '3.11'
                    }
                }

                stages {

                    stage("Checkout Code") {
                        steps {
                            git branch: "main",
                                url: "https://github.com/Nikhil00-7/mlop-demo-first.git"
                        }
                    }

                    stage("Pre Checks") {
                        steps {
                            script {
                                def files = ["requirements.txt", "train.py", "Dockerfile"]
                                def missingFiles = []

                                files.each { file ->
                                    if (!fileExists(file)) {
                                        missingFiles.add(file)
                                    }
                                }

                                if (missingFiles.size() > 0) {
                                    error("Missing required files: ${missingFiles.join(', ')}")
                                }
                            }
                        }
                    }

                    stage("Setup Python Env") {
                        steps {
                            sh """
                            python${PYTHON_VERSION} -m venv venv
                            . venv/bin/activate
                            pip install --upgrade pip setuptools wheel
                            """
                        }
                    }

                    stage("Install Dependencies") {
                        steps {
                            sh """
                            . venv/bin/activate
                            pip install -r requirements.txt
                            """
                        }
                    }

                    stage("Train Model") {
                        steps {
                            sh """
                            . venv/bin/activate
                            python train.py
                            ls -la artifacts || true
                            """
                        }
                    }

                    stage("Archive Artifacts") {
                        steps {
                            archiveArtifacts artifacts: "artifacts/**", fingerprint: true
                        }
                    }

                    stage("Docker Build & Push") {
                        steps {
                            sh "docker build -t ${DOCKER_USER}/${IMAGE}:${TAG}-${PYTHON_VERSION} ."

                            withCredentials([usernamePassword(
                                credentialsId: "dockerhub-login",
                                usernameVariable: "DOCKER_USER_NAME",
                                passwordVariable: "DOCKER_PASSWORD"
                            )]) {
                                sh """
                                echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USER_NAME --password-stdin
                                docker push ${DOCKER_USER}/${IMAGE}:${TAG}-${PYTHON_VERSION}
                                """
                            }
                        }
                    }
                }
            }
        }
    }
}