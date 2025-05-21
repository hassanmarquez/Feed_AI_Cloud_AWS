# FeedAI en AWS - Implementación paso a paso

Este documento describe cómo desplegar una versión escalable de FeedAI en AWS, usando contenedores para OpenWebUI, instancias EC2 para Ollama y OpenSearch para la indexación de documentos.

---

## Requisitos previos

- Cuenta en AWS
- AWS CLI configurado
- Docker + ECS CLI (o AWS Copilot)
- Terraform o consola AWS opcional

---

## 1. Crear un bucket de Amazon S3

```bash
aws s3 mb s3://feedai-user-files
```

Este bucket almacenará los archivos cargados por los usuarios.

---

## 2. Desplegar OpenWebUI en Amazon ECS

### a. Crear una definición de tarea para ECS

```json
{
  "family": "openwebui-task",
  "containerDefinitions": [
    {
      "name": "openwebui",
      "image": "ghcr.io/open-webui/open-webui:main",
      "portMappings": [
        {
          "containerPort": 8080,
          "hostPort": 8080
        }
      ],
      "environment": [
        {
          "name": "OLLAMA_API_BASE_URL",
          "value": "http://<ollama-ec2-ip>:11434"
        }
      ]
    }
  ]
}
```

### b. Crear un ALB para enrutar el tráfico al servicio ECS

- Usa la consola o Terraform para crear un Application Load Balancer y apuntarlo al grupo de destino del servicio ECS.

---

## 3. Desplegar Ollama en una instancia EC2

### a. Lanzar una instancia EC2 (recomendado con GPU)

- Tipo: `g5.xlarge`, `p3.2xlarge`, etc.
- Sistema operativo: Ubuntu 20.04

### b. Instalar Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run llama3
```

Asegúrate de que el puerto `11434` esté abierto en el grupo de seguridad.

---

## 4. Configurar OpenSearch para indexación RAG

- Desplegar un dominio OpenSearch desde la consola de AWS
- Crear un índice llamado `feedai-context`
- Agregar permisos de escritura desde OpenWebUI (si es necesario, via IAM)

---

## 5. Integrar Amazon S3 para archivos

- Asegúrate de que OpenWebUI tenga acceso a S3 (vía credenciales IAM)
- Usa SDK de AWS o librerías en el backend de OpenWebUI para:
  - Subir archivos a `s3://feedai-user-files`
  - Extraer contenido y generar embeddings
  - Indexar en OpenSearch

---

## 6. Seguridad y monitoreo

- **IAM Roles**: para acceso seguro entre servicios
- **CloudWatch**: para logs y métricas
- **GuardDuty**: para detección de amenazas
- **ACM + HTTPS**: para cifrado con el Load Balancer

---

## 7. Opcional: Almacenamiento de inferencias

- Crear un segundo bucket o usar una base de datos (DynamoDB o RDS)
- Guardar entradas y respuestas del modelo para trazabilidad

---

## Resultado Final

- OpenWebUI accesible desde Internet via ALB
- Ollama corriendo y respondiendo prompts
- Archivos de usuarios procesados y almacenados en S3
- Búsqueda contextual con OpenSearch
- Arquitectura escalable y segura

---

¡Listo! FeedAI ahora está desplegado en AWS y listo para escalar.
