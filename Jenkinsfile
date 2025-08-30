// Pipeline de CI/CD para aplicación Node.js con análisis de calidad y despliegue
pipeline {
    // Agente global que se ejecutará en cualquier nodo disponible
    agent any
    
    stages {
        // PRIMERA ETAPA: Instalación y preparación del entorno de desarrollo
        stage('Instalacion preliminar') {
            // Usar contenedor Docker con Node.js 22 para consistencia del entorno
            agent{
                docker{
                    image 'node:22'
                    reuseNode true  // Reutilizar el nodo para mantener el workspace
                }
            }
            stages {
                // Sub-etapa 1: Instalar dependencias del proyecto
                stage('Dependencias'){
                    steps{
                        sh 'npm install'  // Instalar todas las dependencias definidas en package.json
                    }
                }
                // Sub-etapa 2: Ejecutar pruebas unitarias y generar reporte de cobertura
                stage('Testing y cobertura') {
                    steps {
                        sh 'npm run test:cov'  // Ejecutar tests con reporte de cobertura de código
                    }
                }
                // Sub-etapa 3: Compilar y construir la aplicación
                stage('Build') {
                    steps {
                        sh 'npm run build'  // Compilar el código fuente para producción
                    }
                }
            }
        }
        
        // SEGUNDA ETAPA: Análisis de calidad del código con SonarQube
        stage('QA Sonarqube'){
            // Usar contenedor especializado de SonarQube Scanner
            agent{
                docker{
                    image 'sonarsource/sonar-scanner-cli'
                    args '--network devops-infra_default'  // Conectar a la red donde está SonarQube
                    reuseNode true
                }
            }
            stages{
                // Sub-etapa: Subir código a SonarQube para análisis
                stage('Upload a Sonarqube'){
                    steps{
                        // Configurar entorno de SonarQube usando credenciales configuradas
                        withSonarQubeEnv('SonarQube'){
                            sh 'sonar-scanner'  // Ejecutar análisis de calidad del código
                        }
                    }
                }
            }
        }
        
        // TERCERA ETAPA: Empaquetado y despliegue de la aplicación
        stage('Empaquetado y Delivery'){
            steps{
                script{
                    // Configurar registro Docker (Nexus) para almacenar la imagen
                    docker.withRegistry('http://localhost:8082', 'nexus-credentials'){	
                       sh 'docker tag backend-test localhost:8082/backend-test:${BUILD_NUMBER}'
					   sh 'docker tag backend-test localhost:8082/backend-test:latest'
					   sh 'docker push localhost:8082/backend-test:${BUILD_NUMBER}'
					   sh 'docker push localhost:8082/backend-test:latest'
                    }
                }
            }
        }
    }
}