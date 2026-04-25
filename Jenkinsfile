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
                        sh "docker build -t sror/blogging-app:${IMAGE_TAG} ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.txt --image-src docker sror/blogging-app:${IMAGE_TAG}"
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
        stage('k8s-deploy'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'blogging-cluster', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://706C5327DA7E76231AB1F8B80438529C.gr7.eu-north-1.eks.amazonaws.com') {
                    sh "kubectl apply -f deployment-service.yaml"
                    sleep 20
                }
            }
        }
        stage('Verify The Deployment'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'blogging-cluster', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://706C5327DA7E76231AB1F8B80438529C.gr7.eu-north-1.eks.amazonaws.com') {
                    sh "kubectl get pods"
                    sh "kubectl get svc"
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: 'm7mdsror0@gmail.com',
                subject: "Deployment Successful - blogging-app #${BUILD_NUMBER}",
                body: """
                    <html>
                        <body>
                            <h2 style="color: green;"> Deployment Successful</h2>
                            <table border="1" cellpadding="6" cellspacing="0">
                                <tr><td><b>Project</b></td><td>blogging-app</td></tr>
                                <tr><td><b>Build Number</b></td><td>#${BUILD_NUMBER}</td></tr>
                                <tr><td><b>Image Tag</b></td><td>sror/blogging-app:${IMAGE_TAG}</td></tr>
                                <tr><td><b>Cluster</b></td><td>blogging-cluster</td></tr>
                                <tr><td><b>Namespace</b></td><td>webapps</td></tr>
                                <tr><td><b>Duration</b></td><td>${currentBuild.durationString}</td></tr>
                                <tr><td><b>Build URL</b></td><td><a href="${BUILD_URL}">${BUILD_URL}</a></td></tr>
                            </table>
                            <br/>
                            <p>The application has been successfully deployed to EKS. </p>
                        </body>
                    </html>
                """,
                mimeType: 'text/html',
                attachmentsPattern: 'image.txt,fs.html'
            )
        }
        failure {
            emailext(
                to: 'm7mdsror@gmail.com',
                subject: "Deployment Failed - blogging-app #${BUILD_NUMBER}",
                body: """
                    <html>
                        <body>
                            <h2 style="color: red;"> Deployment Failed</h2>
                            <table border="1" cellpadding="6" cellspacing="0">
                                <tr><td><b>Project</b></td><td>blogging-app</td></tr>
                                <tr><td><b>Build Number</b></td><td>#${BUILD_NUMBER}</td></tr>
                                <tr><td><b>Failed Stage</b></td><td>${env.STAGE_NAME}</td></tr>
                                <tr><td><b>Duration</b></td><td>${currentBuild.durationString}</td></tr>
                                <tr><td><b>Build URL</b></td><td><a href="${BUILD_URL}">${BUILD_URL}</a></td></tr>
                            </table>
                            <br/>
                            <p>Please check the Jenkins console output for details.</p>
                        </body>
                    </html>
                """,
                mimeType: 'text/html'
            )
        }
    }
}