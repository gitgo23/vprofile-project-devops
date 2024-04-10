COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    tools {
        maven "Maven-3.9.6"
    }

    environment {
        ScannerHome = tool 'Sonar-5'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'vp-rem', url: 'https://github.com/gitgo23/vprofile-project-devops.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "mvn clean package"
            }
        }

        /*
        stage('Code Analysis with Sonarqube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }*/

        stage('Scan with Sonarqube') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'SONAR-TK') {
                     sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile -Dsonar.projectName=ecommerce-success -Dsonar.java.binaries=." 
                         }
                }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage('Scan Dependencies with Owasp') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications'
            slackSend channel: 'jenkins-slack-notification', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "${currentBuild.currentResult}: Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            
            // Send email notification
            emailext subject: "Pipeline ${currentBuild.result}: ${env.JOB_NAME}",
                      body: "Build ${env.BUILD_NUMBER} ${currentBuild.result}\n\nCheck console output at: ${env.BUILD_URL}",
                      to: "www.gyenoch@gmail.com, enoch3054@gmail.com",
                      attachLog: true
        }
    }
}