pipeline {
  agent {
    any
  }

  options {
    disableConcurrentBuilds()
    timestamps()
    ansiColor('xterm')
  }
  
  parameters {
    choice(name: 'DEPLOYMENT_ENV', choices: ['dev01', 'dev02', 'dev03', 'dev04', 'dev05'], description: 'Environment to deploy' )
    choice(name: 'RESTORE_DATA_BKP', choices: ['no', 'yes'], description: 'Restore data from jira-data filesystem backup' )
    choice(name: 'RESTORE_DB_BKP', choices: ['no', 'yes'], description: 'Restore data from database backup' )
    choice(name: 'OBFUSCATE_DB_PRO', choices: ['yes', 'no'], description: 'Obfuscate data from production' )
  }
  
  environment {
    JIRA_VERSION = "8.1.1"
    // DEPLOYMENT_ENV = "${params.DEPLOYMENT_ENV?:'dev03'}" //
    DEPLOYMENT_ENV = get_deployment_env() 
    SLACK_CHANNEL = "#collaboration-jenkins"
    DOCKER_IMAGE = "bin.private.zooplus.net/collaboration/jira/jira:$DEPLOYMENT_ENV"
    TF_VERSION = "0.12.24"
    COLLABORATION_CREDS = credentials('collaboration')
    RDS_ENDPOINT = "atlasdevmx.caiac2ctdqte.eu-central-1.rds.amazonaws.com"
    DATABASE_BKP_FILE = "zp-jira-01_backup.tar.bz2"
    DATABASE_BKP_DEST = "/tmp"
    DATABASE_SQL_FILE = "zp-jira-01_backup.sql"
    DATABASE_BKP_BUCKET = "dev-bkps"
    SCHEMA_ORIG = "jira"
    SCHEMA_DEST = "jira_${DEPLOYMENT_ENV}"
    APPLICATION_DATA_BUCKET = "prod-jiradata"
    APPLICATION_DATA_HOST_DIR = "/srv/jira/jira-data"
    RESTORE_DATA_BKP = "${params.RESTORE_DATA_BKP}"
    RESTORE_DB_BKP = "${params.RESTORE_DATA_BKP == 'yes'?'yes':params.RESTORE_DB_BKP}"
    OBFUSCATE_DB_PRO = "~${params.OBFUSCATE_DB_PRO}"
  }

  stages {

    stage('Confirm deployment') {
      steps {
          timeout(time: 3000, unit: 'SECONDS') {
          input message: "Confirm deployment in $DEPLOYMENT_ENV?", ok: 'Yes'
        }
      }         
    }

    stage("Check Preconditions") {
      steps {
        sh 'printenv'
        script {
            if(!"${BRANCH_NAME}".startsWith("development") ) {
              currentBuild.result = 'ABORTED'
              error('Stopping due to error in branch naming')
          }
        }        
      }      
    }

    // stage('Test Credentials') {
    //   steps {
    //     withCredentials([usernamePassword(credentialsId: 'collaboration', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
    //       echo "${USERNAME} ${PASSWORD}"
    //     }
    //   }        
    // }

    // stage('usernamePassword') {
    //   steps {
    //     script {
    //       withCredentials([
    //         usernamePassword(credentialsId: 'collaboration',
    //           usernameVariable: 'USERNAME',
    //           passwordVariable: 'PASSWORD')
    //       ]) {
    //         print 'username=' + USERNAME + 'password=' + PASSWORD

    //         print 'username.collect { it }=' + USERNAME.collect { it }
    //         print 'password.collect { it }=' + PASSWORD.collect { it }
    //         println(hudson.util.Secret.decrypt(PASSWORD))
    //       }
    //     }
    //   }
    // }

  }
}

def get_deployment_env() {
  return env.BRANCH_NAME.split('-')[1]
}