pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    triggers {
        // pollSCM('* * * * *') // Ceci semble ne pas avoir fonctionner, vérifie toutes les minutes pour détecter les modifications, 
        githubPush()
    }
    properties([pipelineTriggers([githubPush()])])
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace from jenkinsfile'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/a7xr/Netflix-mrcloudbook'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        // stage("quality gate"){
        //    steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
        //         }
        //     }
        // }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh "docker build --build-arg TMDB_V3_API_KEY=39b767e94099fd31177dfeac1b64f682 -t netflix ."
                        sh "docker tag netflix rx7a/netflix:latest "
                        sh "docker push rx7a/netflix:latest "
                    }
                }
            }
        }
        stage('Deploy to kubernetes MMT'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rx7a/netflix:latest > trivyimage.txt"
            }
        }
    }
}