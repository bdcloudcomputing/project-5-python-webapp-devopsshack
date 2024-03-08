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
        
         
		stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DP'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
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
                    sh "docker build -t python-webapp . "
                 }
               }
            }
        }

        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker tag python-webapp bpsod10/python-webapp:latest"
                    sh "docker push bpsod10/python-webapp:latest"
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
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-secret-token', namespace: '', restrictKubeConfigAccess: false, serverUrl: 'https://44.200.116.37:6443') {
                sh 'kubectl apply -f python-k8.yaml'
                sh 'kubectl get svc'
                }
            }
        }
        
        
        
        
        }
        
        //  post {
        //     always {
        //         emailext (
        //             subject: "Pipeline Status: ${BUILD_NUMBER}",
        //             body: '''<html>
        //                         <body>
        //                             <p>Build Status: ${BUILD_STATUS}</p>
        //                             <p>Build Number: ${BUILD_NUMBER}</p>
        //                             <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
        //                         </body>
        //                     </html>''',
        //             to: 'bpsod10@gmail.com',
        //             from: 'jenkins@example.com',
        //             replyTo: 'jenkins@example.com',
        //             mimeType: 'text/html'
        //         )
        //     }
        // }
		
		

    
}
