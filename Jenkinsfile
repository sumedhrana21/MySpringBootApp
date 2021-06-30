pipeline {
    agent any
    tools {
        jdk "jdk11"
        maven "mvn"
    }
    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Maven Tests') {
            steps {

                try {
                    sh 'mvn clean test'; junit '**/surefire-reports/*Test.xml'
                } catch(err) {
                    junit '**/surefire-reports/*Test.xml';
                    // if(currentBuild.result == 'UNSTABLE') { 
                    //     currentBuild.result = 'FAILURE'; 
                    //     throw err;
                    // };
                }
            }
        }

        stage('Sonar Scan') {
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=demo -Dsonar.host.url=http://ec2-3-142-132-142.us-east-2.compute.amazonaws.com:9000 -Dsonar.login=cb62d433bf1846d579f2879a8f080c77d4755b42'
            }
        }

        if (!env.CHANGE_TARGET) {

            stage('Maven Install') {
                steps {
                    sh 'mvn clean install -Dmaven.test.skip=true'
                }
            }

            stage('Docker build') {
                steps {
                    sh 'docker build -t pkuma343/myimage:${env.BUILD_NUMBER} -f Dockerfile .'
                }
            }

            stage('Push Image') {
                steps {
                    sh 'docker login -u "pkuma343" -p "Password"'
                    sh 'docker push pkuma343/myimage:${env.BUILD_NUMBER}'
                }
            }

            stage('Deploy') {
                steps{
                    sh 'echo "/path/to/kubectl apply -f api.yaml --kube-config=${PATH_TO_KUBE_CONFIG}"'
                }
            }
        }
    }
}
