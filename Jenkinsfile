pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_NAME', defaultValue: 'ghcr.io/ramoswilly/yolo-api', description: 'Full Docker image name from registry')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag to deploy')
    }

    stages {
        // --- ETAPA 1: Aprobación Manual para Producción ---
        stage('Approve for Production') {
            steps {
                script {
                    // Se formatea el mensaje para que el aprobador sepa exactamente qué está aprobando
                    def message = "Aprobar despliegue de la imagen ${params.IMAGE_NAME}:${params.IMAGE_TAG} a Producción."
                    timeout(time: 7, unit: 'DAYS') {
                        input id: 'DeployGate', message: message
                    }
                }
            }
        }

        // --- ETAPA 2: Desplegar a Producción ---
        stage('Deploy to Production') {
            steps {
                script {
                    echo "Clonando repositorio de configuración..."
                    // Jenkins hace automaticamente
                }

                dir('kubernetes/overlays/production') {
                    script {
                        echo "Actualizando imagen para Producción a: ${params.IMAGE_NAME}:${params.IMAGE_TAG}"
                        // Kustomize edita la imagen con el nombre completo y el nuevo tag.
                        sh "kustomize edit set image ${params.IMAGE_NAME}=${params.IMAGE_NAME}:${params.IMAGE_TAG}"
                    }
                }
                echo "Aplicando configuración de Producción en Kubernetes..."
                sh "sudo kubectl apply -k kubernetes/overlays/production/"

		// --- PASO CLAVE AÑADIDO ---
                echo "Forzando el reinicio del deployment para aplicar la nueva imagen..."
                // 3. Fuerza el reinicio del deployment para asegurar que los nuevos pods se creen.
                sh "sudo kubectl rollout restart deployment backend-deployment -n production"
            }
        }
    }

    post {
        always {
            echo "Limpieza de workspace..."
            cleanWs()
        }
    }
}
