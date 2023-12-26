pipeline {
  agent none

  stages {
      stage('worker-build') {
        when {
            changeset '**/worker/**'
        }

        agent {
          docker {
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }

        steps {
          echo 'Compiling worker app..'
          dir('worker') {
            sh 'mvn compile'
          }
        }
      }
      stage('worker-test') {
        when {
          changeset '**/worker/**'
        }
        agent {
          docker {
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }
        steps {
          echo 'Running Unit Tets on worker app..'
          dir('worker') {
            sh 'mvn clean test'
          }
        }
      }
      stage('worker-package') {
        when {
          branch 'master'
          changeset '**/worker/**'
        }
        agent {
          docker {
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }
        steps {
          echo 'Packaging worker app'
          dir('worker') {
            sh 'mvn package -DskipTests'
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
          }
        }
      }

      stage('worker-docker-package') {
          agent any
          when {
            changeset '**/worker/**'
            branch 'master'
          }
          steps {
            echo 'Packaging worker app with docker'
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-token') {
                def workerImage = docker.build("ennc0d3/worker:v${env.BUILD_ID}", './worker')
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                workerImage.push('latest')
              }
            }
          }
      }

      // result
      stage('result-build') {
        when {
          changeset '**/result/**'
        }
        agent {
          docker {
            image 'nodejs:latest'
          }
        }
        steps {
          echo 'TODO:Starting to Build'
          sh 'npm install'
        }
      }
      stage('vote-test') {
        when {
          changeset '**/result/**'
        }
        agent {
            docker {
              image 'nodejs:latest'
            }
        }
        steps {
          echo 'Starting to test'
          sh 'npm install'
          sh 'npm test'
        }
      }

      // vote pipeline
      stage('vote-build') {
        when {
          changeset '**/vote/**'
        }
        steps {
          echo 'TODO:Starting to Build'
        }
      }
      stage('vote-test') {
          when {
            changeset '**/vote/**'
          }
          steps {
            echo 'Starting to test'
            sh 'vote/integration_test.sh'
          }
      }
  }

  post {
    failure {
      slackSend(channel: '#ci-cd', message: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
    }
    success {
      slackSend(channel: '#ci-cd', message: "Build Success: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
    }
    always {
        echo 'Building multibranch pipeline for worker is completed..'
    }
  }
}
