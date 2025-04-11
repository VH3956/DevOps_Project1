pipeline {
    agent none  // Không chạy trên Master, chỉ điều phối

    environment {
        DOCKER_HUB_CREDS = credentials('docker-hub-cre')
        APP_NAME = 'spring-petclinic-'
        DOCKER_IMAGE = "vh3956/${APP_NAME}"
    }

    stages {
        // stage('Check Changes') {
        //     agent { label 'built-in' } // Chạy trên Master
        //     steps {
        //         script {
        //             echo "Commit SHA: ${GIT_COMMIT}"
        //             def changedFiles = []
        //             env.NO_SERVICES_TO_BUILD = 'false'
        //             if (env.CHANGE_TARGET) {
        //                 changedFiles = sh(script: "git diff --name-only HEAD^", returnStdout: true).trim().split('\n').toList()
        //             } else {
        //                 changedFiles = sh(script: "git diff --name-only HEAD^", returnStdout: true).trim().split('\n').toList()
        //             }

        //             def services = ['spring-petclinic-customers-service', 'spring-petclinic-visits-service', 'spring-petclinic-vets-service']
                    
        //             echo "Changed files: ${changedFiles}"

        //             if (changedFiles.isEmpty() || changedFiles[0] == '') {
        //                 echo "No changes detected. Skipping pipeline."
        //                 currentBuild.result = 'ABORTED'
        //                 return
        //             }

        //             def detectedServices = []
        //             for (service in services) {
        //                 if (changedFiles.any { it.startsWith(service + '/') }) {
        //                     detectedServices << service
        //                 }
        //             }

        //             if (detectedServices.isEmpty()) {
        //                 echo "No relevant service changes detected. Skipping pipeline."
        //                 env.NO_SERVICES_TO_BUILD = 'true'
        //             } else {
        //                 echo "Detected Services: ${detectedServices}"
        //                 env.SERVICE_CHANGED = detectedServices.join(",")
        //             }
        //         }
        //     }
        // }

        // stage('Test & Coverage - Agent 1') {
        //     agent { label 'agent1' }  
        //     when {
        //         expression { env.NO_SERVICES_TO_BUILD == 'false' && (env.SERVICE_CHANGED.contains('customers-service') || env.SERVICE_CHANGED.contains('visits-service')) }
        //     }
        //     steps {
        //         script {
        //             def services = env.SERVICE_CHANGED.split(',').findAll { it in ['spring-petclinic-customers-service', 'spring-petclinic-visits-service'] }
        //             for (service in services) {
        //                 echo "Running unit tests for service: ${service} on Agent 1"
        //                 sh "./mvnw clean verify -pl ${service} -am"
        //                 stash name: "${service}-coverage", includes: "${service}/target/site/jacoco/**"
        //             }
        //         }
        //     }
        //     post {
        //         always {
        //             script {
        //                 def services = env.SERVICE_CHANGED.split(',')
        //                 for (service in services) {
        //                     junit "${service}/target/surefire-reports/*.xml"
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Test & Coverage - Agent 2') {
        //     agent { label 'agent2' }  
        //     when {
        //         expression { env.NO_SERVICES_TO_BUILD == 'false' && env.SERVICE_CHANGED.contains('vets-service') }
        //     }
        //     steps {
        //         script {
        //             echo "Running unit tests for vets-service on Agent 2"
        //             sh "./mvnw clean verify -pl spring-petclinic-vets-service -am"
        //             stash name: "spring-petclinic-vets-service-coverage", includes: "spring-petclinic-vets-service/target/site/jacoco/**"
        //         }
        //     }
        //     post {
        //         always {
        //             junit "spring-petclinic-vets-service/target/surefire-reports/*.xml"
        //         }
        //     }
        // }

        // stage('Check Coverage') {
        //     agent { label 'built-in' }
        //     when {
        //         expression { env.NO_SERVICES_TO_BUILD == 'false' }
        //     }
        //     steps {
        //         script {
        //             def services = env.SERVICE_CHANGED.split(',')
        //             def failedCoverageServices = []

        //             for (service in services) {
        //                 unstash "${service}-coverage"
        //             }

        //             for (service in services) {
        //                 def coverageHtml = sh(
        //                     script: "xmllint --html --xpath 'string(//table[@id=\"coveragetable\"]/tfoot/tr/td[3])' ${service}/target/site/jacoco/index.html 2>/dev/null",
        //                     returnStdout: true
        //                 ).trim()

        //                 def coverage = coverageHtml.replace('%', '').toFloat() / 100
        //                 echo "Test Coverage for ${service}: ${coverage * 100}%"

        //                 if (coverage < 0.70) {
        //                     failedCoverageServices << service
        //                 }
        //             }

        //             if (failedCoverageServices.size() > 0) {
        //                 error "Coverage below 70% for services: ${failedCoverageServices.join(', ')}! Pipeline failed."
        //             }
        //         }
        //     }
        // }


        stage('Build and Push All Service Images') {
            agent { label 'built-in' }
            steps {
                script {
                    def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def versionTag = "3.4.1"

                    echo "Building all service images using: ./mvnw clean install -P buildDocker"
                    sh './mvnw clean install -P buildDocker'

                    def services = [
                        'admin-server',
                        'api-gateway',
                        'config-server',
                        'customers-service',
                        'discovery-server',
                        'vets-service',
                        'visits-service',
                        'genai-service',
                    ]

                    sh 'docker images'
                    echo "Docker images before tagging and pushing: ${services}"

                    sh "echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin"

                    for (svc in services) {
                        def image = "${DOCKER_IMAGE}-${svc}"
                        echo "Pushing image: ${image}:${commitId}"
                        sh """
                            docker tag ${image} ${image}:${commitId}
                            docker push ${image}:${commitId}

                            docker tag ${image} ${image}:latest
                            docker push ${image}:latest
                        """
                    }
                    echo "All images successfully built and pushed."
                }
            }
        }


    }
    post {
        success {
            script {
                currentBuild.description = "Built image: ${DOCKER_IMAGE}:${env.BUILD_IMAGE_TAG ?: 'N/A'}"
                echo "Pipeline completed successfully"
            }
        }
        failure {
            echo "Pipeline failed - Check logs for details"
        }
        aborted {
            echo "Pipeline was aborted - No changes detected"
        }
    }
}