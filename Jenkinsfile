pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    triggers {
        githubPush()
    }

    environment {
        ARTIFACTORY_URL  = 'http://164.90.157.224:8082/artifactory'
        ARTIFACTORY_REPO = 'petclinic-libs-release'
        APP_VERSION      = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rashed4949/spring-petclinic-devops-project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests -Dcheckstyle.skip=true'
            }
        }

        stage('Upload to Artifactory') {
            steps {
                script {
                    // Auto-detect correct JAR (ignore .original)
                    def JAR_NAME = sh(
                        script: "ls target/*.jar | grep -v original | head -1",
                        returnStdout: true
                    ).trim()

                    withCredentials([usernamePassword(
                        credentialsId: 'artifactory-creds',
                        usernameVariable: 'ART_USER',
                        passwordVariable: 'ART_PASS'
                    )]) {
                        sh """
                            curl -u $ART_USER:$ART_PASS -T ${JAR_NAME} \
                            ${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}/petclinic-${APP_VERSION}.jar
                        """
                    }
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
                    )
                ]) {
                    sh """
                        ansible-playbook \
                        -i /var/lib/jenkins/ansible/inventory.ini \
                        /var/lib/jenkins/ansible/deploy-petclinic.yml \
                        --extra-vars "app_version=${APP_VERSION} artifactory_user=$ART_USER artifactory_password=$ART_PASS"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful 🚀 Version: ${APP_VERSION}"
        }
        failure {
            echo "Pipeline failed ❌"
        }
    }
}
