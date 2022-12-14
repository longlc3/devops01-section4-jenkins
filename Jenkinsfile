pipeline {
    agent any
    environment{
        WORKPLACE_DIR="/var/lib/jenkins/workspace/nginx-app"
        TEST_DIR="/var/lib/jenkins/workspace/nginx-app/unittest"
        DOCKER_IMAGE="longlc3/devops01-nginx"
        ANSIBLE_PATH="/var/lib/jenkins/workspace/nginx-app/ansible"
    }
    stages {
        stage("Build Stage"){
            steps{
                
                sh 'docker-compose build'
                withCredentials([usernamePassword(credentialsId: 'docker-credential', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    // sh "docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:$BUILD_NUMBER"
                    sh "docker push $DOCKER_IMAGE"
                }

            }
        }
        stage("Test Stage"){
            tools {
                allure 'allure2.2'
            }
            steps{
                dir("$TEST_DIR"){
                    sh 'dotnet test -o target'
                    // allure results: [[path: 'target/allure-results']]
                    script{
                        allure ([
                            includeProperties: false,
                            jdk:'',
                            properties: [],
                            reportBuildPolicy: 'ALWAYS',
                            results: [[path: 'target/allure-results']]
                        ])
                    }
                }
            }
        }

        stage("Deploy Stage"){
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            steps {
                dir("$ANSIBLE_PATH"){
                    withCredentials([usernamePassword(credentialsId: 'docker-credential', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        ansiblePlaybook(
                            credentialsId: 'ssh-key',
                            playbook: 'playbook.yml',
                            inventory: 'inventory.ini',
                            become: 'yes',
                            extraVars: [
                                DOCKER_USERNAME: "$DOCKER_USERNAME",  
                                DOCKER_PASSWORD: "$DOCKER_PASSWORD",
                                WORKPLACE_DIR: "$WORKPLACE_DIR"
                            ]
                        )
                    }
                    
                }
                
            }
        }

    }
}