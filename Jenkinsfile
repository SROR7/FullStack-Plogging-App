pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner '
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
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
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
    }
}
