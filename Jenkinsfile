@Library("global-shared-lib") _

pipeline {
    agent any

    environment {
        netcoreVersion = 'netcoreapp3.1'
      	APP_DB = credentials('xmera-db')
      	PROJECT = 'PROJECT'
    }
    
    stages {

        stage("Unit Testing & Code Analysis Parallel") {
            parallel {
                stage("Unit Testing") {
                    steps {
                        powershell """
                            echo "Running Unit Tests"
                            """
                    }
                }

                stage("Code Analysis") {
                    steps {
                        powershell """
                            echo "Running Code Analysis"
                            """
                    }
                }
            }
        }

        stage("Build") {
            steps {
                script {
                    // Build with Release on Demo and Prod, everthing else in Debug
                    def configuration = 'Debug'
                    if(env.BRANCH_NAME == 'demo' || env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'qa') {
                        configuration = 'Release'
                    }
                  
                  	powershell script:"""
                    	# Init secrets
                        dotnet user-secrets init --project $PROJECT
                        
                        # Set secrets
                        dotnet user-secrets set "ConnectionStrings:DB" "$APP_DB" --project $PROJECT
                    	""", label: "Setting credentials"

                    powershell script: """
                        dotnet publish -c ${configuration} --output build/ --framework ${netcoreVersion}
                        """, label: "Building Artifacts"
                }
            }
        }

        stage("Deploy") {
            steps {
                script {
                    def GIT_REPO_NAME = getGitRepoName(env.GIT_URL)

                    switch (env.BRANCH_NAME) {
                      case "dev":
                        deployIISServerApp([destination: "\\\\\Server-Desarrol\\Dev_Server\\" + GIT_REPO_NAME, server: env.BRANCH_NAME, local: false])
                        break
                      case "qa":
                        deployIISServerApp([destination: "C:\\inetpub\\wwwroot\\" + GIT_REPO_NAME, server: env.BRANCH_NAME, local: true])
                        break
                      case "demo":
                        deployIISServerStatic([destination: "\\\\185.17.18.26\\demos_server\\" + GIT_REPO_NAME, server: env.BRANCH_NAME, local: true])
                        break
                      case "master":
                        echo 'Nothing yet!'
                        //deployIISServerApp([destination: "C:\\inetpub\\wwwroot\\" + GIT_REPO_NAME, server: 'PROD', local: true])
                        break
                    }
                }
            }
        }
    }
}  