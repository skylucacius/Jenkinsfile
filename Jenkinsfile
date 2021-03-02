pipeline {
// Replace by specific label for narrowing down to OutSystems pipeline-specific agents. 
// A dedicated agent will be allocated for the entire pipeline run.
  agent any
  parameters {
    // App List Parameters -> automatically filled by LT Trigger plugin
    string(name: 'ApplicationScope', defaultValue: '',
        description: 'Comma-separated list of LifeTime applications to deploy.')
    string(name: 'ApplicationScopeWithTests', defaultValue: '',
        description: 'Comma-separated list of LifeTime applications to deploy (including test applications)')
    string(name: 'TriggeredBy', defaultValue: 'N/A',
        description: 'Name of LifeTime user that triggered the pipeline remotely.')
  }
  options { skipStagesAfterUnstable() }
  environment {
    // Artifacts Folder
    ArtifactsFolder = "Artifacts"
    // LifeTime Specific Variables
    LifeTimeHostname = 'https://mktsystems-lt.outsystemsenterprise.com/'
    LifeTimeAPIVersion = 2
    // Authentication Specific Variables
    AuthorizationToken = credentials('LifeTimeServiceAccountToken')
    // Environments Specification Variables
    /*
    * Pipeline for 5 Environments:
    * DevelopmentEnvironment -> apps.graodegente.com.br
    * RegressionEnvironment -> Where your automated tests will test your applications.
    * AcceptanceEnvironment -> Where you run your acceptance tests of your applications.
    * PreProductionEnvironment -> 'qa.mktsystems.com'
    * ProductionEnvironment -> 'apps.graodegente.com.br'
    */
    DevelopmentEnvironment = 'Development'
    RegressionEnvironment = 'Regression'
    AcceptanceEnvironment = 'Acceptance'
    PreProductionEnvironment = 'Pre-Production'
    ProductionEnvironment = 'Production'
    // Regression URL Specification
    ProbeEnvironmentURL = 'https://qa.mktsystems.com/'
    BddEnvironmentURL = 'https://qa.mktsystems.com/'
    // OutSystems PyPI package version
    OSPackageVersion = '0.3.1'
  }
  stages {
      stage('Get and Deploy Latest Tags') {
        agent any // Replace by specific label for narrowing down to OutSystems pipeline-specific agents
        steps {
            echo 'teste 4'
        //   echo "Pipeline run triggered remotely by '${params.TriggeredBy}' for the following applications (including tests): '${params.ApplicationScopeWithTests}'"
        //   echo "Create ${env.ArtifactsFolder} Folder"
        //   // Create folder for storing artifacts
        //   powershell "mkdir ${env.ArtifactsFolder}"
        //   // Only the virtual environment needs to be installed at the system level
        //   echo 'Install Python Virtual environments'
        //   powershell 'pip install -q -I virtualenv --user'
        //   withPythonEnv('python') {
        //     echo 'Install Python requirements'
        //     // Install the rest of the dependencies at the environment level and not the system level
        //     powershell "pip install -U outsystems-pipeline==\"${env.OSPackageVersion}\""
        //     echo 'Retrieving latest application tags from Development environment...'
        //     // Retrive the Applications and Environment details from the Source environment
        //     powershell "python -m outsystems.pipeline.fetch_lifetime_data --artifacts \"
        //         ${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion}"
        //     echo 'Deploying latest application tags to Regression...'
        //     // Deploy the application list, with tests, to the Regression environment
        //     lock('deployment-plan-REG') {
        //       powershell "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.DevelopmentEnvironment}\" --destination_env \"${env.RegressionEnvironment}\" --app_list \"${params.ApplicationScopeWithTests}\""
        //     }
          }
        }
    //     post {
    //       // It will always store the cache files generated, for observability purposes
    //       always {
    //         dir ("${env.ArtifactsFolder}") {
    //           archiveArtifacts artifacts: "*.cache", onlyIfSuccessful: true
    //           archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
    //           stash name: "manifest",includes: "deployment_data/deployment_manifest.cache", allowEmpty: true
    //         }
    //       }
    //       // If there's a failure, tries to store the Deployment conflicts (if exists), for observability and troubleshooting purposes
    //       failure {
    //         dir ("${env.ArtifactsFolder}") {
    //           archiveArtifacts artifacts: "DeploymentConflicts"
    //         }
    //       }
    //       // Delete artifacts folder to avoid impacts in subsquent builds
    //       cleanup {
    //         dir ("${env.ArtifactsFolder}") {
    //           deleteDir()
    //         }
    //       }
    //     }
    //   }
    }
  }
