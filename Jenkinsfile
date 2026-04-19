pipeline {
    agent any
    tools{
        maven 'maven-3.9'
    }
    stages {
        stage('prepare image tag') {
            steps {
                script {
                    echo 'preparing docker image tag'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
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
                    sh 'mvn clean package'
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
                        sh "docker build -t saeedha/java-maven-app:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push saeedha/java-maven-app:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploying docker image to EC2 server') {
            when{
                expression{
                    (env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceFirst('^origin/', '') ?: 'main') == "main"
                }
            }
            steps {
                script {
                    def shellCmd ="bash ./server-commands.sh saeedha/java-maven-app:${IMAGE_NAME}"
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                        sh "scp -i \$SSH_KEY -o StrictHostKeyChecking=no server-commands.sh \$SSH_USER@54.157.58.253:/home/ec2-user"
                        sh "scp -i \$SSH_KEY -o StrictHostKeyChecking=no docker-compose.yaml \$SSH_USER@54.157.58.253:/home/ec2-user"
                        sh "ssh -i \$SSH_KEY -o StrictHostKeyChecking=no \$SSH_USER@54.157.58.253 '${shellCmd}'"
                    }
                }
            }
        }
    }
}
