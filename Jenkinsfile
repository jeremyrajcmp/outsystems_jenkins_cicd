pipeline {
  agent any // Replace by specific label for narrowing down to OutSystems pipeline-specific agents. A dedicated agent will be allocated for the entire pipeline run.
  parameters {
    // Pipeline parameters are automatically filled by LT Trigger plugin
    string(name: 'TriggerManifest', defaultValue: '', description: 'Trigger manifest artifact (in JSON format) for the current pipeline run.')
    string(name: 'TriggeredBy', defaultValue: 'N/A', description: 'Name of LifeTime user that triggered the pipeline remotely.')
  }
  options { skipStagesAfterUnstable() }
  environment {
    // Artifacts Folder
    ArtifactsFolder = "Artifacts"
    // LifeTime Specific Variables
    LifeTimeHostname = 'showcase-lt.outsystemsdevopsexperts.com'
    LifeTimeAPIVersion = 2
    // Authentication Specific Variables
    AuthorizationToken = credentials('LifeTimeServiceAccountToken_Showcase')
    ADHostname = "archpp.outsystems.com"
    ADActivationCode = "Y4B.U3L.IEY.OSR.VNB.AZS.E1I.QUT"
    AIMentor_API_Key = credentials("AIMentor_Api_Key")
    // Environments Specification Variables
    /*
    * Pipeline for 5 Environments:
    *   DevelopmentEnvironment    -> Where you develop your applications. This should be the default environment you connect with service studio.
    *   RegressionEnvironment     -> Where your automated tests will test your applications.
    *   AcceptanceEnvironment     -> Where you run your acceptance tests of your applications.
    *   PreProductionEnvironment  -> Where you prepare your apps to go live.
    *   ProductionEnvironment     -> Where your apps are live.
    */
    DevelopmentEnvironmentLabel = 'DEV'
    RegressionEnvironmentLabel = 'REG'
    AcceptanceEnvironmentLabel = 'ACC'
    PreProductionEnvironmentLabel = 'PRE'
    ProductionEnvironmentLabel = 'PRD'
    // Regression URL Specification
    ProbeEnvironmentURL = 'https://showcase-reg.outsystemsdevopsexperts.com/'
    BddEnvironmentURL = 'https://showcase-reg.outsystemsdevopsexperts.com/'
    // Test Management API Endpoints
    RegressionTestJobURL = 'https://showcase-dev.outsystemsdevopsexperts.com/TestManagement_CS/rest/v1/TriggerTestRun?ReportType=junit&Env=REG&JobKey=demo-reg-job'
    // OutSystems PyPI package version
    OSPackageVersion = '0.5.0'
  }
  stages {
    stage('Install Python Dependencies') {
      steps {
        // Create folder for storing artifacts
        sh script: "mkdir ${env.ArtifactsFolder}", label: 'Create artifacts folder'
        // Write trigger manifest content to a file
        writeFile file: "${env.ArtifactsFolder}/trigger_manifest.cache", text: "${params.TriggerManifest}"
          // Create folder for storing artifacts
        // Write trigger manifest content to a file
      //  writeFile file: "${env.ArtifactsFolder}/manifest.file", text: ""
        // Only the virtual environment needs to be installed at the system level
        sh script: 'pip3 install -q -I virtualenv --user', label: 'Install Python virtual environments'
        // Install the rest of the dependencies at the environment level and not the system level
        withPythonEnv('python3') {
          sh script: "pip3 install -U outsystems-pipeline==\"${env.OSPackageVersion}\"", label: 'Install required packages'
        }
      }
    }
    
    stage('Validate Tech Debt') {
      steps {     
        withPythonEnv('python3') {
          // check AD tech debt
          sh script :"python3 -m outsystems.pipeline.fetch_tech_debt --artifacts \"${env.ArtifactsFolder}\" --ad_hostname ${env.ADHostname} --activation_code ${env.ADActivationCode} --api_key ${env.AIMentor_API_Key} --manifest_file \"${env.ArtifactsFolder}/trigger_manifest.cache\""
          sh script :"python3 -u techdebt_validation.py -m \"${env.ArtifactsFolder}/trigger_manifest.cache\" -d \"${env.ArtifactsFolder}/techdebt_data/\"", returnStatus: true, label: 'Run tech debt validation'
        }        
      }
    }
  }
  post {
    // It will always store the cache files generated, for observability purposes, and notifies the result
    always {
      dir ("${env.ArtifactsFolder}") {
        archiveArtifacts artifacts: "**/*.cache"
      }
    }
    // If there's a failure, tries to store the Deployment conflicts (if exists), for troubleshooting purposes
    failure {
      dir ("${env.ArtifactsFolder}") {
        archiveArtifacts artifacts: 'DeploymentConflicts', allowEmptyArchive: true
      }
    }
    // Delete artifacts folder content
    cleanup {      
      dir ("${env.ArtifactsFolder}") {
        deleteDir()
      }
    }
  }
}
