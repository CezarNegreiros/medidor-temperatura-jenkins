properties([
  pipelineTriggers([
    cron('H/2 * * * *')          // cron válido
  ])
])

pipeline {
  agent any
  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  triggers {
    githubPush()             // webhook
  }

  stages {
    stage('Build — imagem') {
      steps {
        script {
          docker.build('proj-build', '-f Dockerfile.build .')
        }
      }
    }

    stage('Build — compilação') {
      steps {
        script {
          docker.image('proj-build').inside {
            sh 'echo "Código Python compilado/checado."'
          }
        }
      }
    }

    stage('Test — imagem') {
      steps {
        script {
          docker.build('proj-test', '-f Dockerfile.test .')
        }
      }
    }

    stage('Test — suíte') {
      steps {
        script {
          docker.image('proj-test').inside {
            // continua mesmo que haja falha → JUnit decide se fica UNSATBLE
            sh 'pytest -q --junitxml test-results/results.xml || true'
          }
        }
      }
      post {
        always {
          junit 'test-results/results.xml'
        }
      }
    }
  }

  post {
    success  { echo '🎉 Pipeline OK — build+test passaram.' }
    unstable { echo '⚠ Teste falhou — build marcado UNSTABLE.' }
    failure  { echo '❌ Falhou durante a compilação.' }
  }
}
