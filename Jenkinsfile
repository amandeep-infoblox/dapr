@Library('jenkins.shared.library') _

pipeline {
  agent {
    label 'ubuntu_docker_label'
  }
  tools {
    go "Go 1.16"
  }
  options {
    checkoutToSubdirectory('src/github.com/infobloxopen/dapr')
  }
  environment {
    GOPATH = "$WORKSPACE"
    DIRECTORY = "src/github.com/infobloxopen/dapr"
    DOCKER_IMAGE = "infoblox/dapr"
    
  }
  stages {
    stage("Setup") {
      steps {
        prepareBuild()
      }
    }
    stage("Test") {
      steps {
        sh "cd $DIRECTORY && make test"
      }
    }
    stage("build-and-archive-binaries-linux-amd64"){
      steps {
        sh "cd $DIRECTORY && make tidy && make release GOOS='linux' GOARCH='amd64' "
      }
    }
    stage("Build-And-Push-Docker") {
      steps {
        dir ("$DIRECTORY") {
          withDockerRegistry([credentialsId: "dockerhub-bloxcicd", url: ""]) {
            sh "make docker-push GOOS='linux' GOARCH='amd64' "
            // Explicitly run list-of-images in the stage and output to console for debugging
            sh '''
              echo "Executing make list-of-images..."
              pwd
              make list-of-images
              echo "Completed make list-of-images"
            '''
          }
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline succeeded, executing finalizeBuild"
      dir("${WORKSPACE}/${DIRECTORY}"){
        script {
          echo "Running in directory: ${pwd()}"
          echo "Executing make list-of-images for finalizeBuild..."
          def images = sh(script: "make list-of-images", returnStdout: true).trim()
          echo "Docker images to finalize: ${images}"
          finalizeBuild(images)
          echo "finalizeBuild completed"
        }
      }
    }
    cleanup {
      sh "cd $DIRECTORY && make clean GOOS='linux' GOARCH='amd64'"
    }
  }
}
