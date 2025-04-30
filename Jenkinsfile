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
        }
        sh "make list-of-images"
      }
     }
    }

  post {
    success {
      finalizeBuild(sh(script: "$DIRECTORY/make list-of-images", returnStdout: true))
    }
    cleanup {
      sh "cd $DIRECTORY && make clean GOOS='linux' GOARCH='amd64'"
    }
  }
}
