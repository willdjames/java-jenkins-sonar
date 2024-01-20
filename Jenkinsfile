pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }

    stages {
        stage('Clone') {
            steps {
                // Get some code from a GitHub repository
                git branch:'main', url:'https://github.com/willdjames/java-jenkins-sonar.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn -Dmaven.test.failure.ignore=true clean compile"
            }
        }

        stage('Sonar Analise') {
            steps {
                sh "/var/jenkins_home/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner -Dsonar.host.url=http://sonar:9000 -Dsonar.token=squ_036dcc7c1d1b2e0a2ae6d3cd888c9c49ecdd6716"
                //  sh "/var/jenkins_home/sonar-scanner-5.0.1.3006-linux/bin/sonar-scanner -h"
            }
            
        }

        stage('Sonar Gste') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
    }
}
