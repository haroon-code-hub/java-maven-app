pipeline {
    agent any
    tools{
        maven 'maven-3.9'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version'
                    sh '''
                        mvn build-helper:parse-version versions:set \
                            -DnewVersion=\\${parsedVersion.majorVersion}.\\${parsedVersion.minorVersion}.\\${parsedVersion.nextIncrementalVersion} \
                            versions:commit
                    '''
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

        stage('commit version update') {
            when{
                expression{
                    def branchName = env.BRANCH_NAME ?: env.GIT_BRANCH?.replaceFirst('^origin/', '') ?: 'main'
                    def lastCommit = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
                    branchName == "main" && lastCommit != "ci: version bump"
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId:'git-credentials', passwordVariable:'GIT_PASS', usernameVariable:'GIT_USER')]){
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'

                        sh 'git remote set-url origin https://${GIT_USER}:${GIT_PASS}@github.com/haroon-code-hub/java-maven-app.git'
                        sh 'git checkout -B main'
                        sh 'git add pom.xml'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git pull --rebase origin main'
                        sh 'git push origin main'
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
