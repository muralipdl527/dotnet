pipeline {
  agent any
  options { timestamps() }

  environment {
    SCANCENTRAL_PATH = "${WORKSPACE}/bin/scancentral"
    JAVA_HOME        = "/usr/lib/jvm/java-17-openjdk-amd64"
    PATH             = "${JAVA_HOME}/bin:${env.PATH}"
  }

  stages {
    stage('Prep ScanCentral CLI') {
      steps {
        sh '''
          if [ ! -f "${SCANCENTRAL_PATH}" ]; then
            echo "ERROR: ${SCANCENTRAL_PATH} not found in repo/bin"
            exit 1
          fi
          chmod +x "${SCANCENTRAL_PATH}"
          "${SCANCENTRAL_PATH}" -version
        '''
      }
    }

    stage('Package for FoD') {
      steps {
        sh '''
          rm -rf fod_package && mkdir -p fod_package/bin

          echo "=== Copying source files ==="
          find . -type f -name "*.cs" | grep -v scancentral | xargs -I {} cp --parents {} fod_package/

          echo "=== Copying prebuilt binaries from repo ==="
          BIN_FILES=$(find . -type f \\( -name "*.dll" -o -name "*.exe" -o -name "*.pdb" \\) | grep -v scancentral || true)
          if [ -n "$BIN_FILES" ]; then
              echo "$BIN_FILES" | xargs -I {} cp --parents {} fod_package/
          else
              echo "No binaries found in repo. Did you commit the bin/ folder?"
              exit 1
          fi

          echo "=== Creating ScanCentral package ==="
          "${SCANCENTRAL_PATH}" package -bt none -o output.zip -bf $(find . -name "*.csproj" | head -n 1)

          echo "=== Package contents ==="
          unzip -l output.zip | grep -E "\\.dll|\\.exe|\\.pdb" || true
        '''
      }
    }

    stage('Upload to FoD') {
      steps {
        script {
          fodStaticAssessment(
            releaseId: '1562867',
            scanCentral: 'none',
            srcLocation: "${WORKSPACE}/output.zip",
            overrideGlobalConfig: false
          )
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'output.zip', fingerprint: true
    }
  }
}
