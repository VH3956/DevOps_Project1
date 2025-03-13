pipeline {
    agent any
    
    environment {
        SERVICE_CHANGED = ''
    }
    
    stages {
        stage('Check Changes') {
            steps {
                script {
                    def changedFiles = sh(script: "git diff --name-only origin/main", returnStdout: true).trim().split("\n")
                    def services = ['spring-petclinic-customers-service', 'spring-petclinic-vets-service', 'spring-petclinic-visits-service', 'spring-petclinic-genai-service']
                    
                    echo "Changed files: ${changedFiles}"
                    
                    if (changedFiles.size() == 0 || changedFiles[0] == '') {
                        echo "No changes detected. Skipping pipeline."
                        currentBuild.result = 'ABORTED'
                        return
                    }
                    
                    for (service in services) {
                        echo "Checking service: ${service}"
                        if (changedFiles.find { it.indexOf(service) >= 0 }) {
                            withEnv(["SERVICE_CHANGED=${detectedService}"])
                            echo "Matched service: ${env.SERVICE_CHANGED}"
                            break
                        }
                    }

                    
                    if (env.SERVICE_CHANGED == '') {
                        echo "No relevant service changes detected. Skipping pipeline."
                        currentBuild.result = 'ABORTED'
                        return
                    }
                    
                    echo "Changes detected in service: ${env.SERVICE_CHANGED}"
                }
            }
        }
        
        stage('Test') {
            when {
                expression { return env.SERVICE_CHANGED != '' }
            }
            steps {
                echo "Testing service: ${env.SERVICE_CHANGED}"
                sh "./mvnw test -pl ${env.SERVICE_CHANGED} -am"
            }
            post {
                always {
                    junit "${env.SERVICE_CHANGED}/target/surefire-reports/*.xml"
                }
            }
        }
        
        stage('Build') {
            when {
                expression { return env.SERVICE_CHANGED != '' }
            }
            steps {
                echo "Building service: ${env.SERVICE_CHANGED}"
                sh "./mvnw package -pl ${env.SERVICE_CHANGED} -am -DskipTests"
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully for service: ${env.SERVICE_CHANGED}"
        }
        failure {
            echo "Pipeline failed for service: ${env.SERVICE_CHANGED}"
        }
    }
}
