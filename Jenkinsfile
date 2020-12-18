// https://github.com/camunda/jenkins-global-shared-library
// https://github.com/camunda/cambpm-jenkins-shared-library
@Library(['camunda-ci', 'cambpm-jenkins-shared-library']) _

def failedStageTypes = []

pipeline {
  agent none
  options {
    buildDiscarder(logRotator(numToKeepStr: '5')) //, artifactNumToKeepStr: '30'
    copyArtifactPermission('*');
  }
  parameters {
    string defaultValue: 'cambpm-ee-main-pr/pipeline-master', description: 'The name of the EE branch to run the EE pipeline on, e.g. cambpm-ee-main/PR-333', name: 'EE_BRANCH_NAME'
  }
  stages {
        stage('aurora') {
          agent {
            node {
              label 'aws-ecs-aurora-slave'
            }
          }
          steps {
            withCredentials([usernamePassword(usernameVariable: 'AURORA_POSTGRES_USR', passwordVariable: 'AURORA_POSTGRES_PSW', credentialsId: 'aws_aurora_postgresql')]) {
              sh '.ci/scripts/create-aurora-db.sh'
              cambpmRunMaven('distro/sql-script/', 'install -Pcheck-sql,postgresql -Ddatabase.host=${PGHOST} -Ddatabase.port=${PGPORT} -Ddatabase.name="${PGDATABASE}" -Ddatabase.username=${PGUSER} -Ddatabase.password=${PGPASSWORD} ', runtimeStash: true, archiveStash: true, qaStash: true)
            }
          }
          post {
            always {
              cambpmPublishTestResult();
               withCredentials([usernamePassword(usernameVariable: 'AURORA_POSTGRES_USR', passwordVariable: 'AURORA_POSTGRES_PSW', credentialsId: 'aws_aurora_postgresql')]) {
                 sh '.ci/scripts/delete-aurora-db.sh'
               }
            }
          }
        }
  }
  post {
    changed {
      script {
        if (!agentDisconnected()){
          cambpmSendEmailNotification()
        }
      }
    }
    always {
      script {
        if (agentDisconnected()) {// Retrigger the build if the slave disconnected
          build job: currentBuild.projectName, propagate: false, quietPeriod: 60, wait: false
        }
      }
    }
  }
}
