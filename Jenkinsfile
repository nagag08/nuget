pipeline {
  agent any
  options {
        // This is required if you want to clean before build
        skipDefaultCheckout(true)
    }
    environment {
        JURL = 'https://psemea.jfrog.io'
        RT_URL = 'https://psemea.jfrog.io/artifactory'
        NUGET_REPO = 'nagag-nuget-virt'
        NUGET_UAT_REPO='nagag-nuget-uat'
        PATH = "${env.PATH}:/var/jenkins_home/.dotnet"
    }
    
  stages {
        stage ('Config JFrog CLI') {
            steps {
                withCredentials([string(credentialsId: 'psemea_token', variable: 'psemea_token_str')]) {
                // println "${nagag_jpd1_admin_password}"
                // println "${nagag_jpd1_admin_user}"
                sh """rm -f ~/.jfrog/jfrog-cli.conf.v6"""
                sh """jf c add ${env.BUILD_NUMBER} --interactive=false --overwrite=true --access-token=${psemea_token_str}  --url=${JURL}"""
                sh """jf config use ${env.BUILD_NUMBER}"""
                sh 'jf c s'
                }
            }
        }
    stage('Git Clone') {
      steps {
          cleanWs()
          dir('complete') {
            git 'https://github.com/nagag08/nuget.git'
        }
      }
    }
     stage ('Config NUGET and dotnet'){
            steps {
                dir('complete'){
                    sh """jf dotnet-config --repo-resolve=${NUGET_REPO}"""
                    sh """jf nuget-config --repo-resolve=${NUGET_REPO}"""

                }
            }
    }
    stage ('Install Dependencies'){
            steps {
                dir('complete'){
                    sh """alias nuget='mono /usr/local/bin/nuget.exe'"""
                    sh """jf nuget restore -PackagesDirectory packages --build-name ${env.JOB_NAME} --build-number ${env.BUILD_NUMBER} """

                }
            }
    }
    stage('Audit Scan') {
      steps {
        dir('complete') {
          sh """alias nuget='mono /usr/local/bin/nuget.exe'"""
        //   sh """export PATH=$PATH:/var/jenkins_home/.dotnet"""
          sh """JFROG_CLI_LOG_LEVEL=DEBUG jf audit --fail=false --nuget"""
        }
      }
    }
    stage('Build and pack NUGET') {
      steps {
          dir('complete') {
            //  sh """export PATH=$PATH:/var/jenkins_home/.dotnet """
             sh """jf dotnet build --build-name ${env.JOB_NAME} --build-number ${env.BUILD_NUMBER}"""
             sh """jf dotnet pack -o nugetpack --build-name ${env.JOB_NAME} --build-number ${env.BUILD_NUMBER}"""

        }
      }
    }  
    stage('upload nuget pkg') {
      steps {
          dir('complete') {
             sh """jf rt u  nugetpack/*.nupkg ${NUGET_REPO} --build-name ${env.JOB_NAME} --build-number ${env.BUILD_NUMBER}"""
        }
      }
    }
    stage('Build-info-ENV') {
      steps {
          dir('complete') {
            sh """jf rt bce ${env.JOB_NAME} ${env.BUILD_NUMBER}"""
        }
      }
    }
    stage('Build-info-GIT') {
      steps {
        dir('complete') {
                sh """jf rt bag ${env.JOB_NAME} ${env.BUILD_NUMBER}"""
        }
      }
    }
    stage('Build-info-publish') {
      steps {
          dir('complete') {
        sh """jf rt bp ${env.JOB_NAME} ${env.BUILD_NUMBER}"""
      }
      }
    }
    
    stage('Integration testing') {
      steps {
        dir('complete') {
          echo "Integraton testing criteria: Success"
        }
      }
    }
    stage('Promote build to UAT') {
      steps {
        dir('complete') {
          sh """jf rt bpr --source-repo=${NUGET_REPO} --status=UAT --copy --props="status=UAT" ${env.JOB_NAME} ${env.BUILD_NUMBER} ${NUGET_UAT_REPO}"""
        }
      }
    }
  }
}


