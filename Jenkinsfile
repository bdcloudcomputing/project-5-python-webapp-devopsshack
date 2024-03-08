pipeline {
    agent any
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git-checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bdcloudcomputing/project-5-python-webapp-devopsshack.git'
            }
        }

       // this will call the function 'test' from the makefile in repo  
        stage('Test') {
            steps {
               sh "make test"
            }
        }
        
         
		stage('trivy filescan') {
            steps {
                    sh "trivy fs --format json -o trivy-repofilesscan-report.json ."
            }
        }


        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar-server'){
                   sh ' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=python-webapp  -Dsonar.projectKey=python-webapp '
               }
            }
        }


         stage('Docker Build') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "make image"
                 }
               }
            }
        }

        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                     sh "make push"
                 }
               }
            }
        }
        
        	 
        stage('Docker Image Scan') {
            steps {
               sh "trivy image --format table -o trivy-report.html bpsod10/python-webapp:latest"
            }
        }


        stage('k8s deploy') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-secret-token', namespace: 'devops', restrictKubeConfigAccess: false, serverUrl: 'https://3.231.94.6:6443') {
                sh 'kubectl apply -f python-k8.yaml'
                sh 'kubectl get svc'
                }
            }
        }
        
        
        

		
		
    }
    
}
