pipeline {
  agent any 
  tools {
    maven 'maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
stage ('Check-git-Secrets') {
  steps {
      sh 'rm trufflehog || true'
      sh 'docker run gesellix/trufflehog --json https://github.com/Avadhut8003/Petclinic.git > trufflehog'
      sh 'cat trufflehog'
  }
}
stage ('Software Composition Analysis') {
  steps {
      sh 'rm owasp* || true'
      sh 'wget https://raw.githubusercontent.com/Avadhut8003/Petclinic/master/owasp-dependency-check.sh'
      sh 'chmod +x owasp-dependency-check.sh'
      sh 'bash owasp-dependency-check.sh'
      sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
  }
}
    stage ('SAST') {
      steps {
          withSonarQubeEnv('sonar') {
           sh 'mvn sonar:sonar'
           sh 'cat target/sonar/report-task.txt'
          }
        }
      }

   stage ('Build') {
      steps {
      sh 'mvn clean package'
    }
   }
  stage ('Deplpoy to Tomcat') {
      steps {
      sshagent (['tomcat']) {
      sh 'scp -o ConnectTimeout=60 -o StrictHostKeyChecking=no target/*.war ubuntu@3.108.190.222:/prod/apache-tomcat-10.1.18/webapps/Petclinic.war'

      }
    }
   }
    stage ('DAST') {
   steps {
      sshagent (['zap']) {
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@65.0.100.69 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://3.108.190.222:8080/webapp/" || true'
      }
   }
 }  
  }
}
  
 
    
   
