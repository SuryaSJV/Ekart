pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SONARQUBE_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SuryaSJV/Ekart.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }
        stage('owaspt') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                  dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv("sonar") {
                    sh '''$SONARQUBE_HOME/bin/sonar-scanner  \
                    -Dsonar.projectName=EKART \
                    -Dsonar.projectKey=EKART \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('build') {
            steps {
                sh 'mvn package -DskipTests' 
            }
        }
        stage('Nexus') {
           steps {
               withMaven(globalMavenSettingsConfig: 'global-settings-xml') {
                  sh "mvn deploy -DskipTests"
               }
           }
        }
        stage('build and tag docker image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                      sh "docker build -t ekart -f docker/Dockerfile ."
                      sh "docker tag ekart suryasjv/ekart:latest"
                   }
                }
            }
        }
        stage('push') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                      sh "docker push suryasjv/ekart:latest"
                   }
                }
            }
        }
        stage('deploy') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                      sh "docker run -d --name ekart -p 8070:8070 suryasjv/ekart:latest"
                   }
                }
            }
        }
    }
}
