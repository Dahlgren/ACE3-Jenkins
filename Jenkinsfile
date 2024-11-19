pipeline {
  agent {
    label 'hemtt'
  }

  stages {
    stage('Checkout') {
      steps {
        script {
          def aceGit = git url: 'https://github.com/acemod/ACE3.git', branch: 'master', changelog: true, poll: true
          env.ACE_COMMIT = aceGit.GIT_COMMIT
        }
      }
    }

    stage('Download HEMTT') {
      steps {
        powershell 'Invoke-WebRequest -Uri https://github.com/BrettMayson/HEMTT/releases/latest/download/windows-x64.zip -OutFile hemtt.zip'
        powershell 'Expand-Archive -Path hemtt.zip -DestinationPath .'
      }
    }

    stage('Build') {
      steps {
        // Build ACE
        bat 'hemtt.exe build'

        // Move built mod to root of workspace
        bat 'move .hemttout/build @ace'
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

      // Cleanup workspace to avoid wasting disk space
      deleteDir()
    }
  }
}

void publishSteamWorkshop(String id, String mod, String changeNote) {
  bat "\"C:\\Program Files (x86)\\Steam\\SteamApps\\common\\Arma 3 Tools\\Publisher\\PublisherCmd.exe\" update /changeNote:$changeNote /id:$id /path:$mod"
}
