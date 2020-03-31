pipeline {
  agent {
    label 'hmdaops'
  }

  stages {
    stage('init') {
      steps {
        script {
          library identifier: "hmdaUtils@jenkinsSharedLibraries", retriever: modernSCM (
              [
                  $class: 'GitSCMSource',
                  remote: 'https://github.cfpb.gov/hmda-devops/hmda-devops.git'
              ]
          )

          init.setEnvironment('hmda_help')
        }
      }
    }

    stage('Build And Publish Docker Image') {
      steps {
        script {
          withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub',
            usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'hmda-platform-jenkins-service',
              usernameVariable: 'DTR_USER', passwordVariable: 'DTR_PASSWORD']]) {
              withCredentials([string(credentialsId: 'internal-docker-registry', variable: 'DOCKER_REGISTRY_URL')]){
                dockerBuild.dockerBuild('hmda-help', '')
                security.dockerImageScan('hmda-help', env.DOCKER_TAG)
              }
            }
          }
        }
      }
    }

     stage('Deploy') {
      agent {
        docker {
          image 'hmda/helm'
          reuseNode true
          args '--entrypoint=\'\''
        }
      }
      steps {
        script {
          withCredentials([file(credentialsId: 'hmda-ops-kubeconfig', variable: 'KUBECONFIG')]) {
            if (env.DOCKER_PUSH == 'true') {
              helm.deploy('hmda-help')
            }
          }
        }
      }
    }

  }

}
