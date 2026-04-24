pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Clean Up Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git CheckOut') {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/SROR7/FullStack-Plogging-App.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Trivy Fs Scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=blogging-app \
                    -Dsonar.projectKey=blogging-app \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=target
                    ''' 
                }
            }
        }
        stage("Build"){
            steps{
                sh 'mvn package'
            }
        }
        stage('Publish The Artifacts'){
            steps{
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Bulid & Tag Docker Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t sror/blogging-app:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.txt sror/blogging-app:${IMAGE_TAG}"
            }
        }
        stage('Push Docker Image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push sror/blogging-app:${IMAGE_TAG}"
                    }
                }
            }
        }
    }
}
