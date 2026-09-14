pipeline {
    agent any

    environment {
        BRANCH_NAME = "master"
        DOCKER_ID = "dly17"
        DOCKER_IMAGE = "jenkins-exam"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {
        stage('Docker Build') {
            steps {
                sh '''
                docker rm -f jenkins || true
                docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG cast-service/
                '''
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Deploy Dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                sh '''
                rm -Rf ~/.kube/
                mkdir -p ~/.kube/
                cat $KUBECONFIG > ~/.kube/config
                cp charts/values.yaml values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ./charts --values=values.yml --namespace dev
                '''
            }
        }

        stage('Deploy QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                sh '''
                rm -Rf ~/.kube/
                mkdir -p ~/.kube/
                cat $KUBECONFIG > ~/.kube/config
                cp charts/values.yaml values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ./charts --values=values.yml --namespace qa
                '''
            }
        }

        stage('Deploy Staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                sh '''
                rm -Rf ~/.kube/
                mkdir -p ~/.kube/
                cat $KUBECONFIG > ~/.kube/config
                cp charts/values.yaml values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ./charts --values=values.yml --namespace staging
                '''
            }
        }

        stage('Deploy Prod') {
            when {
                anyOf {
                    branch 'master'
                    expression { return env.GIT_BRANCH == 'origin/master' || env.GIT_BRANCH == 'master' }
                }
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Déployer en Prod ?', ok: 'Oui'
                }
                sh '''
                rm -Rf ~/.kube/
                mkdir -p ~/.kube/
                cat $KUBECONFIG > ~/.kube/config
                cp charts/values.yaml values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ./charts --values=values.yml --namespace prod
                '''
            }
        }
    }
}
