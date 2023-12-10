pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "ankur1825/numeric-app:${GIT_COMMIT}"
    applicationURL = "http://devops-azure.eastus.cloudapp.azure.com/"
    applicationURI = "/increment/99"
  }


  stages {
      stage('Build Artifact - Maven') {
            steps {
              sh """
                mvn clean package -DskipTests=true
                #archive 'target/*.jar' 
              """  
            }
        } 
    
      stage('Unit Tests') {
            steps {
              sh """
                mvn test
              """  
            }
        }

      stage('Vulnerability Scan - Docker ') {
          steps {
	    parallel(
	      "Dependancy Scan": {	
            	sh "mvn dependency-check:check"
              },
	      "Trivy Scan": {
		sh" bash trivy-docker-image-scan.sh"	
      	      },
	      "OPA Conftest": {
           	 sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
              }	
	    )
	  }
	}

      stage('Mutation Tests - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
        }

      stage('SonarQube - SAST') {
            steps {
              withSonarQubeEnv('SonarQube') {
                sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devops-azure.eastus.cloudapp.azure.com:9000"
              }
              timeout(time: 2, unit: 'MINUTES') {
                script {
                  waitForQualityGate abortPipeline: true
                }
              }
            }
      }
    
      stage('Docker build and Push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                sh 'printenv'
                sh 'sudo docker build -t ankur1825/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push ankur1825/numeric-app:""$GIT_COMMIT""'
              }
            }
        }

     /* stage('Vulnerability Scan - Kubernetes') {
     	     steps {
        	sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
     	     }
      }*/

     stage('Vulnerability Scan - Kubernetes') {
      steps {
        parallel(
          "OPA Scan": {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          },
          "Kubesec Scan": {
            sh "bash kubesec-scan.sh"
          },
	 // "Trivy Scan": {
         //   sh "bash trivy-k8s-scan.sh"
         // }
        )
      }
    }	

   /* stage('Kubernetes Deployment - DEV') {
       steps {
         withKubeConfig([credentialsId: 'kubeconfig']) {
           sh "sed -i 's#replace#ankur1825/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
           sh "kubectl apply -f k8s_deployment_service.yaml"
         }
       }
    }*/
    
    stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment.sh"
            }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment-rollout-status.sh"
            }
          }
        )
      }
    }

    stage('OWASP ZAP - DAST') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh 'bash zap.sh'
        }
      }
    }
  }  
    post { 
        always { 
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
    }
}  
  
