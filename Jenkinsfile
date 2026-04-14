pipeline {
    agent any
    tools{
        maven 'maven-3.9'
    }
    stages {
        stage('test') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceFirst('^origin/', '') ?: 'main'
                    echo 'Testing the application...'
                    echo "executing the pipeline for branch ${branchName}"
                }
            }
        }
        stage('build jar') {
            when{
                expression{
                    (env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceFirst('^origin/', '') ?: 'main') == "main"
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
                    (env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceFirst('^origin/', '') ?: 'main') == "main"
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
                    (env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceFirst('^origin/', '') ?: 'main') == "main"
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
