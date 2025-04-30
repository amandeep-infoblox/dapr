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
        }
      }
    }
    stage("List-Built-Images") {
      steps {
        dir ("$DIRECTORY") {
          sh "make list-of-images > image_list.txt"
          stash includes: 'image_list.txt', name: 'image-list'
        }
      }
    }
  }

  post {
    success {
      dir("${WORKSPACE}/${DIRECTORY}"){
        script {
          unstash 'image-list'
          def images = readFile('image_list.txt').trim()
          echo "Docker images built: ${images}"
          finalizeBuild(images)
        }
      }
    }
    cleanup {
      sh "cd $DIRECTORY && make clean GOOS='linux' GOARCH='amd64'"
    }
  }
}
