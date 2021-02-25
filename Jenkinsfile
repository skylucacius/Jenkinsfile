pipeline {
  agent any // Replace by specific label for narrowing down to OutSystems pipeline-specific agents. A dedicated agent will be allocated for the entire pipeline run.
  parameters {
    // App List Parameters -> automatically filled by LT Trigger plugin
    string(name: 'ApplicationScope', defaultValue: '', description: 'Comma-separated list of LifeTime applications to deploy.')
    string(name: 'ApplicationScopeWithTests', defaultValue: '', description: 'Comma-separated list of LifeTime applications to deploy (including test applications)')
    string(name: 'TriggeredBy', defaultValue: 'N/A', description: 'Name of LifeTime user that triggered the pipeline remotely.')
  }
  options { skipStagesAfterUnstable() }
  environment {
    // Artifacts Folder
    ArtifactsFolder = "Artifacts"
    // LifeTime Specific Variables
    LifeTimeHostname = 'https://mktsystems-lt.outsystemsenterprise.com/'
    LifeTimeAPIVersion = 2
    // Authentication Specific Variables
    AuthorizationToken = credentials('eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJsaWZldGltZSIsInN1YiI6Ik4yTmlZV1kzTldVdFpXSXlZeTAwWkRWaUxXRTRaakl0Tm1FNVpqQmpNVE0yTVRNeSIsImF1ZCI6ImxpZmV0aW1lIiwiaWF0IjoiMTYxMzc2MjYwNiIsImppdCI6IjV5cGJwWG5pd1QifQ==.2TUnheqK0Xrd+st9nEiBBF4Po0h3Qrt4pPB+P7VKzhA=')
    // Environments Specification Variables
    /*
    * Pipeline for 5 Environments:
    * DevelopmentEnvironment -> apps.graodegente.com.br
    * RegressionEnvironment -> Where your automated tests will test your applications.
    * AcceptanceEnvironment -> Where you run your acceptance tests of your applications.
    * PreProductionEnvironment -> 'qa.mktsystems.com'
    * ProductionEnvironment -> 'apps.graodegente.com.br'
    */
    DevelopmentEnvironment = 'apps.graodegente.com.br'
    RegressionEnvironment = 'Testing'
    AcceptanceEnvironment = 'Testing'
    PreProductionEnvironment = 'Testing'
    ProductionEnvironment = 'apps.graodegente.com.br'
    // Regression URL Specification
    ProbeEnvironmentURL = 'https://qa.mktsystems.com/'
    BddEnvironmentURL = 'https://qa.mktsystems.com/'
    // OutSystems PyPI package version
    OSPackageVersion = '0.3.1'
  }
  stages {
    stage('Install Python Dependencies') {
      steps {
        echo "Create ${env.ArtifactsFolder} Folder"
        // Create folder for storing artifacts
        powershell "mkdir ${env.ArtifactsFolder}"
        // Only the virtual environment needs to be installed at the system level
        echo "Install Python Virtual environments"
        powershell 'pip install -q -I virtualenv --user'
        // Install the rest of the dependencies at the environment level and not the system level
        withPythonEnv('python') {
          echo "Install Python requirements"
          powershell "pip install -U outsystems-pipeline==\"${env.OSPackageVersion}\""
        }
      }
    }
    stage('Get and Deploy Latest Tags') {
      steps {
        withPythonEnv('python') {
          echo "Pipeline run triggered remotely by '${params.TriggeredBy}' for the following applications (including tests): '${params.ApplicationScopeWithTests}'"
          echo 'Retrieving latest application tags from Development environment...'
          // Retrive the Applications and Environment details from the Source environment
          powershell "python -m outsystems.pipeline.fetch_lifetime_data --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion}"
          echo 'Deploying latest application tags to Regression...'
          // Deploy the application list, with tests, to the Regression environment
          lock('deployment-plan-REG') {
            powershell "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.DevelopmentEnvironment}\" --destination_env \"${env.RegressionEnvironment}\" --app_list \"${params.ApplicationScopeWithTests}\""
          }
        }
      }
      post {
        // It will always store the cache files generated, for observability purposes
        always {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*.cache", onlyIfSuccessful: true
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
        // If there's a failure, tries to store the Deployment conflicts (if exists), for observability and troubleshooting purposes
        failure {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "DeploymentConflicts"
          }
        }
      }
    }
    stage('Run Regression') {
      when {
        expression { return params.ApplicationScope != params.ApplicationScopeWithTests }
      }
      steps {
        withPythonEnv('python') {
          echo 'Generating URLs for BDD testing...'
          // Generate the URL endpoints of the BDD tests
          powershell "python -m outsystems.pipeline.generate_unit_testing_assembly --artifacts \"${env.ArtifactsFolder}\" --app_list \"${params.ApplicationScopeWithTests}\" --cicd_probe_env ${env.ProbeEnvironmentURL} --bdd_framework_env ${env.BddEnvironmentURL}"
          echo "Testing the URLs and generating the JUnit results XML..."
          // Run those tests and generate a JUNIT test report
          powershell(script: "python -m outsystems.pipeline.evaluate_test_results --artifacts \"${env.ArtifactsFolder}\"", returnStatus: true)
        }
      }
      post {
        always {
          withPythonEnv('python') {
            echo "Publishing JUnit test results..."
            junit(testResults: "${env.ArtifactsFolder}\\junit-result.xml", allowEmptyResults: true)
          }
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
      }
    }
    stage('Accept Changes') {
      steps {
        withPythonEnv('python') {
          echo 'Deploying latest application tags to Acceptance...'
          // Deploy the application list, without tests, to the Acceptance environment
          lock('deployment-plan-ACC') {
            powershell "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.RegressionEnvironment}\" --destination_env \"${env.AcceptanceEnvironment}\" --app_list \"${params.ApplicationScope}\" --manifest \"${env.ArtifactsFolder}\\deployment_data\\deployment_manifest.cache\""
          }
        }
        // Define milestone before approval gate to manage concurrent builds
        milestone(ordinal: 40, label: 'before-approval')
        // Wrap the confirm option in a timeout to avoid hanging Jenkins forever
        timeout(time:1, unit:'DAYS') {
          input 'Accept changes and deploy to Production?'
        }
        // Discard previous builds that have not been accepted yet
        milestone(ordinal: 50, label: 'after-approval')
      }
      post {
        always {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
        failure {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "DeploymentConflicts"
          }
        }
      }
    }
    stage('Deploy Dry-Run') {
      steps {
        withPythonEnv('python') {
          echo 'Deploying latest application tags to Pre-Production...'
          // Deploy the application list, without tests, to the Pre-Production environment
          lock('deployment-plan-PRE') {
            powershell "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.AcceptanceEnvironment}\" --destination_env \"${env.PreProductionEnvironment}\" --app_list \"${params.ApplicationScope}\" --manifest \"${env.ArtifactsFolder}\\deployment_data\\deployment_manifest.cache\""
          }
        }
      }
      post {
        always {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
        failure {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "DeploymentConflicts"
          }
        }
      }
    }
    stage('Deploy Production') {
      steps {
        withPythonEnv('python') {
          echo 'Deploying latest application tags to Production...'
          // Deploy the application list, without tests, to the Production environment
          lock('deployment-plan-PRD') {
            powershell "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.PreProductionEnvironment}\" --destination_env \"${env.ProductionEnvironment}\" --app_list \"${params.ApplicationScope}\" --manifest \"${env.ArtifactsFolder}\\deployment_data\\deployment_manifest.cache\""
          }
        }
      }
      post {
        always {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
          }
        }
        failure {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "DeploymentConflicts"
          }
        }
      }
    }
  }
}