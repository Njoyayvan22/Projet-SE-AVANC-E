pipeline {
    agent any

    environment {
        // Configuration Git
        REPO_URL = "git@github.com:Njoyayvan22/Projet-SE-AVANC-E.git"  // ou HTTPS avec token
        BRANCH = "main"

        // Outils (adaptez les chemins)
        NODEJS_HOME = "/usr/bin/node"  // Pour les projets avec npm/yarn
        NGINX_CONF_DIR = "/etc/nginx/conf.d"  // Pour le déploiement Nginx
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        // Étape 1 - Checkout du code
        stage('Checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: env.BRANCH]],
                        extensions: [],
                        userRemoteConfigs: [[
                            url: env.REPO_URL,
                            credentialsId: 'github-ssh'  // ID de vos credentials
                        ]]
                    ])
                }
            }
        }

        // Étape 2 - Installer les dépendances (si projet Node.js)
        stage('Install') {
            when {
                expression { fileExists('package.json') }
            }
            steps {
                script {
                    try {
                        sh """
                            npm install --silent
                            # ou yarn install --frozen-lockfile
                        """
                    } catch (err) {
                        error("Échec de l'installation : ${err.message}")
                    }
                }
            }
        }

        // Étape 3 - Build (si nécessaire)
        stage('Build') {
            when {
                expression { fileExists('package.json') }
            }
            steps {
                script {
                    try {
                        sh """
                            npm run build
                            # ou yarn build
                        """
                        archiveArtifacts artifacts: 'dist/**/*'  // Archive le dossier de build
                    } catch (err) {
                        error("Échec du build : ${err.message}")
                    }
                }
            }
        }

        // Étape 4 - Tests (optionnelle)
        stage('Test') {
            when {
                expression { fileExists('package.json') }
            }
            steps {
                script {
                    sh """
                        npm run test -- --watchAll=false
                        # ou yarn test
                    """
                }
            }
        }

        // Étape 5 - Déploiement (exemple avec Nginx)
        stage('Deploy') {
            when {
                branch 'main'  // Seulement sur la branche principale
            }
            steps {
                script {
                    try {
                        // 1. Copier les fichiers vers le serveur
                        sshagent(['ssh-deploy-key']) {
                            sh """
                                rsync -avz --delete ./dist/ user@server:/var/www/html/
                                # Ou pour un site statique simple :
                                # rsync -avz --delete ./ user@server:/var/www/html/
                            """
                        }

                        // 2. Redémarrer Nginx (optionnel)
                        sh """
                            ssh user@server "sudo systemctl restart nginx"
                        """
                    } catch (err) {
                        error("Échec du déploiement : ${err.message}")
                    }
                }
            }
        }
    }

    // Post-traitement
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                color: "good",
                message: "Déploiement réussi : ${env.JOB_NAME} - ${env.BUILD_URL}"
            )
        }
        failure {
            slackSend(
                color: "danger",
                message: "ÉCHEC : ${env.JOB_NAME} - Consulter ${env.BUILD_URL}"
            )
        }
    }
}
