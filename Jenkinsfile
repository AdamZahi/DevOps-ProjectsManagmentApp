pipeline {
    agent any

    /*
     * Si les outils Maven / JDK / NodeJS sont configurés dans
     * Jenkins -> Administrer Jenkins -> Gestion des outils, décommenter :
     *
     * tools {
     *     maven  'Maven 3'
     *     jdk    'JDK 21'
     *     nodejs 'NodeJS 22'
     * }
     */

    environment {
        // Préfixe des images Docker (adapter selon Docker Hub ou registry privé)
        IMAGE_BACKEND  = "gestion-projets-backend"
        IMAGE_FRONTEND = "gestion-projets-frontend"
        // Tag basé sur le numéro de build Jenkins (ex. :42) ou :latest
        IMAGE_TAG = "${env.BUILD_NUMBER ?: 'latest'}"
    }

    stages {

        // ── 1. Récupération du code source ─────────────────────────────
        stage('Checkout SCM') {
            steps {
                echo '=== [1/7] Checkout SCM ==='
                checkout scm
            }
        }

        // ── 2. Build du backend (Maven, packaging JAR) ─────────────────
        stage('Build Backend') {
            steps {
                echo '=== [2/7] Build Backend (Maven) ==='
                dir('backend') {
                    script {
                        if (isUnix()) {
                            sh 'mvn clean package -DskipTests -B'
                        } else {
                            bat 'mvn clean package -DskipTests -B'
                        }
                    }
                }
            }
        }

        // ── 3. Tests unitaires du backend ──────────────────────────────
        stage('Test Backend') {
            steps {
                echo '=== [3/7] Test Backend (JUnit) ==='
                dir('backend') {
                    script {
                        if (isUnix()) {
                            sh 'mvn test -B'
                        } else {
                            bat 'mvn test -B'
                        }
                    }
                }
            }
            post {
                always {
                    // Publier les résultats de tests dans l'interface Jenkins
                    junit allowEmptyResults: true,
                          testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        // ── 4. Build du frontend Angular ───────────────────────────────
        stage('Build Frontend') {
            steps {
                echo '=== [4/7] Build Frontend (Angular) ==='
                dir('frontend') {
                    script {
                        if (isUnix()) {
                            sh 'npm ci --legacy-peer-deps'
                            sh 'npm run build'
                        } else {
                            bat 'npm ci --legacy-peer-deps'
                            bat 'npm run build'
                        }
                    }
                }
            }
        }

        // ── 5. Archivage des artefacts frontend ────────────────────────
        stage('Archive Frontend Artifacts') {
            steps {
                echo '=== [5/7] Archive Frontend Artifacts ==='
                archiveArtifacts artifacts: 'frontend/dist/**',
                                  fingerprint: true,
                                  allowEmptyArchive: false
            }
        }

        // ── 6. Build des images Docker ─────────────────────────────────
        stage('Docker Build') {
            steps {
                echo '=== [6/7] Docker Build (backend + frontend) ==='
                script {
                    if (isUnix()) {
                        sh """
                            docker build -t ${IMAGE_BACKEND}:${IMAGE_TAG}  ./backend
                            docker build -t ${IMAGE_FRONTEND}:${IMAGE_TAG} ./frontend
                        """
                    } else {
                        bat """
                            docker build -t %IMAGE_BACKEND%:%IMAGE_TAG%  backend
                            docker build -t %IMAGE_FRONTEND%:%IMAGE_TAG% frontend
                        """
                    }
                }
            }
        }

        // ── 7. Déploiement via Docker Compose ──────────────────────────
        stage('Docker Deploy') {
            steps {
                echo '=== [7/7] Docker Compose Up ==='
                script {
                    if (isUnix()) {
                        sh '''
                            docker compose down --remove-orphans || true
                            docker compose up -d --build
                        '''
                    } else {
                        bat '''
                            docker compose down --remove-orphans || exit /b 0
                            docker compose up -d --build
                        '''
                    }
                }
            }
        }
    }

    // ── Post-exécution ────────────────────────────────────────────────
    post {
        success {
            echo """
            ──────────────────────────────────────────
             Pipeline terminé avec succès !
             Frontend  → http://localhost:80
             Backend   → http://localhost:8080
             Base de données → localhost:3306 / test_db
            ──────────────────────────────────────────
            """
        }
        failure {
            echo ' Pipeline en échec — vérifier les logs ci-dessus.'
        }
        always {
            // Afficher l'état des conteneurs en fin de pipeline
            script {
                if (isUnix()) {
                    sh 'docker compose ps || true'
                } else {
                    bat 'docker compose ps || exit /b 0'
                }
            }
        }
    }
}
