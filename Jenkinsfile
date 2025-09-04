pipeline {
    agent any
    environment {
        MAJOR = '1'
        MINOR = '0'
        UIPATH_ORCH_URL = 'https://cloud.uipath.com/'
        UIPATH_ORCH_ACCOUNT_NAME = 'learnjvubbvf'
        UIPATH_ORCH_TENANT_NAME = 'DefaultTenant'
        UIPATH_ORCH_FOLDER_NAME = 'Shared'
        UIPATH_APP_ID = credentials('uipath-app-id')
        UIPATH_APP_SECRET = credentials('uipath-app-secret')
    }
    stages {
        stage('Prepare') {
            steps {
                echo "Starting build ${env.BUILD_NUMBER}"
                checkout scm
            }
        }
        stage('Build Package') {
            steps {
                echo 'Packing UiPath project...'
                UiPathPack(
                    outputPath: "Output\\${env.BUILD_NUMBER}",
                    projectJsonPath: 'project.json',
                    version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                    useOrchestrator: false,
                    traceLevel: 'Information'
                )
            }
        }
        stage('Deploy to Orchestrator') {
            steps {
                echo 'Deploying to UAT environment...'
                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}\\*.nupkg",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    environments: 'UAT',
                    credentials: ExternalApp(
                        accountName: "${UIPATH_ORCH_ACCOUNT_NAME}",
                        applicationId: "${UIPATH_APP_ID}",
                        applicationSecret: "${UIPATH_APP_SECRET}",
                        applicationScope: 'OR.Execution OR.Folders OR.Jobs OR.Settings OR.Settings.Read OR.Settings.Write'
                    ),
                    entryPointPaths: 'Main.xaml',
                    createProcess: true,
                    traceLevel: 'Information'
                )
            }
        }
        stage('Run Test Job') {
            steps {
                echo 'Running the deployed process...'
                UiPathRunJob(
                    processName: 'CICD_UsingJenkins',
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    credentials: ExternalApp(
                        accountName: "${UIPATH_ORCH_ACCOUNT_NAME}",
                        applicationId: "${UIPATH_APP_ID}",
                        applicationSecret: "${UIPATH_APP_SECRET}",
                        applicationScope: 'OR.Execution OR.Folders OR.Jobs OR.Settings OR.Settings.Read OR.Settings.Write'
                    ),
                    priority: 'Low',
                    strategy: Dynamically(jobsCount: 1),
                    waitForJobCompletion: true,
                    timeout: 300,
                    traceLevel: 'Information'
                )
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}