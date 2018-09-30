pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }
        stage('Building image') {
            steps{
                script {
                docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
            steps {
                docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest")
                }
        }
        }
  }
}
