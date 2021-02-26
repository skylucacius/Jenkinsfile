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
    stages {
        stage('Test') {
            steps {
                /* `make check` returns non-zero on test failures,
                * using `true` to allow the Pipeline to continue nonetheless
                */
                sh 'make check || true' 
                junit '**/target/*.xml' 
            }
        }
    }    
  }
    stages {
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' 
              }
            }
            steps {
                sh 'make publish'
            }
        }
    }
  }