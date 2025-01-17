pipeline {
    /**
    * Environmental variables
    */
    environment {
        /**
        * Id of Jenkins credentials
        */
        registryCredential = 'rgcolussi'
        /**
         * Credentials for Jenkis
         */
        USR_JENKINS = credentials('rgcolussi')
        /** 
        * Name of the image using for building the test image
        */
        IMAGE_NAME = 'nbch/esb-automated-test'

        /**
        * Project repository
        */
        PROJECT = 'https://github.com/rgcolussi/prueba.git'
        PROJECT_BRANCH = 'dev'
        /**
        * Data repository
        */
        DATA = 'https://github.com/rgcolussi/prueba.git'
        DATA_BRANCH = 'dev'
    }

    agent any

    stages {
        stage("Checkout SCM") {
            agent {
                docker {
                    image 'alpine'
                }
            }
            steps {
                dir('checkout') {
                    git branch: "${PROJECT_BRANCH}",
                    credentialsId: "${registryCredential}",
                    url: "${PROJECT}"
                }
                dir('data') {
                    git branch: "${DATA_BRANCH}",
                    credentialsId: "${registryCredential}",
                    url: "${DATA}"
                }
            }
        }
        stage("Build docker image tests") {
            options {
                skipDefaultCheckout()
            }
            agent {
                docker {
                    image 'docker'
                    args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'mv --verbose data/* checkout/project/samples/'
                dir('checkout/project') {
                    script {
                        docker.build('${IMAGE_NAME}:latest')
                    }
                }
                sh 'docker inspect --type=image ${IMAGE_NAME}:latest'
            }
        }
        stage("Run gradle tests") {
            options {
                skipDefaultCheckout()
            }
            agent {
                docker {
                    image '${IMAGE_NAME}:latest'
                    args '--entrypoint ""'
                }
            }
            
            steps {
                dir('checkout/project') {
                    sh 'gradle clean'
                    sh 'gradle --no-daemon AllTest'
                }
            }
            post {
                always {
                    dir('checkout/project/build/allure-results') {
                        stash 'allure-results'
                    }
                }
            }
        }
    }
    post {
        always {
            dir('allure-results') {
                unstash 'allure-results'
            }
            allure results: [[path: 'allure-results']]
            cleanWs()
        }
    }
}