pipeline {
  agent {
    node {
      label 'us-slaves'
    }
  }
   parameters {
        string(defaultValue: 'sudharshan.ke@oracle.com', description: '', name: 'EMAIL')
        string(defaultValue: '', description: 'Topic branch to be built', name: 'BRANCH')
        string(defaultValue: 'http://slc14rif.us.oracle.com:8080/hudson/cis/job/psm-dev-pipeline-smkit-generation', description: 'Dev Hudson URL', name: 'SMKIT_GENERATE_JOB_URL')

    }

  stages {
    stage('Trigger SMKIT Generation on Dev Hudson') {
      steps {
        sh 'bash /net/sca00jpl/scratch/paasqa/dtejob/scripts/psm_dev_pipline_trigger_smkit_build.sh'
      }
    }
    stage('Monitor SMKIT Generation on Dev Hudson') {
      steps {
        sh 'bash /net/sca00jpl/scratch/paasqa/dtejob/scripts/psm_dev_pipline_monitor_smkit_build.sh'
      }
    }
    stage('Trigger SM Install on DTE') {
      steps {
        sh 'bash /net/sca00jpl/scratch/paasqa/dtejob/scripts/psm_dev_pipline_trigger_smkit_dteinstall.sh'
      }
    }
    stage('Monitor SM Install on DTE') {
      steps {
        sh 'bash /net/sca00jpl/scratch/paasqa/dtejob/scripts/psm_dte_job_status.sh $DTE_JOB_ID'
      }
    }
  }
}
