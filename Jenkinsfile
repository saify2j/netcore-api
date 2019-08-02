def SERVICE_NAME="netcoreapi"
def SUBDOMAIN="sub1"
def PACKAGE_NAME="${env.BRANCH_NAME}-netcore-api.${SUBDOMAIN}.1.0.${env.BUILD_NUMBER}"

node {
    def remote = [:]
    def app


    if (env.BRANCH_NAME == "dev") {
      stage('Cleaning ENV') {
          bat "IF EXIST Publish RMDIR /S /Q Publish"
          bat "IF EXIST obj RMDIR /S /Q obj"
          bat "IF EXIST bin RMDIR /S /Q bin"
          // deleteDir()
          dir("${workspace}@tmp") {
                deleteDir()
            }
      }
      stage('Clone repository') {
          /* repository cloned to our workspace */
          checkout scm
      }
      stage('DEV: Restore Packages') {
          /* This restoring of the packages of the application. */
          bat "dotnet restore netcore-api.sln"
      }
      stage('DEV: Clean') {
          /* This clean the solution. */
          bat "dotnet clean"

      }
      stage('DEV: Build') {
          /* This builds the solution */
          // bat "dotnet publish netcore-api.sln -o Publish -c Release -r win10-x86"
          bat "dotnet build --configuration Release -o Publish"

      }
      stage('DEV: Pack') {
          /* This will create zip */
          zip zipFile: "${PACKAGE_NAME}.zip", archive: false, dir: 'Publish'
          bat "dir"

      }
    }



    // stage('Test') {
    //
    //      bat "echo 'test passed'"
    // }



    /* This builds the solution **\\nupkgs\\*.nupkg */
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'AppServer',
    usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
    {
      remote.name = 'appserver'
      remote.host = "${env.ServerIP}"
      remote.user = "${USERNAME}"
      remote.password = "${PASSWORD}"
      remote.allowAnyHosts = true
      if (env.BRANCH_NAME == "dev") {
        stage('DEV: Deploy Artifact') {
          sshPut remote: remote, from: "${PACKAGE_NAME}.zip", into: "deployments/packages/"
          sshCommand remote: remote, command: "ls -al deployments/${env.BRANCH_NAME}/"
        }
        stage('DEV: Run Application') {
          sshCommand remote: remote, command: "rm -rf deployments/${env.BRANCH_NAME}/*"
          sshCommand remote: remote, command: "unzip deployments/packages/${PACKAGE_NAME}.zip -d deployments/${env.BRANCH_NAME}/${SUBDOMAIN}/"
          sshScript remote: remote, script: "deployments/systemd/deploy.sh ${SUBDOMAIN} ${env.BRANCH_NAME}"
          // sshCommand remote: remote, command: "systemctl restart netcore-api.service"
          // for extracting into multiple directory: ${PACKAGE_NAME}
          //     sshGet remote: remote, from: 'abc.sh', into: 'bac.sh', override: true
          //     sshRemove remote: remote, path: "deployments/${env.BRANCH_NAME}", failOnError: false
        }
      }

    }

    // if (env.BRANCH_NAME == "dev") {
    //   def Powershell_V="%SystemRoot%\\SysWOW64\\WindowsPowerShell\\v1.0\\powershell.exe"
    //     stage('DEV: Deploy') {
    //       bat "${Powershell_V} Import-Module Webadministration"
    //       bat "${Powershell_V} GET-IISSite"
    //       bat "${Powershell_V} -ExecutionPolicy RemoteSigned -File scripts/deploy.ps1 ${PACKAGE_NAME}.zip"
    //     }
    // }

}
