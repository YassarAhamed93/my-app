pipeline {
    agent any
    
    stages {
        stage('SCM Checkout') {
            steps {
                git url: 'https://github.com/YassarAhamed93/my-app.git'
            }
        }
        
        stage('maven-buildstage') {
            steps {
                script {
                    def mvnHome = tool name: 'maven3', type: 'maven'   
                    sh "${mvnHome}/bin/mvn package"
                    sh 'mv target/myweb*.war target/newapp.war'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def mvnHome = tool name: 'maven3', type: 'maven'
                    withSonarQubeEnv('sonar') { 
                      sh "${mvnHome}/bin/mvn sonar:sonar"
                    }
                }
            }
        }        
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t yassarahamed/myweb:0.0.2 .'
            }
        }
        
        stage('Docker Image Push') {
            steps {
                withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                    sh "docker login -u yassarahamed -p ${dockerPassword}"
                }
                sh 'docker push yassarahamed/myweb:0.0.2'
            }
        }
        stage('Nexus Image Push') {
            steps {
                sh "docker login -u admin -p admin123 13.232.133.49:8083"
                sh "docker tag yassarahamed/myweb:0.0.2 13.232.133.49:8083/yasu:0.0.2"
                sh 'docker push 13.232.133.49:8083/yasu:0.0.2'
            }
        }
        stage('Remove Previous Container') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'docker rm -f tomcattest'
                }
            }
        }

        stage('Docker Deployment') {
            steps {
                sh 'docker run -d -p 8090:8080 --name tomcattest yassarahamed/myweb:0.0.2' 
            }
        }
    }
}
