pipeline {
  agent {
    label 'mikero'
  }

  environment {
    PYTHONUNBUFFERED = '1'
  }

  stages {
    stage('Checkout') {
      steps {
        dir('x/cba') {
          git url: 'https://github.com/CBATeam/CBA_A3.git', branch: 'master', changelog: false, poll: false
        }

        dir('z/ace') {
          script {
            def aceGit = git url: 'https://github.com/acemod/ACE3.git', branch: 'master', changelog: true, poll: true
            env.ACE_COMMIT = aceGit.GIT_COMMIT
          }

           // Fix legacy registry settings for exclusion
          powershell '((Get-Content -path tools/make.py -Raw) -replace "m_exclude_compression", "wildcard_exclude_from_compression") | Set-Content -Path tools/make.py'
          powershell '((Get-Content -path tools/make.py -Raw) -replace "m_exclude2", "wildcard_exclude_from_pbo_unbinarised_missions") | Set-Content -Path tools/make.py'
          powershell '((Get-Content -path tools/make.py -Raw) -replace "m_exclude", "wildcard_exclude_from_pbo_normal") | Set-Content -Path tools/make.py'
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

        // Link dummy CDLC data
        bat 'mklink /j gm z\\ace\\tools\\pDummies\\gm'
        bat 'mklink /j vn z\\ace\\tools\\pDummies\\vn'

        // Link RHS Data
        bat 'mklink /j rhsafrf %RHS_DATA%\\rhsafrf'
        bat 'mklink /j rhsgref %RHS_DATA%\\rhsgref'
        bat 'mklink /j rhssaf %RHS_DATA%\\rhssaf'
        bat 'mklink /j rhsusf %RHS_DATA%\\rhsusf'
      }
    }

    stage('Build') {
      steps {
        // Mount P: drive
        bat 'subst P: .'

        // Build ACE with CI exit status and external files check enabled
        bat 'python3\\python.exe z/ace/tools/make.py ci'

        // Move built mod to root of workspace
        bat 'move z/ace/release/@ace @ace'
      }
    }

    stage('Steam Workshop') {
      steps {
        publishSteamWorkshop '1882627645', '@ace', "https://github.com/acemod/ACE3/commit/${env.ACE_COMMIT}"
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

void publishSteamWorkshop(String id, String mod, String changeNote) {
  bat "\"C:\\Program Files (x86)\\Steam\\SteamApps\\common\\Arma 3 Tools\\Publisher\\PublisherCmd.exe\" update /changeNote:$changeNote /id:$id /path:$mod"
}
