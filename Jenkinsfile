pipeline {
    environment {
        registry = "karlosarr/myshuttle"
        registryCredential = 'dockerhub'
    }
    agent {
        label "swarm"
    }
    stages {
        stage('preparing docker') {
            agent {
                docker { 
                    image 'maven:3.6.2-jdk-8'
                    args '--entrypoint=\'\' -v ${PWD}:/usr/src/app -w /usr/src/app'
                    reuseNode true
                }
            }
            stages {
                stage ('Install') {
                    steps {
                        script {
                            sh 'mvn -DskipITs --settings ./maven/settings.xml clean install'
                        }
                    }
                }
                stage ('Analyzing with SonarQube') {
                    steps {
                        sh 'mvn -DskipITs --settings ./maven/settings.xml sonar:sonar'
                    }
                }
                stage('Deploy for production') {
                    when {
                        branch 'master'
                    }
                    stages {
                        stage('Building binary') {
                            steps {
                                script {
                                    sh 'mvn -DskipITs --settings ./maven/settings.xml clean package'
                                }
                                step([$class: 'TeamCollectResultsPostBuildAction'])
                            }
                            post {
                                always {
                                    archiveArtifacts artifacts: 'target/*.war, *.sql', onlyIfSuccessful: true
                                }
                            }
                        }
                        stage('Building image') {
                            steps{
                                script {
                                    dockerImage = docker.build registry + ":v1.0.$BUILD_NUMBER"
                                    dockerImage = docker.build registry + ":latest"
                                }
                            }
                        }
                        stage('Deploy Image') {
                            steps{
                                script {
                                    docker.withRegistry( '', registryCredential ) {
                                        dockerImage.push()
                                    }
                                }
                            }
                        }
                        stage('Remove Unused docker image') {
                            steps{
                                sh "docker rmi $registry:v1.0.$BUILD_NUMBER"
                                sh "docker rmi $registry:latest"
                            }
                        }
                    }
                }
            }
        }
        stage('Clean') {
            steps {
                deleteDir()
            }
        }
    }
}
