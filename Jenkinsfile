pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'yadex34'
        BACKEND_IMAGE = "${DOCKER_HUB_USER}/sdle-backend:latest"
        FRONTEND_IMAGE = "${DOCKER_HUB_USER}/sdle-frontend:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 Récupération du code de déploiement...'
                checkout scm
            }
        }

        stage('Pull Latest Images') {
            steps {
                echo '🐳 Récupération des dernières images depuis Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker pull ${BACKEND_IMAGE}
                        docker pull ${FRONTEND_IMAGE}
                        docker logout
                    '''
                }
            }
        }

        stage('Stop Existing Services') {
            steps {
                echo '🛑 Arrêt des services existants...'
                sh '''
                    docker compose down --remove-orphans || true
                '''
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo '🚀 Déploiement de l\'application avec Docker Compose...'
                sh '''
                    docker compose up -d --force-recreate
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo '🏥 Vérification de la santé des services...'
                sh '''
                    echo "⏳ Attente du démarrage des services..."
                    sleep 30

                    echo "🔍 Vérification du Backend..."
                    curl -sf http://localhost:8081/actuator/health || echo "⚠️ Backend pas encore prêt"

                    echo ""
                    echo "🔍 Vérification du Frontend..."
                    curl -sf http://localhost:3000 > /dev/null && echo "✅ Frontend OK" || echo "⚠️ Frontend pas encore prêt"

                    echo ""
                    echo "🔍 Vérification de PostgreSQL..."
                    docker exec sdle-postgres pg_isready -U sgle_user -d sgle_db && echo "✅ PostgreSQL OK" || echo "⚠️ PostgreSQL pas encore prêt"

                    echo ""
                    echo "📊 État des conteneurs :"
                    docker compose ps
                '''
            }
        }
    }

    post {
        success {
            echo '''
            ✅ Déploiement terminé avec succès !
            🌐 Frontend : http://localhost:3000
            🔧 Backend API : http://localhost:8081
            🗄️ pgAdmin : http://localhost:5050
            '''
        }
        failure {
            echo '❌ Déploiement échoué. Vérifiez les logs.'
            sh 'docker compose logs --tail=50 || true'
        }
    }
}
