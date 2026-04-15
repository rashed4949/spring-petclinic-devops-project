pipeline {
    agent any

    tools {
        maven 'Maven'         // matches name in Global Tool Config
        jdk   'JDK17'
    }

    environment {
        ARTIFACTORY_URL      = 'http://164.90.215.116:8082/artifactory'
        ARTIFACTORY_REPO     = 'petclinic-libs-release'
        ARTIF_CREDS          = credentials('artifactory-creds')
        APP_VERSION          = "${env.BUILD_NUMBER}"
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
                sh 'mvn clean verify -DskipTests=false'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                rtMavenDeployer(
                    id: 'maven-deployer',
                    serverId: 'artifactory',            // matches Jenkins JFrog config ID
                    releaseRepo: "${ARTIFACTORY_REPO}"
                )
                rtMavenRun(
                    pom: 'pom.xml',
                    goals: 'install',
                    deployerId: 'maven-deployer',
                    buildName: 'petclinic',
                    buildNumber: "${APP_VERSION}"
                )
                rtPublishBuildInfo(
                    serverId: 'jfrog-artifactory',
                    buildName: 'petclinic',
                    buildNumber: "${APP_VERSION}"
                )
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh """
                    ansible-playbook \
                      -i /var/lib/jenkins/ansible/inventory.ini \
                      /var/lib/jenkins/ansible/deploy-petclinic.yml \
                      --extra-vars "app_version=3.3.0 artifactory_password=${ARTIF_CREDS_PSW}"
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
