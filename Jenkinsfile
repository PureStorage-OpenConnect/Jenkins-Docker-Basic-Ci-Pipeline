def StartContainer() {
    bat "docker run -e \"ACCEPT_EULA=Y\" -e \"SA_PASSWORD=P@ssword1\" --name ${CONTAINER_NAME} -d -i -p ${PORT_NUMBER}:1433 microsoft/mssql-server-linux:2017-GA"    
    powershell "While (\$((docker logs ${CONTAINER_NAME} | select-string ready | select-string client).Length) -eq 0) { Start-Sleep -s 1 }"    
}

def RemoveContainer() {
    powershell "If (\$((docker ps -a --filter \"name=${CONTAINER_NAME}\").Length) -eq 2) { docker rm -f ${CONTAINER_NAME} }"
}

def DeployDacpac() {
    def SqlPackage = "C:\\Program Files\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "${SCM_PROJECT}\\bin\\Release\\${SCM_PROJECT}.dacpac"
    def ConnString = "server=localhost,${PORT_NUMBER};database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}

pipeline {
    agent any
    
    environment {
        CONTAINER_NAME = "SqlLinux"
        PORT_NUMBER    = 15556
        SCM_PROJECT    = "Jenkins-Docker-Basic-Ci-Pipeline"
    }
    
    parameters {
        booleanParam(defaultValue: true, description: '', name: 'HAPPY_PATH')
    }
    
    stages {
        stage('git checkout') {     
            steps {
                timeout(time: 5, unit: 'SECONDS') {
                    checkout "https://github.com/PureStorage-OpenConnect/Jenkins-Docker-Basic-Ci-Pipeline"
                }
            }
        }
        stage('build dacpac') {
            steps {
                bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
                stash includes: "${SCM_PROJECT}\\bin\\Release\\${SCM_PROJECT}.dacpac", name: 'theDacpac'
            }
        }
    
        stage('start container') {
            steps {
                RemoveContainer()
                timeout(time: 20, unit: 'SECONDS') {
                    StartContainer()
                }
            }
        }
    
        stage('deploy dacpac') {
            steps {
                timeout(time: 60, unit: 'SECONDS') {
                   DeployDacpac()
                }
            }
        }
    }
    post {
        always {
            RemoveContainer()
        }
        success {
            print 'post: Success'
        }
        unstable {
            print 'post: Unstable'
        }
        failure {
            print 'post: Failure'
        }
    }
}