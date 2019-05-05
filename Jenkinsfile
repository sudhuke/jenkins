pipeline {
  agent {
    node {
      label 'us-slave'
    }

  }
  stages {
    stage('Trigger SMKIT Generation on Dev Hudson') {
      steps {
        sh '''concurrent_build=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/concurrentBuild/text\\(\\))
if [[ $concurrent_build == true ]]
then
next_build_no=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/nextBuildNumber/text\\(\\))
check_queue=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/inQueue/text\\(\\))
if [[ $check_queue == true ]]
then
next_build_no=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/queueItem/id/text\\(\\))
sleep 5
reason=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/queueItem/why/text\\(\\))
while [[ $reason == "Waiting for next available executor" ]] 
do
echo "Build submission is waiting for $reason"
reason=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/queueItem/why/text\\(\\))
done
fi
fi
next_build_no=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/nextBuildNumber/text\\(\\))
echo "Submitting the job for smkit generation"
job_enabled=$(curl -X GET ${SMKIT_GENERATE_JOB_URL}/api/xml?xpath=/*/buildable/text\\(\\))
if [[ $job_enabled == "false" ]]
then
echo "${SMKIT_GENERATE_JOB_URL} is disabled. Exiting.."
exit 1
fi

curl  -X POST "${SMKIT_GENERATE_JOB_URL}/buildWithParameters?PAAS_BRANCH=${PAAS_BRANCH}&EMAIL=${EMAIL}"
if [[ $? -ne 0 ]]
then
echo "Problem in triggering the ${SMKIT_GENERATE_JOB_URL}"
exit 1
fi
echo "Build no to be monitored is: $next_build_no"
echo "${SMKIT_GENERATE_JOB_URL}/$next_build_no" > joburl.txt
printf "topic_branch=${PAAS_BRANCH}\\nEMAIL=${EMAIL}" > job_params.txt
'''
      }
    }
    stage('Monitor SMKIT Generation on Dev Hudson') {
      steps {
        sh '''smkit_file=$(curl -X GET ${smkit_generate_job_url}/consoleText | grep -w "SMKIT_ZIPFILE"  | cut -d"=" -f2)
if [[ -f ${smkit_file} ]]
then
echo "$smkit_file exists"
printf "EMAIL_ID=${EMAIL}\\nBRANCH=${PAAS_BRANCH}\\nSMKIT_PATH=${smkit_file}" > ${WORKSPACE}/smtrigger.properties
else
echo "SMKIT generated by ${smkit_generate_job_url} is not found"
exit 1
fi'''
      }
    }
    stage('Trigger SM Install on DTE') {
      steps {
        sh '''if [[ ! -f $SMKIT_PATH ]]
then
echo "SMKIT zip file should be available at $SMKIT_PATH"
exit 1
fi

if [[ ` basename $CONFIG_PROPERTIES_FILE` == "SmConfig.properties" ]]
then
 if [[ $releaseVersion -lt 1741 ]]
 then
 CONFIG_PROPERTIES_FILE=$CONFIG_PROPERTIES_FILE.$psm_release
 echo "Using $CONFIG_PROPERTIES_FILE as CONFIG_PROPERTIES_FILE"
 else
 echo "Using $CONFIG_PROPERTIES_FILE as CONFIG_PROPERTIES_FILE"
 fi
fi

if [[ -z $psm_release ]]
then
if [[ -z $SM_KIT_TOKEN ]]
then
psm_release=smtoken
else
psm_release=$SM_KIT_TOKEN
fi
fi


touch ${WORKSPACE}/custom_job_info.$BUILD_NUMBER
echo "$psm_release::$BRANCH::$SITE::$HUDSON_URL::$SCHEDULE" > ${WORKSPACE}/custom_job_info.$BUILD_NUMBER
cat ${WORKSPACE}/custom_job_info.$BUILD_NUMBER

if [[ -f $SMKIT_PATH ]]
then
echo "Triggering psm dte job"
export JDK15HOME=/usr
/usr/local/packages/aime/dte/DTE/bin/jobReqAgent -topoid 79254 -p LINUX.X64 -s %ADE_VIEW_ROOT%/cloudpltfsvc/src/oracle.jaas -u sudke -l ESE_NOENV_GENERIC_RELEASE -noalways=true -e sudharshan.ke@ORACLE.COM -maxduration 100 JAVA_HOME=/net/adcnas418/export/farm_fmwqa/java/linux64/jdk7u51_b13/ M3_HOME=/net/slc02pnd/scratch/aime1/apache-maven-3.0.4 ANT_HOME=/net/slc02pnd/scratch/aime1/apache-ant-1.8.2 MATS_FILES=$MATS_FILES CONFIG_PROPERTIES_FILE=$CONFIG_PROPERTIES_FILE SM_KIT_TOKEN=$psm_release SM_KIT_ARCHIVE=$SMKIT_PATH JOB_INFO=/net/`hostname -f`/${WORKSPACE}/custom_job_info.$BUILD_NUMBER PSM_VERSION=$VERSION | tee $WORKSPACE/trigger_dte.log

echo "SMKIT_PATH=${SMKIT_PATH}" > ./smkit_install.txt
echo "PAAS_BRANCH=${BRANCH}" >> ./smkit_install.txt
echo "EMAIL=${EMAIL_ID}" >> ./smkit_install.txt



DTE_JOBID=`cat $WORKSPACE/trigger_dte.log | grep SM_With_Hudson_Mats | cut -d":" -f1`
 if [[ -z $DTE_JOBID ]]
 then
 echo "unable to submit dtejob due to farm issue"
 exit 1
 fi
echo "$DTE_JOBID" >  $WORKSPACE/dte_jobid.txt
echo "DTE_JOBID=${DTE_JOBID}" >> ./smkit_install.txt

else
echo "$SMKIT_PATH not exists.. quitting"
exit 1
fi
'''
      }
    }
    stage('Monitor SM Install on DTE') {
      steps {
        sh 'bash /net/sca00jpl/scratch/paasqa/dtejob/scripts/psm_dte_job_status.sh $DTE_JOB_ID'
      }
    }
  }
}