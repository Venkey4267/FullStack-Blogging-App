pipeline {
    agent any
    
    tools {
        maven 'maven3'  // Use Maven tool configured in Jenkins
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // Reference to SonarQube scanner
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Venkey4267/FullStack-Blogging-App.git'
            }
        }

        // Compile the code using Maven
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        // Run Unit Tests
        stage('Unit-Test') {
            steps {
                sh 'mvn test'
            }
        }

        // Trivy File System Scan
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }

        // Perform SonarQube Analysis with updated project name 'blogging'
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=blogging -Dsonar.projectName=blogging -Dsonar.java.binaries=target"
                }

            }
        }

        // Build the application using Maven
        stage('Build Application') {
            steps {
                sh 'mvn package'  // Maven build command to create the artifact
            }
        }

        // Publish Artifact to a Maven repository (e.g., Nexus, Artifactory)
        stage('Publish Artifact') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'  // Deploy the built artifact to the repository
                }
            }
        }

        // Build Docker Image
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t adijaiswal/taskmaster:latest .'  // Build the Docker image
                    }
                }
            }
        }

        // Trivy Image Scan
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o image.html Venkey4267/blogging:latest'  // Scan the Docker image
            }
        }

        // Push Docker Image to the Registry
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push Venkey4267/blogging:latest'  // Push the Docker image to the registry
                    }
                }
            }
        }
    }
}
