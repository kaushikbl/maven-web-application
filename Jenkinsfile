pipeline {
    
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3.6.0'
    
    }
    environment{
        SCANNER_HOME = tool 'sonar-server'
        
    }
    
    stages {
        stage('Git-checkout') {
            steps {
                git credentialsId: 'git-jenkins', url: 'https://github.com/kaushikbl/secretsanta-generator.git'
            }
        }
    
        stage('compile') {
            steps {
                sh "mvn clean compile"
            }
        }    
        
        stage('OWASP-scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        } 
        stage('sonarqube') {
            steps {
             withSonarQubeEnv('sonarqube-server') {
             sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=maven-web-application -Dsonar.projectName=maven-web-application"
            }
        }
        }
        
        stage('mvn-Build') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage("Build Docker Image") { 
            steps {
                script {
                    readpom = readMavenPom file: '';
                    
                    artifactversion = readpom.version;
                    
                    timeStamp = Calendar.getInstance().getTime().format("dd-MM-yyyy.HH-mm-ss",TimeZone.getTimeZone('IST'))
                    
                    buildNumber = "${BUILD_NUMBER}"
                    
                    version_number = artifactversion + "." + buildNumber + "." + timeStamp + "."
                    
                    currentBuild.displayName = version_number
                    
                    echo version_number
                    
                    
                     sh "docker build -t kaushikbl/maven-web-application:${version_number} ."
                }
           }
        }
        
        stage("Login & push to DockerHub") {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker-jenkins', toolName: 'docker') {
                   sh "docker push kaushikbl/maven-web-application:${version_number}"
                }
            }
        }
    }
        stage('Deploy app') { 
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-jenkins', toolName: 'docker') {
                    sh "docker run -d --name maven-web-application -p 8080:8082 kaushikbl/maven-web-application:${version_number}"
                }
            }
        }
    }
}
}