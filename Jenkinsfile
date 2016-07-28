#!groovy

def commit_id

    
stage 'Build and Publish'
node(){

    // COMPILE AND JUNIT

    //use git checkout
    checkout scm

    sh('pwd')
    sh('git rev-parse HEAD > GIT_COMMIT')
    commit_id=readFile('GIT_COMMIT')
    echo "COMMIT_ID ${commit_id}"

    //ensureMaven()
    //sh 'mvn clean install'
    def mvnHome = tool 'M3'
    sh "${mvnHome}/bin/mvn clean install"

    step $class: 'hudson.tasks.junit.JUnitResultArchiver', testResults: 'target/surefire-reports/*.xml'
    echo "INFO - Ending build phase"

    //Ensure branch is master  $env.BRANCH_NAME
    //if (env.BRANCH_NAME.startsWith("master")) //Only publish to docker if on master branch
    //{
     docker.withServer('unix:///var/run/docker.sock')
     {
        def metarappImage = docker.build "rjstewart/metarapp-jboss-app:${env.BUILD_NUMBER}"

        sh "docker -v"
        //use withDockerRegistry to make sure we are logged in to docker hub registry
        withDockerRegistry(registry: [credentialsId: 'docker-hub-rjstewart']) 
        {
          metarappImage.push()
        }
      }
   //}
}

checkpoint "Build Complete"

stage 'Dev Deploy via Ansible'
node()
{
   echo "Launching Dev Server for ${commit_id}"

   def IP = '127.0.0.1'
   echo "Application Link: ${IP}:8080/metarapp/metars.html"

   stage "Verify Dev Deployment"
   timeout(time: 60, unit: 'SECONDS')
   {
      try
      {
        input message: "Does Dev at ${IP}:8080/metarapp/metars.html look good?"
      }
      catch(Exception e)
      {
         echo "No input provided, resuming build"
      }
   }

   echo "Deployed to Dev"
}

checkpoint "Deployed and Verified at Dev"

stage name: 'Quality analysis and Perfs'
parallel(qualityAnalysis: 
{

    node()
    {
        // RUN SONAR ANALYSIS
        echo "INFO - Starting SONAR"
        //ensureMaven()
        sleep 10
        //sh 'mvn -o sonar:sonar'
        echo "INFO - Ending SONAR"
    }
}, 
performanceTest: 
{

    node()
    {
        // DEPLOY ON PERFS AND RUN JMETER STRESS TEST
        echo "INFO - starting Perf Tests"
        sleep 11
        //sh 'mvn -o jmeter:jmeter'
        echo "INFO - Ending Perf Tests"
    }
}
)  //End QA

stage 'Tear Down DEV'
node()
{
	echo "Tear Down DEV"
	
	sleep 5

    //Call Ansible
   //sh "tower-cli job launch --monitor --job-template=63 --extra-vars=\"commit_id=${commit_id}\""
}

checkpoint "QA analysis complete"

stage "Approval for QA Deploy"
timeout(time: 60, unit: 'SECONDS')
{
   try
   {
      input message: 'Deploy to QA?'
      
      sleep 20
   }
   catch(Exception e)
   {
      echo "No input provided, resuming build"
   }
}

stage 'QA Deploy via Ansible'
node()
{
   echo "Deploying to QA"

   //Call Ansible
   //sh "tower-cli job launch --monitor --job-template=62 --extra-vars=\"commit_id=${commit_id}\" > JOB_OUTPUT"

   //sh 'scripts/get-instance-ip.sh > IP'
   
   sleep 20

   //def IP=readFile('IP')
   //echo "Application Link: ${IP}:8080/metarapp/metars.html"

   stage "Verify QA Deployment"
   timeout(time: 60, unit: 'SECONDS')
   {
      try
      {
        input message: "Does QA at ${IP}:8080/metarapp/metars.html look good?"
      }
      catch(Exception e)
      {
         echo "No input provided, resuming build"
      }
   }
   echo "Deployed to QA"
}

checkpoint "Deployed to QA"

stage 'Tear Down QA'
node()
{
	echo "Tear Down QA"
	timeout(time: 60, unit: 'SECONDS')
	{
	   try
	   {
	      input message: 'QA Test Completed, Tear Down QA'
	   }
	   catch(Exception e)
	   {
	      echo "No input provided, resuming build"
	   }
	}
    //Call Ansible
    //sh "tower-cli job launch --monitor --job-template=63 --extra-vars=\"commit_id=${commit_id}\""
}

checkpoint "QA Testing complete, Ready for Prod Deployment"

   stage 'Approval for Production Deploy'
   timeout(time: 60, unit: 'SECONDS')
   {
      try
      {
	input message: "Deploy to Prod?"
      }
      catch(Exception e)
      {
        echo "No input provided, resuming build"
      }
   }

stage 'Deploy to Production'
node()
{
    echo "Deploying to Prod"

    //wrap([$class: 'OpenShiftBuildWrapper', url: 'https://master.ose.dlt-demo.com:8443', credentialsId: 'DLT-OC', insecure: true]) {

    // sh "oc project prod"
     // Delete existing service:
  //   sh "oc delete all -l app=metarapp-jboss-app"
     sleep 90
     //Hook into oepnshift deployment
  //   sh "oc new-app rjstewart/metarapp-jboss-app:${env.BUILD_NUMBER}"
  //   sh "oc expose svc/metarapp-jboss-app --hostname=metarapp-jboss-app-prod.ose.dlt-demo.com"

     echo "Verify Application Deployed to: http://metarapp-prod.ose.dlt-demo.com/metars_map.html"
   }
  
    echo "Deployed to Prod"
}

/**
 * Deploy Maven on the slave if needed and add it to the path
 */
def ensureMaven() {
    env.PATH = "${tool 'mvn'}/bin:${env.PATH}"
}
