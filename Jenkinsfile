def commit_id
pipeline {
    agent any
    stages {
        stage('preparation') {
            steps {
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage ('build') {
            steps {
                echo 'building maven workload'
                sh "mvn clean install"
                echo 'build complete'
            }
        }
        stage ('Code Quality') {
            steps {
                echo 'testing code quality'
               sh "mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=DevOpsProject \
                        -Dsonar.host.url=http://192.168.254.136:9000 \
                        -Dsonar.login=52dfa1c9b53371b2e2002809878f6be36bc1238c"

                echo 'Quality Test Complete'
            }        
        } 
        stage ("image build") {
            steps {
                echo 'building docker image'
                sh "docker build -t 192.168.254.136:8082/position-simulator:${commit_id} ."
                echo 'docker image built'
            }
        }
        stage ('Image Push') {
            steps {
                sh "docker push 192.168.254.136:8082/position-simulator:${commit_id}"
            }
        }
        stage('deploy') {
            steps {
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|192.168.254.136:8082/position-simulator:${commit_id}|' workloads.yaml"
                sh 'kubectl apply -f workloads.yaml'
            }
        }
    }
}
