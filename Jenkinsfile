pipeline {
    agent {
        label 'Worker'
    }
    tools {
        maven 'Apache Maven 3.6'
        jdk 'OpenJDK 11 Latest'
    }
    stages {
        stage('Build') {
            steps {
                checkout scm
                sh """ \
                    ./mvnw -B clean verify
                """
            }
        }
        stage('Deploy image') {
            when {
                beforeAgent true
                not { changeRequest() }
            }
            environment {
                QUAY_CREDS = credentials('hibernate.quay.io')
            }
            steps {
                script {
                    if ( env.BRANCH_NAME == 'main' ) {
                        env.QUARKUS_CONTAINER_IMAGE_ADDITIONAL_TAGS = 'latest'
                    }
                    else {
                        env.QUARKUS_CONTAINER_IMAGE_ADDITIONAL_TAGS = env.BRANCH_NAME
                    }
                }
                sh '''
                    ./mvnw -B clean package -Dquarkus.container-image.build=true -Dquarkus.container-image.push=true \
                            -Dquarkus.container-image.username=${QUAY_CREDS_USR} \
                            -Dquarkus.container-image.password=${QUAY_CREDS_PSW} \
                '''
            }
        }
        stage('Deploy container') {
            when {
                beforeAgent true
                not { changeRequest() }
                branch 'build'
            }
            // Bots are hosted on the same machine as in.relation.to
            environment {
                SSH_CREDS = credentials('jenkins.in.relation.to')
            }
            steps {
                sh 'ssh -vvvvvv -i "${SSH_CREDS}" ${SSH_CREDS_USR}@in.relation.to echo foo'
            }
        }
    }
}
