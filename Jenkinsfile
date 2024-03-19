pipeline {
    agent none

    environment {
        DOCKERHUB_AUTH = credentials('9cefdfca-96f9-4db0-8590-319f70870737')
        ID_DOCKER = "${DOCKERHUB_AUTH_USR}"
        PORT_EXPOSED = "80"
    }

    stages {
        stage ('Build Image') {
            agent any
            steps {
                script {
                    sh 'docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage ('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Clean Environment"
                        docker rm -f ${IMAGE_NAME} || echo "container does not exist"
                        docker run --name ${IMAGE_NAME} -d -p ${PORT_EXPOSED}:5000 -e PORT=5000 ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                    '''
                }
            }
        }

        stage ('Unit Tests') {
            agent any
            steps {
                script {
                    sh 'docker exec -t ${IMAGE_NAME} python tests.py'
                }
            }
        }

        stage ('Fonctional Tests') {
            agent any
            steps {
                script {
                    sh 'curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"'
                }
            }
        }

        stage('Clean Container') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop ${IMAGE_NAME}
                        docker rm ${IMAGE_NAME}
                    '''
                }
            }
        }

        stage ('Release') {
            agent any
            steps {
                script {
                    sh '''
                        echo ${DOCKERHUB_AUTH_PSW} | docker login -u ${DOCKERHUB_AUTH_USR} --password-stdin
                        docker push ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage ('Deploy in staging') {
            agent any
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            environment {
                HOSTNAME_DEPLOY_STAGING = "34.230.0.167"
            }
            steps {
                sshagent(credentials : ['SSH_AUTH_SERVER']) {
                    sh '''
                        [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                        ssh-keyscan -t rsa,dsa ${HOSTNAME_DEPLOY_STAGING} >> ~/.ssh/known_hosts
                        command1="docker login -u ${DOCKERHUB_AUTH_USR} -p ${DOCKERHUB_AUTH_PSW}"
                        command2="docker pull ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
                        command3="docker rm -f ${IMAGE_NAME} || echo 'app does not exist'"
                        command4="docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
                        ssh -t centos@${HOSTNAME_DEPLOY_STAGING} \
                            -o SendEnv=IMAGE_NAME \
                            -o SendEnv=IMAGE_TAG \
                            -o SendEnv=DOCKERHUB_AUTH_USR \
                            -o SendEnv=DOCKERHUB_AUTH_PSW \
                            -C "${command1} && ${command2} && ${command3} && ${command4}"
                    '''
                }
            }
        }
       stage('Test staging') {
           agent any
           steps {
              script {
                    sh 'curl http://34.230.0.167:${PORT_EXPOSED} | grep -q "Hello world!"'
              }
           }
       }
        stage ('Deploy in prod') {
            agent any
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            environment {
                HOSTNAME_DEPLOY_PROD = "3.80.175.207"
            }
            steps {
                sshagent(credentials : ['SSH_AUTH_SERVER']) {
                    sh '''
                        [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                        ssh-keyscan -t rsa,dsa ${HOSTNAME_DEPLOY_PROD} >> ~/.ssh/known_hosts
                        command1="docker login -u ${DOCKERHUB_AUTH_USR} -p ${DOCKERHUB_AUTH_PSW}"
                        command2="docker pull ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
                        command3="docker rm -f ${IMAGE_NAME} || echo 'app does not exist'"
                        command4="docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
                        ssh -t centos@${HOSTNAME_DEPLOY_PROD} \
                            -o SendEnv=IMAGE_NAME \
                            -o SendEnv=IMAGE_TAG \
                            -o SendEnv=DOCKERHUB_AUTH_USR \
                            -o SendEnv=DOCKERHUB_AUTH_PSW \
                            -C "${command1} && ${command2} && ${command3} && ${command4}"
                    '''
                }
            }
       stage('Test production') {
           agent any
           steps {
              script {
                    sh 'curl http://3.80.175.207:${PORT_EXPOSED} | grep -q "Hello world!"'
              }
           }
        }

    }
}
}
