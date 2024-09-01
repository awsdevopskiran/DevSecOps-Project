pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('ACCOUNT_ID')
        AWS_ECR_REPO_NAME = credentials('ECR_REPO')
        AWS_DEFAULT_REGION = 'ap-south-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/awsdevopskiran/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server')  {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        // stage('OWASP FS SCAN') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Image Build") {
            steps {
                script {
                    
                    sh 'docker system prune -f'
                    sh 'docker container prune -f'
                    sh 'docker build --build-arg TMDB_V3_API_KEY=e9af65c0d63a9307139104e294d1b1c6 -t ${AWS_ECR_REPO_NAME} .'
                }
            }
        }
        
        
        stage("ECR Image Pushing") {
            steps {
                script {
                        sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                        // sh 'docker tag ${AWS_ECR_REPO_NAME}:latest ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                        sh 'docker tag netflix-application:latest 718488514653.dkr.ecr.ap-south-1.amazonaws.com/netflix-application:${BUILD_NUMBER}'
                        // sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                        sh 'docker push 718488514653.dkr.ecr.ap-south-1.amazonaws.com/netflix-application:${BUILD_NUMBER}'
                }
            }
        }
        
        stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image 718488514653.dkr.ecr.ap-south-1.amazonaws.com/netflix-application:${BUILD_NUMBER} > trivyimage.txt' 
            }
        }
       
    }
}
