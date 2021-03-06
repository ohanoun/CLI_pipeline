pipeline {
  agent none

  parameters {
    string(defaultValue: "XJTmuUe9QQURKax0yvn2YOhX", description: 'Neoload Web Token', name: 'token')
    string(defaultValue: "defaultzone", description: 'Zone identifier', name: 'zone_id')
    string(defaultValue: "https://dockerps1.apps.neotys.com:8080", description: 'NeoLoad Web Api Url', name: 'api_url')
  }

  stages {
    stage('Attach Worker') {
      agent {
        docker {
          image 'python:3-alpine'
        }
      }
      environment {
        NEOLOAD="${WORKSPACE}/.local/bin/neoload"
        HOME="${WORKSPACE}"
      }
      stages {
        stage('Get NeoLoad CLI') {
          steps {
              sh "pip3 install -U --pre neoload"
              sh "$NEOLOAD --version"
          }
        }
        stage('Get NeoLoad project'){
            steps {
                sh 'rm -f *.yml; wget https://raw.githubusercontent.com/Neotys-Labs/neoload-cli/78a1366743b041088f357f0f702babfce5e55187/tests/neoload_projects/simpledemo.yml'
            }
        }
        stage('Prepare NeoLoad test') {
          steps {
              sh """$NEOLOAD \
                     login --url ${params.api_url} --ssl-cert False ${params.token} \
                     test-settings --zone ${params.zone_id} --scenario 'simpledemo' createorpatch "My Jenkins Test With CLI" \
                     project --path simpledemo.yml upload
                """
          }
        }
        stage('Run Test') {
          steps {
              sh """$NEOLOAD run \
                  --return-0 \
                  --web-vu 50 \
                 """
          }
        }
        stage('Generate Test Report') {
          steps {
             sh "$NEOLOAD test-results junitsla"
          }
          post {
              always {
                  junit 'junit-sla.xml'
              }
          }
        }
      }
    }
  }
}
