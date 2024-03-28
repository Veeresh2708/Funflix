pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Veeresh2708/Funflix.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonarserver') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker'){
                   //withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                   sh 'printenv'
                   sh 'docker build --build-arg TMDB_V3_API_KEY=351823a305617537180a58b4308a0427 -t netflix:""$GIT_COMMIT"" .'
                   sh 'docker tag netflix:""$GIT_COMMIT"" veereshvanga/ott:$BUILD_NUMBER'
                   sh 'docker push veereshvanga/ott:""$BUILD_NUMBER""'
                   }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nasi101/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d -p 8081:80 veereshvanga/ott:""$BUILD_NUMBER""'
            }
        }
        //stage('Deploy to kubernets'){
        //    steps{
        //        script{
        //            dir('Kubernetes') {
        //                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                        sh 'kubectl apply -f deployment.yml'
        //                       sh 'kubectl apply -f service.yml'
        //               }   
        //           }
        //       }
        //   }
        //}

    }
    //post {
    //always {
    //    emailext attachLog: true,
    //        subject: "'${currentBuild.result}'",
    //        body: "Project: ${env.JOB_NAME}<br/>" +
    //            "Build Number: ${env.BUILD_NUMBER}<br/>" +
    //            "URL: ${env.BUILD_URL}<br/>",
    //        to: 'iambatmanthegoat@gmail.com',                                #change mail here
    //       attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    //   }
    //}
}
