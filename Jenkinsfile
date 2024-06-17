pipeline {
    agent any
    
    tools {
        jdk 'JDK17'
        maven 'M3'
    }
    environment {
        // jenkins에 등록해 놓은 docker hub credentials 이름
        DOCKERHUB_CREDENTIALS = credentials('Dockerhub-jenkins')
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
        stage('Docker Login') {
            steps {
                // docker hub 로그인
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage ('Docker Image Push') {
            steps {
                // docker hub에 이미지 업로드
                sh 'docker push joonsunglee90/spring-petclinic:latest'
            }
        }
        stage ('Docker Image Remove') {
            steps {
                // docker image 삭제
                sh """
                docker rmi joonsunglee90/spring-petclinic:$BUILD_NUMBER
                docker rmi joonsunglee90/spring-petclinic:latest
                """
            }
        }
        stage('SSH Publish') {
            steps {
                echo 'SSH Publish'
                sshPublisher(publishers: [sshPublisherDesc(configName: 'target', 
                transfers: [sshTransfer(cleanRemote: false, 
                excludes: '', 
                execCommand: '''
                docker rm -f $(docker ps -aq)
                docker rmi $(docker images -q)
                docker pull joonsunglee90/spring-petclinic:latest
                docker run -d -p 80:8080 --name spring-petclinic joonsunglee90/spring-petclinic:latest
                ''', 
                execTimeout: 120000, 
                flatten: false, 
                makeEmptyDirs: false, 
                noDefaultExcludes: false, 
                patternSeparator: '[, ]+', 
                remoteDirectorySDF: false, 
                removePrefix: 'target', 
                usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, verbose: false)])])
            }
        }
    }
}
