pipeline {

    agent any

    environment {

        registry = "pprasant/demo-app"
        registryCredentials = "dockerhub"
    }

    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('Build docker image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":vpro$BUILD_NUMBER"
                }
            }
        }

        stage('Upload image to dockerhub') {
            steps {
                script {
                    docker.withRegistry('', registryCredentials) {
                        dockerImage.push("vpro$BUILD_NUMBER")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Remove unused docker image') {
            steps {
                sh "docker rmi $registry:vpro$BUILD_NUMBER"
            }
        }

        stage('Kubernetes deployment') {
            agent {label 'KOPS'}
            steps {
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:vpro${BUILD_NUMBER} --namespace prod"
            }
        }


    }


}

