pipeline {
  agent any
    tools { 
        maven 'localMaven' 
    }
    stages{
      stage ('Build'){
        steps{
	  echo 'Maven Build'
          sh 'mvn -f pom.xml clean install deploy'
        }
      }

      stage ('SonarQube'){
          steps{
              sh 'mvn sonar:sonar -Dsonar.host.url=http://127.0.0.1:9000/sonar'
              }
           }


      stage('archive') {
      steps {
        parallel(
          "Junit": {
            junit 'target/surefire-reports/*.xml'
            
          },
          "Archive": {
            archiveArtifacts(artifacts: 'target/greenhouse-*.war', onlyIfSuccessful: true, fingerprint: true)
          }
        )
      }
    }
 
      stage ('Build Image and Push to Dockerhub'){  
        steps{ 
           sh """
           sudo docker image build -t tomcat-application .
           sudo docker tag tomcat-application kunalborkar/tomcat-application
           sudo docker push kunalborkar/tomcat-application
           """
        }
    }
    stage ('Deploy on EC2'){
     steps{
        sh """
           ssh -tt ec2-user@34.216.1.103 sudo docker container run -d -p 9999:8080 --name app kunalborkar/tomcat-application
           """
        }
    }
}
}

