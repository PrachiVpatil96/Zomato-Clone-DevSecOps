pipeline {
    agent any

    triggers {
        cron('H/30 * * * *')  // Runs the pipeline every 30 minutes
    }
    
    tools {
        jdk 'JDK 17'  // Ensure this matches the JDK name in Jenkins Global Tool Configuration
        nodejs 'Node.js 16'  // Ensure Node.js is installed in Jenkins
    }
    
    environment {
        NVD_API_KEY = credentials('NVD_API_KEY')  // Fetch API key securely
        SCANNER_HOME = tool 'sonar-scanner'  // Ensure this matches the SonarScanner tool name in Jenkins
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/PrachiVpatil96/Zomato-Clone-DevSecOps.git'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarserver') {  // Replace 'sonar-server' with the actual name from Jenkins config
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato \
                        -Dsonar.host.url=http://65.1.99.154:9000 \
                        -Dsonar.login=squ_a8dd3e743cb06f26e9cbc561b2227a1298e0f8ae


                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonarqube'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit' --apiKey ${NVD_API_KEY}", odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t zomato ."
                       sh "docker tag zomato prachiii123/zomato:1.0 "
                       sh "docker push prachiii123/zomato:1.0 "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image prachiii123/zomato:1.0 > trivy.txt"
            }
        }

        stage('Deploy to container'){
             steps{
              sh 'docker run -d --name zomato -p 3000:3000 prachiii123/zomato:1.0'
          }
      }
    }
}
