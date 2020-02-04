pipeline {
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
                            sh 'mvn clean install'
                        }
                    }
                }
                stage ('Analyzing with SonarQube') {
                    steps {
                        sh 'mvn sonar:sonar'
                    }
                }
                stage('Deploy for production') {
                    when {
                        branch 'master'
                    }
                    steps {
                        script {
                            sh 'echo "Produción"'
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
