pipeline {
    agent any

    environment {
        ID_DOCKER = "${ID_DOCKER_PARAMS}"
        IMAGE_NAME = "jk-flask-auth-tp"
        IMAGE_TAG = "latest"
        DOCKERHUB_PASSWORD = "${DOCKERHUB_PASSWORD_PARAMS}"
        RENDER_API_TOKEN = credentials('RENDER_API_TOKEN')
        RENDER_SERVICE_ID = "srv-coplmvv79t8c7380lqjg"
        RENDER_DEPLOY_HOOK_URL = credentials('RENDER_DEPLOY_HOOK_URL_TP3')
    }

    triggers {
        pollSCM('H/5 * * * *') // check every 5 minutes
    }

    stages {
        stage('Build image') {
            steps {
                script {
                    sh 'docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Test image') {
            steps {
                script {
                    echo "Exécution des tests"
                }
            }
        }

        stage('Login and Push Image on docker hub') {
            steps {
                script {
                    sh '''
                        echo ${DOCKERHUB_PASSWORD} | docker login -u ${ID_DOCKER} --password-stdin
                        docker push ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Trigger Deploy to Render') {
            steps {
                withCredentials([string(credentialsId: 'RENDER_DEPLOY_HOOK_URL', variable: 'DEPLOY_HOOK_URL')]) {
                    script {
                        sh "curl -X POST ${DEPLOY_HOOK_URL}"
                    }
                }
            }
        }

    }

    post {
        success {
            mail to: 'markus.gapaz@gmail.com', // notif-jenkins@joelkoussawo.me
                 subject: "Succès du Pipeline ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "Le pipeline a réussi. L'application a été déployée sur Render."
        }
        failure {
            mail to: 'markus.gapaz@gmail.com', // notif-jenkins@joelkoussawo.me
                 subject: "Échec du Pipeline ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "Le pipeline a échoué. Veuillez vérifier Jenkins pour plus de détails."
        }
    }
}