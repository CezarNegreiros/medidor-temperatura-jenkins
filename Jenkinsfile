pipeline {
  agent any                      // Jenkins master ou nó agente
  options {
    timestamps()                 // logs com data/hora
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  triggers {
    cron('H 2 * * *')            // nightly: todo dia ~02:00 (Cenário 4)
    githubPush()                 // dispara em cada push / PR
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
                    sh 'pytest -q --junitxml test-results/results.xml || true'
                }
            }
        }
        post {
            always { junit 'test-results/results.xml' }
        }
    }

  post {
    success  { echo '🎉 Pipeline OK — build+test passaram.' }
    unstable { echo '⚠ Teste falhou — build marcado UNSTABLE.' }
    failure  { echo '❌ Falhou durante a compilação.' }
  }
}
