pipeline {
    agent any

    environment {
        // El tag único se basa en el hash del commit de Git
        IMAGE_TAG = env.GIT_COMMIT.substring(0, 8)
        // El nombre completo de la imagen con su tag
        FULL_IMAGE_NAME = "yolo-api:${IMAGE_TAG}"
        // Nombre del archivo .tar para la imagen
        IMAGE_TAR = "${IMAGE_TAG}.tar"
    }

    stages {
        // --- ETAPA 1: Construir la Imagen Docker ---
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Construyendo la imagen: ${FULL_IMAGE_NAME}"
                    sh "sudo docker build -t ${FULL_IMAGE_NAME} ."
                }
            }
        }

        // --- ETAPA 2: Importar Imagen a k3s ---
        stage('Import Image to k3s') {
            steps {
                script {
                    echo "Guardando e importando imagen a k3s..."
                    sh "sudo docker save -o ${IMAGE_TAR} ${FULL_IMAGE_NAME}"
                    sh "sudo k3s ctr images import ${IMAGE_TAR}"
                }
            }
        }

        // --- ETAPA 3: Desplegar a Staging con Kustomize ---
        stage('Deploy to Staging') {
            steps {
                // El bloque 'dir' ejecuta los comandos dentro del directorio especificado
                dir('kubernetes/overlays/staging') {
                    script {
                        echo "Actualizando el tag de la imagen para Staging a: ${IMAGE_TAG}"
                        // Se usa 'kustomize edit' para cambiar el tag de forma segura y declarativa.
                        // Busca la imagen con 'name: yolo-api' y le asigna el nuevo tag.
                        sh "kustomize edit set image yolo-api:${IMAGE_TAG}"
                    }
                }
                echo "Aplicando configuración de Staging en Kubernetes..."
                // Se usa 'kubectl apply -k' que entiende Kustomize nativamente.
                sh "sudo kubectl apply -k kubernetes/overlays/staging/"
            }
        }

        // --- ETAPA 4: Aprobación Manual ---
        stage('Approve for Production') {
            steps {
                timeout(time: 7, unit: 'DAYS') {
                    input message: "La aplicación ha sido desplegada en Staging. ¿Aprobar para producción?"
                }
            }
        }

        // --- ETAPA 5: Desplegar a Producción con Kustomize ---
        stage('Deploy to Production') {
            steps {
                dir('kubernetes/overlays/production') {
                    script {
                        echo "Actualizando el tag de la imagen para Producción a: ${IMAGE_TAG}"
                        // Se usa la misma imagen probada en Staging para el despliegue en Producción.
                        sh "kustomize edit set image yolo-api:${IMAGE_TAG}"
                    }
                }
                echo "Aplicando configuración de Producción en Kubernetes..."
                sh "sudo kubectl apply -k kubernetes/overlays/production/"
            }
        }
    }

    // --- ETAPA FINAL: Limpieza ---
    post {
        always {
            script {
                echo "Ejecutando limpieza de disco..."
                sh "rm -f ${IMAGE_TAR}"
                sh "sudo docker system prune -a -f"
                sh "sudo crictl rmi --prune"
                echo "Limpieza finalizada."
            }
        }
    }
}
