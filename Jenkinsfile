pipeline {
  agent any

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
            post {
              always {
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
            }
          }
        }
    
      stage('Docker build and Push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
              sh """
                printenv
                sudo docker build -t ankur1825/numeric-app:""$GIT_COMMIT"" .
                docker push ankur1825/numeric-app:""$GIT_COMMIT""
              """  
              }
            }
        }
    }
}  
