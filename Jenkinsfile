pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'     // noms configurés dans Administrer Jenkins -> Tools (atelier 3)
        maven 'M2_HOME'
    }

    environment {
        DOCKERHUB_USER = 'ihebbns'          // ton nom d'utilisateur Docker Hub
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build Backend (Maven)') {
            steps {
                dir('backend') { sh 'mvn -B clean package -DskipTests' }
            }
        }

        stage('Tests Backend') {
            steps {
                dir('backend') { sh 'mvn -B test' }
            }
            post {
                always { junit allowEmptyResults: true, testResults: 'backend/target/surefire-reports/*.xml' }
            }
        }

        stage('Docker Build') {
            steps { sh 'docker compose build' }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                    sh 'docker compose push'
                    sh '''
                        for svc in backend frontend; do
                          img="$DOCKERHUB_USER/benyounesiheb-5erp-bi1-gestionprojets-$svc"
                          docker tag "$img:$IMAGE_TAG" "$img:latest"
                          docker push "$img:latest"
                        done
                    '''
                }
            }
        }

        stage('Deploy (Docker Compose)') {
            steps { sh 'docker compose up -d' }
        }
    }

    post {
        always  { sh 'docker logout || true' }
        success { echo 'Application : http://<IP_VM>:4200  |  API : http://<IP_VM>:8089/entreprise/all' }
    }
}
