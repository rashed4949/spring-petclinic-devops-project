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
                        releaseRepo: "${ARTIFACTORY_REPO}"
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
                withCredentials([
                    usernamePassword(
                        credentialsId: 'artifactory-creds',
                        usernameVariable: 'ART_USER',
                        passwordVariable: 'ART_PASS'
                    ),
                    sshUserPrivateKey(
                        credentialsId: 'ansible-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ansible-playbook \
                        -i /var/lib/jenkins/ansible/inventory.ini \
                        /var/lib/jenkins/ansible/deploy-petclinic.yml \
                        --private-key $SSH_KEY \
                        --extra-vars "app_version=${APP_VERSION} artifactory_user=$ART_USER artifactory_password=$ART_PASS"
                    '''
                }
            }
        }

    post {
        success {
            echo "PetClinic deployed successfully at http://139.59.157.224"
        }

        failure {
            echo "Pipeline failed — check logs above"
        }
    }
}
