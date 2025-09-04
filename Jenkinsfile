pipeline {
    agent any  // Runs on any available agent (your Windows machine)

    environment {
        // Define versions and Orchestrator details
        MAJOR = '1'
        MINOR = '0'
        UIPATH_ORCH_URL = 'https://cloud.uipath.com/'  // Orchestrator base URL
        UIPATH_ORCH_TENANT_NAME = 'DefaultTenant'  // Your tenant name
        UIPATH_ORCH_FOLDER_NAME = 'Shared'  // Your folder name
    }

    stages {
        stage('Prepare') {
            steps {
                echo "Starting build ${env.BUILD_NUMBER}"
                checkout scm  // Pulls code from Git
            }
        }

        stage('Build Package') {
            steps {
                echo 'Packing UiPath project...'
                UiPathPack(
                    outputPath: "Output\\${env.BUILD_NUMBER}",  // Where to save .nupkg
                    projectJsonPath: 'project.json',  // Main project file
                    version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],  // Auto-version
                    useOrchestrator: false,  // Local pack
                    traceLevel: 'Information'  // For debugging
                )
            }
        }

        stage('Deploy to Orchestrator') {
            steps {
                echo 'Deploying to UAT environment...'
                withCredentials([
                    string(credentialsId: 'uipath-app-id', variable: 'APP_ID'),
                    string(credentialsId: 'uipath-app-secret', variable: 'APP_SECRET')
                ]) {
                    UiPathDeploy(
                        packagePath: "Output\\${env.BUILD_NUMBER}\\*.nupkg",  // Path to packed file
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                        folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                        environments: 'UAT',  // Environment name
                        credentials: ExternalApp(
                            applicationId: "${APP_ID}",
                            applicationSecret: "${APP_SECRET}",
                            applicationScope: 'OR.Execution OR.Folders OR.Jobs OR.Settings OR.Settings.Read OR.Settings.Write'
                        ),
                        entryPointPaths: 'Main.xaml',  // Main workflow file
                        createProcess: true,  // Auto-create process in Orchestrator
                        traceLevel: 'Verbose'  // Set to Verbose for debugging
                    )
                }
            }
        }

        stage('Run Test Job') {
            steps {
                echo 'Running the deployed process...'
                withCredentials([
                    string(credentialsId: 'uipath-app-id', variable: 'APP_ID'),
                    string(credentialsId: 'uipath-app-secret', variable: 'APP_SECRET')
                ]) {
                    UiPathRunJob(
                        processName: 'CICD_UsingJenkins',  // Your process name
                        parametersFilePath: '',  // No input parameters file
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                        folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                        credentials: ExternalApp(
                            applicationId: "${APP_ID}",
                            applicationSecret: "${APP_SECRET}",
                            applicationScope: 'OR.Execution OR.Folders OR.Jobs OR.Settings OR.Settings.Read OR.Settings.Write'
                        ),
                        jobType: Unattended(),  // Required for unattended job execution
                        priority: 'Low',
                        strategy: Dynamically(jobsCount: 1),  // Run one job
                        resultFilePath: '',  // No output result file
                        failWhenJobFails: true,  // Fail pipeline if job fails
                        waitForJobCompletion: true,
                        timeout: 300,  // 5 minutes
                        traceLevel: 'Verbose'  // Set to Verbose for debugging
                    )
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}