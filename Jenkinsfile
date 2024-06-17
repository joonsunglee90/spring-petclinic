pipeline {
    agent any
    
    tools {
        jdk 'JDK17'
        maven 'M3'
    }
    
    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/joonsunglee90/spring-petclinic.git',
                branch: 'efficient-webjars'
            }
            post {
                success {
                    echo 'Success git clone step'
                }
                failure {
                    echo 'Fail git clone step'
                }
            }
        }
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true install'
            }
            post {
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        stage('Docker Image Build') {
            steps {
                echo 'Docker Image Build'
                
                dir("${env.WORKSPACE}") {
                    sh """
                        docker build -t joonsunglee90/spring-petclinic:$BUILD_NUMBER .
                        docker tag joonsunglee90/spring-petclinic:$BUILD_NUMBER joonsunglee90/spring-petclinic:latest
                    """    
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploy'
            }
        }
    }
}
