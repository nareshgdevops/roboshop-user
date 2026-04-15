pipeline {
    agent any

    stages {
        stage('NodeJS Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'nodejs' }
            }
            steps {
                sh 'npm install'
            }
        }
        stage('Maven Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'maven' }
            }
            steps {
                sh 'mvn package'
            }
        }
        stage('Python Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'python' }
            }
            steps {
                sh 'echo python package'
            }
        }
        stage('Go lang Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'golang' }
            }
            steps {
                sh 'go build'
            }
        }
        stage('Angular Build app dependencies') {
            when {
                expression { env.APP_TYPE == 'angular' }
            }
            steps {
                sh 'echo angular build'
            }
        }
        stage('Sonar Qube Scan') {
            when {
                expression { env.APP_TYPE != 'python' }
            }
            steps {
                sh 'echo /home/github/sonar-scanner-7.3.0.5189-linux-x64/bin/sonar-scanner -Dsonar.host.url=http://sonarqube.nareshdevops1218.online:9000 -Dsonar.token=$SONAR_TOKEN -Dsonar.projectKey=roboshop-$APP_NAME'
            }
        }

        stage('CheckMarx SAST Scan') {
            steps {
                sh 'echo cx scan create -s . --project-name roboshop-$APP_NAME --scan-types sast'
            }
        }

        stage('Fetch Secrets') {
            steps {
                script {
                    def secrets = [
                        [
                            path: 'roboshop-infra/azure-service-priniciple',
                            engineVersion: 2,
                            secretValues: [
                                [envVar: 'AZURE_SUBSCRIPTION_ID', vaultKey: 'AZURE_SUBSCRIPTION_ID'],
                                [envVar: 'AZURE_CLIENT_ID', vaultKey: 'AZURE_CLIENT_ID'],
                                [envVar: 'AZURE_SECRET',    vaultKey: 'AZURE_SECRET'],
                                [envVar: 'AZURE_TENANT',    vaultKey: 'AZURE_TENANT']
                            ]
                        ],
                        [
                            path: 'roboshop-infra/sonarqube',
                            engineVersion: 2,
                            secretValues: [
                                [envVar: 'SONAR_TOKEN', vaultKey: 'sonar_token']
                            ]
                        ]
                    ]

                    withVault([vaultSecrets: secrets]) {
                        env.AZURE_SUBSCRIPTION_ID = AZURE_SUBSCRIPTION_ID
                        env.AZURE_CLIENT_ID = AZURE_CLIENT_ID
                        env.AZURE_SECRET    = AZURE_SECRET
                        env.AZURE_TENANT    = AZURE_TENANT
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                   sh '''
                        az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_SECRET --tenant $AZURE_TENANT
                        az acr login --name nareshgdevops'
                        docker build -t nareshgdevops.azurecr.io/roboshop-$APP_NAME:$GIT_COMMIT .
                        echo trivy image nareshgdevops.azurecr.io/roboshop-$APP_NAME:$GIT_COMMIT --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1
                    '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    az login --service-principal --username $AZURE_CLIENT_ID --password $AZURE_SECRET --tenant $AZURE_TENANT
                    az acr login --name nareshgdevops
                    docker push nareshgdevops.azurecr.io/roboshop-$APP_NAME:$GIT_COMMIT
                '''
            }
        }
    }
}