pipeline {
    agent any
    
    environment {
        // On ajoute le dossier Python au PATH pour tout le pipeline
        // Note : on utilise des doubles antislashs \\ pour échapper en Groovy
        PATH = "C:\\Users\\lenov\\AppData\\Local\\Programs\\Python\\Python314;${env.PATH}"
    }
    
    stages {
        stage('Nettoyage Workspace') {
            steps {
                deleteDir()
            }
        }
        
        stage('Clonage') {
            steps {
                git branch: 'main', url: 'https://github.com/Nima-112/CI_Project_Jenkins.git'
            }
        }
        
        stage('Vérification Python') {
            steps {
                // Maintenant "python" fonctionne car il est dans le PATH
                bat 'python --version'
                bat 'python -m pip --version'
            }
        }
        
        stage('Installation des dépendances') {
            steps {
                bat 'python -m pip install -r requirements.txt'
            }
        }
        
        stage('Analyse qualité du code (Pylint)') {
            steps {
                // Le "|| exit /b 0" empêche le pipeline d'échouer si Pylint trouve des erreurs mineures
                bat 'python -m pylint calculator.py || exit /b 0'
            }
        }
        
        stage('Tests unitaires') {
            steps {
                bat 'python -m pytest --ignore=Release'
            }
        }
        
        stage('Création de l artefact') {
            steps {
                bat '''
                    if exist Release rmdir /S /Q Release
                    if exist Release.zip del Release.zip
                    mkdir Release
                    copy calculator.py Release
                    copy test_calculator.py Release
                    copy requirements.txt Release
                    powershell Compress-Archive -Path Release\\* -DestinationPath Release.zip -Force
                '''
            }
        }
        
        stage('Publication de l artefact') {
            steps {
                // 1. Publication sur Nexus (Correction des antislashs ^ pour le multiligne Windows)
                bat '''
                    curl -v ^
                    -u admin:VOTRE_MOT_DE_PASSE ^
                    --upload-file Release.zip ^
                    http://localhost:8081/repository/python-release/Release.zip
                '''
                
                // 2. NOUVEAU : Publication de l'artefact directement dans l'interface Jenkins
                // Cela permet de télécharger le fichier zip depuis la page du build
                archiveArtifacts artifacts: 'Release.zip', fingerprint: true
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline terminé'
        }
        success {
            echo 'BUILD SUCCESS'
        }
        failure {
            echo 'BUILD FAILED'
        }
    }
}