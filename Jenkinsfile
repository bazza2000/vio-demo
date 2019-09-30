pipeline {
  agent any
  options {
    timeout(time: 1, unit: 'HOURS')
    disableConcurrentBuilds()
    buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '10', artifactDaysToKeepStr: '30', artifactNumToKeepStr: '10'))
    timestamps()
  }
  triggers {
    GenericTrigger(
      genericVariables: [
        [key: 'ref', value: '$.ref']
      ],

      causeString: 'Triggered on $ref',

      token: 'vio-demo',

      printContributedVariables: true,

      printPostContent: true,

      silentResponse: false,
    )
  }
  stages {
    stage('React Test') {
      agent {
        docker {
          image 'node:12.8.0'
          args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
      }
      steps {
        sh 'npm set strict-ssl false'
        //sh 'npm install'
      }
    }
    stage('Cypress Test') {
      agent {
        docker {
          image 'cypress/browsers:chrome69'
          args '-v /var/run/docker.sock:/var/run/docker.sock --env PERCY_TOKEN=0d2bcb1a8cb81aac8dd9392aa42f37a1f5668efa89c2aa738849dca4a2c277ac --add-host=demo.viosystems.com:54.171.82.255'
        }
      }
      steps {
        sh 'npm set strict-ssl false'
        sh 'npm install'
        sh '$(npm bin)/cypress run --browser chrome'
      }
    }
    stage('React Build') {
      parallel {
        stage('React Build/Push PIBS') {
          steps {
            sh 'docker build -t docker.viosystems.com:8443/vio-demo:${BUILD_NUMBER} .'
            sh 'docker login  docker.viosystems.com:8443 -u ${GITHUB_ASH_CREDS_USR} -p ${GITHUB_ASH_CREDS_PSW}'
            sh 'docker push   docker.viosystems.com:8443/vio-demo:${BUILD_NUMBER}'
            sh 'docker tag   docker.viosystems.com:8443/vio-demo:${BUILD_NUMBER} docker.viosystems.com:8443/vio-demo:latest'
            sh 'docker push   docker.viosystems.com:8443/vio-demo:latest'
          }
        }
      }
    }
  }
  environment {
    GITHUB_ASH_CREDS  = credentials('jenkins-user-for-nexus-repository')
  }
}
