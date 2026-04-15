pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    environment {
        ARTIFACTORY_URL  = 'http://164.90.215.116:8082/artifactory'
        ARTIFACTORY_REPO = 'petclinic-libs-release'
        APP_VERSION      = "${env.BUILD_NUMBER}"
        ARTIF_CREDS      = credentials('artifactory-creds')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rashed4949/spring-petclinic-devops-project.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                    mvn clean install -DskipTests -Dcheckstyle.skip=true
                '''
            }

            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                script {
                    rtMavenDeployer(
                        id: 'maven-deployer',
                        serverId: 'artifactory',
                        releaseRepo: "${ARTIFACTORY_REPO}",
                        snapshotRepo: "petclinic-libs-snapshot"
                    )

                    rtMavenRun(
                        pom: 'pom.xml',
                        goals: 'clean install -DskipTests -Dcheckstyle.skip=true',
                        deployerId: 'maven-deployer',
                        buildName: 'petclinic',
                        buildNumber: "${APP_VERSION}"
                    )

                    rtPublishBuildInfo(
                        serverId: 'artifactory',
                        buildName: 'petclinic',
                        buildNumber: "${APP_VERSION}"
                    )
                }
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh """
                    ansible-playbook \
                    -i /var/lib/jenkins/ansible/inventory.ini \
                    /var/lib/jenkins/ansible/deploy-petclinic.yml \
                    --extra-vars "app_version=${APP_VERSION} artifactory_password=${ARTIF_CREDS_PSW}"
                """
            }
        }
    }

    post {
        success {
            echo "PetClinic deployed successfully at http://139.59.157.224:80"
        }

        failure {
            echo "Pipeline failed — check logs above"
        }
    }
}
