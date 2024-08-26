
pipeline{
  agent any
   environment {
        DOCKER_IMAGE = 'tomcat'
        CONTAINER_NAME = 'tomcat-1'
        WAR_FILE = 'webapp.war'
    }
    stages{
      stage("cleanws"){
        steps{
          cleanWs()
        }
      }
      stage("checkout"){
        steps{
         git branch: 'main', url: 'https://github.com/Hajiva/greedygame-docs.git'
        }
      }
      stage("build"){
        steps{
        sh "mvn clean install"
        }
      }
      stage("code quantity"){
        steps{
          withSonarQubeEnv("sonarqube"){
            sh "mvn sonar:sonar"
          }
        }
      }
       stage("quality gate"){
        steps{
           waitForQualityGate 'sonarqube'
        }
      }
      stage("deploy"){
        steps{
          sshagent(['tomcat-server']) {
            sh "scp -o StrictHostKeyChecking=no webapp/target/webapp.war ec2-user@3.137.192.3:/opt/tomcat/webapps"
          }
        }
      }
     stage('Deploy to Tomcat') {
            steps {
                script {
                    // Pull the latest Tomcat image
                    sh "docker pull ${DOCKER_IMAGE}"
                    
                    // Remove any existing container with the same name
                    sh "docker rm -f ${CONTAINER_NAME} || true"
                    
                    // Run a new Tomcat container
                    sh "docker run -d --name ${CONTAINER_NAME} -p 8082:8080 ${DOCKER_IMAGE}"
                    
                    // Copy the WAR file to the Tomcat container
                    sh "docker cp webapp/target/${WAR_FILE} ${CONTAINER_NAME}:/usr/local/tomcat/webapps/"
              }
          }
      }
  }
}

