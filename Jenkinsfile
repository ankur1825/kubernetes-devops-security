pipeline {
  agent any

  stages {
      stage('Build Artifact - Maven') {
            steps {
              sh """
                export PATH=${M2_HOME}/bin:${PATH}
                mvn clean package -DskipTests=true
                archive 'target/*.jar' //so that they can be downloaded later
              """  
            }
        }   
      stage('Unit Tests') {
            steps {
              sh """
                export PATH=${M2_HOME}/bin:${PATH}
                mvn test
              """  
            }
        }  
    }
}