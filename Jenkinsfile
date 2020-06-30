pipeline {
  agent {
    label 'ubuntu_docker_label'
  }
  tools {
    go "Go 1.13"
  }
  options {
    checkoutToSubdirectory('src/github.com/gravitational/teleport')
  }
  environment {
    GOPATH = "$WORKSPACE"
    DIRECTORY = "src/github.com/gravitational/teleport"
  }
  stages {
//    stage("Test") {
//      steps {
//        sh "cd $DIRECTORY && make test"
//      }
//    }
    stage("Install zip") {
      steps {
        sh '''
          while pgrep apt > /dev/null 2>&1
          do
            echo "Waiting for other software managers (APT) to finish..."
            sleep 1
          done
          sudo apt-get install -y zip
        '''
      }
    }
    stage("Build") {
      when {
        changeRequest()
      }
      steps {
        sh '''
          cd $DIRECTORY && make image
        '''
      }
    }
    stage("Push") {
      when {
        anyOf {
          branch "develop"
          branch "release/*"
          buildingTag()
        }
      }
      steps {
        withDockerRegistry([credentialsId: "dockerhub-bloxcicd", url: ""]) {
          sh '''
            cd $DIRECTORY && make publish
          '''
        }
      }
    }
    stage("Push Helm Chart") {
      when {
        anyOf {
          branch "develop"
          branch "release/*"
          buildingTag()
        }
      }
      steps {
        // AWS_IAM_CI_CD_INFRA
        withAWS(credentials: "CICD_HELM", region: "us-east-1") {
          sh "cd $DIRECTORY && make helm-push"
        }
          dir("${WORKSPACE}/${DIRECTORY}") {
          archiveArtifacts artifacts: 'helm/*.tgz'
          archiveArtifacts artifacts: 'helm/build.properties'
        }
      }
    }
  }
  post {
    cleanup {
      sh "cd $DIRECTORY && make clean"
    }
  }
}
