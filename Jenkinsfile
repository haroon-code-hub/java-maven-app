pipeline {
    agent any
    tools{
        maven 'maven-3.9'
    }
    stages {
        stage('test') {
            steps {
                script {
                    echo 'Testing the application...'
                    echo "executing the pipeline for branch  $BRANCH_NAME"
                }
            }
        }
        stage('build jar') {
            when{
                expression{
                    BRANCH_NAME == "main"
                }
            }
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn package'
                }
            }
        }
        stage('build image') {
            when{
                expression{
                    BRANCH_NAME == "main"
                }
            }
            steps {
                script {
                    echo 'building the docker image...'
                    withCredentials([usernamePassword(credentialsId:'docker-hub-creds', passwordVariable:'PASS', usernameVariable:'USER')]){
                        sh 'docker build -t saeedha/java-maven-app:jma-2.0 .'
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push saeedha/java-maven-app:jma-2.0"
                    }
                }
            }
        }
        stage('deploy') {
            when{
                expression{
                    BRANCH_NAME == "main"
                }
            }
            steps {
                script {
                    echo 'deploying docker image...'
                }
            }
        }
    }
}

