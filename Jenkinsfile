pipeline {
    agent any

    /* 
     * Décommenter et adapter si des outils sont configurés dans 
     * Jenkins -> Administrer Jenkins -> Gestion des outils (Global Tool Configuration)
     *
     * tools {
     *     maven 'Maven 3'
     *     jdk 'JDK 17'
     *     nodejs 'NodeJS'
     * }
     */

    stages {
        // Étape 1 : Récupération du code source depuis le dépôt Git
        stage('Checkout SCM') {
            steps {
                echo '=== Checkout SCM ==='
                checkout scm
            }
        }

        // Étape 2 : Compilation et packaging du backend Spring Boot (sans exécuter les tests)
        stage('Build Backend') {
            steps {
                echo '=== Building Backend (Maven) ==='
                dir('backend') {
                    script {
                        if (isUnix()) {
                            sh 'mvn clean package -DskipTests'
                        } else {
                            bat 'mvn clean package -DskipTests'
                        }
                    }
                }
            }
        }

        // Étape 3 : Exécution des tests unitaires du backend
        stage('Test Backend') {
            steps {
                echo '=== Running Backend Tests ==='
                dir('backend') {
                    script {
                        if (isUnix()) {
                            sh 'mvn test'
                        } else {
                            bat 'mvn test'
                        }
                    }
                }
            }
        }

        // Étape 4 : Installation des dépendances et compilation du frontend Angular
        stage('Build Frontend') {
            steps {
                echo '=== Building Frontend (Angular) ==='
                dir('frontend') {
                    script {
                        if (isUnix()) {
                            sh 'npm install'
                            sh 'npm run build'
                        } else {
                            bat 'npm install'
                            bat 'npm run build'
                        }
                    }
                }
            }
        }

        // Étape 5 : Archivage des artefacts générés pour le frontend
        stage('Archive Frontend Artifacts') {
            steps {
                echo '=== Archiving Frontend Artifacts ==='
                // Archive tous les fichiers produits dans le dossier dist du frontend
                archiveArtifacts artifacts: 'frontend/dist/**', fingerprint: true, allowEmptyArchive: false
            }
        }
    }

    // Gestion des statuts post-exécution
    post {
        success {
            echo ' Pipeline Jenkins terminé avec succès !'
        }
        failure {
            echo ' Échec du pipeline Jenkins.'
        }
    }
}
