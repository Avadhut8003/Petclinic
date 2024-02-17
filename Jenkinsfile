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
    stage ('Defect_DOJO') {
  steps {  
  sh "curl -X 'POST' \
  'http://65.0.127.46:8080/api/v2/import-scan/' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -H 'X-CSRFTOKEN: VaEpfq63lE0vuwlbQKuUFKwvXym9F2oLBcivezaSXuySe9UmEhsviB12p71EblM0' \
  -F 'product_type_name=' \
  -F 'active=true' \
  -F 'endpoint_to_add=' \
  -F 'verified=true' \
  -F 'close_old_findings=false' \
  -F 'test_title=' \
  -F 'engagement_name=' \
  -F 'build_id=' \
  -F 'deduplication_on_engagement=' \
  -F 'push_to_jira=false' \
  -F 'minimum_severity=Info' \
  -F 'close_old_findings_product_scope=false' \
  -F 'scan_date=' \
  -F 'create_finding_groups_for_all_findings=true' \
  -F 'engagement_end_date=' \
  -F 'environment=' \
  -F 'service=' \
  -F 'commit_hash=' \
  -F 'group_by=' \
  -F 'version=' \
  -F 'tags=' \
  -F 'apply_tags_to_findings=' \
  -F 'api_scan_configuration=' \
  -F 'product_name=' \
  -F 'file=' \
  -F 'auto_create_context=' \
  -F 'lead=' \
  -F 'scan_type=SonarQube Scan' \
  -F 'branch_tag=' \
  -F 'source_code_management_uri=' \
  -F 'engagement='"
  }
}
  
 
    
   
