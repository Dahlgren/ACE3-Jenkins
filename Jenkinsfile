pipeline {
  agent {
    label 'mikero'
  }

  stages {
    stage('Checkout') {
      steps {
        dir('x/cba') {
          git url: 'https://github.com/CBATeam/CBA_A3.git', branch: 'master'
        }

        dir('z/ace') {
          git url: 'https://github.com/acemod/ACE3.git', branch: 'master'

          // Fix legacy pboproject parameter 
          powershell '((Get-Content -path tools/make.py -Raw) -replace \'"\\+X"\', \'"+G"\') | Set-Content -Path tools/make.py'
          powershell '((Get-Content -path tools/make.py -Raw) -replace \'"-X"\', \'"-G"\') | Set-Content -Path tools/make.py'

          // Set bad exit code on error
          powershell '((Get-Content -path tools/make.py -Raw) -replace \'sys.exit\\(0\\)\', \'sys.exit(len(failedBuilds))\') | Set-Content -Path tools/make.py'
        }
      }
    }

    stage('Python') {
      steps {
        bat 'curl https://www.python.org/ftp/python/3.7.4/python-3.7.4-embed-win32.zip --output python3.zip'
        powershell 'Expand-Archive -Path python3.zip -DestinationPath python3'
      }
    }

    stage('Arma Data') {
      steps {
        // Link Arma 3 Data
        bat 'mklink /j a3 %A3_DATA%\\a3'

        // Link RHS Data
        bat 'mklink /j rhsafrf %RHS_DATA%\\rhsafrf'
        bat 'mklink /j rhsgref %RHS_DATA%\\rhsgref'
        bat 'mklink /j rhsusf %RHS_DATA%\\rhsusf'
      }
    }

    stage('Build') {
      steps {
        // Mount P: drive
        bat 'subst P: .'

        // Build ACE with CI exit status and external files check enabled
        bat 'python3\\python.exe z/ace/tools/make.py ci checkexternal'

        // Move built mod to root of workspace
        bat 'move z/ace/release/@ace @ace'
      }
    }
  }

  post { 
    always {
      // Archive built mod on success
      archiveArtifacts allowEmptyArchive: true, artifacts: '@ace/**/*'

      // Archive pboproject log files
      archiveArtifacts allowEmptyArchive: true, artifacts: 'temp/*.log'

      // Cleanup workspace to avoid wasting disk space
      deleteDir()

      // Dismount P: drive
      bat 'subst P: /D'
    }
  }
}
