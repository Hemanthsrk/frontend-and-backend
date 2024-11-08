pipeline {
    agent any
    
    tools{
        maven "Maven-3.9.6"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/ashokitschool/maven-web-app.git'
            }
        }
        stage('Maven Build') {
            steps {
               sh 'mvn clean package'
            }
        }
        
        stage('Code Review') {
            steps{
                withSonarQubeEnv('sonar-9.9.3') {
        		   sh "mvn sonar:sonar"
        	    }
            }
        }
        
        stage("Nexus Upload"){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: '01-maven-web-app', classifier: '', file: 'target/maven-web-app.war', type: 'war']], credentialsId: 'nexus-server', groupId: 'in.ashokit', nexusUrl: '3.108.63.133:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'ashokit-snapshot-repo', version: '3.0-SNAPSHOT'
            }
        }
        
        stage('Docker Image') {
            steps {
               sh 'docker build -t ashokit/mavenwebapp .'
            }
        }
        stage('Image Push') {
            steps {
                withCredentials([string(credentialsId: 'docker-acc-pwd-id', variable: 'dockerpwd')]) {
                    sh "docker login -u ashokit -p ${dockerpwd}"
                	sh "docker push ashokit/mavenwebapp"
                }
            }
        }
        
        stage('K8S Deploy') {
            steps {
               sh 'kubectl apply -f maven-web-app-deploy.yml'
            }
        }
    }
}

