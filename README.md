# **Actividad 27: Migrando a Microservicios  Docker, Kubernetes y CI/CD**

## **A. Docker y Docker Compose**


### 1. **Arquitectura de contenedores**

**Explica como un contenedor encapsula la aplicacion y sus dependencias. ¿Que ventajas ofrece frente a una VM en terminos de arranque, consumo de recursos y portabilidad?**

cada contenedor encapsula la app con sus dependencias ya sea bibliotecas, runtime , configuraciones y una porcion minima del sistema operativo que es el kernel compartido con el host tambien usa tecnologias como namespaces para aislar procesos, redes y sistemas de archivos  
En el proyecto los Dockerfile de service-user y service-order incluyen Python 3.10, fastapi, uvicorn, pydantic y el codigo de la aplicacion, garantizando que el entorno sea identico en cualquier maquina.

**Ventajas frente a una VM:**

- **arranque**: los contenedores inician en segundos ya que al no cargar un SO completo  en comparacion con VM
- **consumo de recursos**: los contenedores comparten el kernel del host lo que  usa menos CPU y memoria 
- **portabilidad**: imagenes como user-service:latest son autocontenidas ejecutandose igual en dev, staging o prod, mientras que las VMs dependen de hipervisores y configuraciones especificas


**Pasos de docker build -t user-service .:**

```Docker
# User-Service Dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

- `FROM python:3.10-slim`: descarga imagen base es importante para establecer un entorno ligero y consistente
- `WORKDIR /app`: define  directorio de trabajo ayuda a tener rutas predecibles
- `COPY requirements.txt .`: copia dependencias es importante para instalar solo lo necesario
- `RUN pip install --no-cache-dir -r requirements.txt`: instala dependencias sin almacenar cache reduce tamaño de imagen
- `COPY app.py .`: copia codigo de la app  importante para incluir la logica del servicio
- `EXPOSE 8000`: documenta el puerto
- `CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]`: define  comando de arranque importante para ejecutar la app correctamente

## **Ejecucion del proyecto**

**1. Prerrequisitos**

* Docker instalado
* Minikube (`>= v1.30`)
* `kubectl` en tu `$PATH`
* (Opcional) Bash / Git Bash

**2. Iniciar Minikube y preparar Docker**

```bash
# Arranca Minikube
minikube start --driver=docker

# Apunta Docker al daemon de Minikube
# Forma recomendada:
eval "$(minikube -p minikube docker-env --shell bash)"
```

> **Observacion:**
>
> * Este `eval` exporta automaticamente las variables (`DOCKER_HOST`, `DOCKER_CERT_PATH`, etc.) y afecta solo a la sesion actual.
> * Si prefieres no usar `eval`, tendrias que:
>
>   ```bash
>   export DOCKER_TLS_VERIFY="1"
>   export DOCKER_HOST="tcp://192.168.49.2:2376"
>   export DOCKER_CERT_PATH="/home/kapum/.minikube/certs"
>   export MINIKUBE_ACTIVE_DOCKERD="minikube"
>   ```
>
>   Pero es mas propenso a errores.

**3. Construccion de imagenes**

```bash
# Desde la raiz del proyecto
docker build -t user-service:latest ./service-user
docker build -t order-service:latest ./service-order
```

**4. Despliegue en Kubernetes**

```bash
kubectl apply -f k8s/user-deployment.yaml
kubectl apply -f k8s/order-deployment.yaml
```

**5. Verificacion**

```bash
kubectl get pods
kubectl get svc

# Exponer los servicios localmente (abre navegador)
minikube service user-service
minikube service order-service
```

### Scripts de ayuda

Para automatizar los pasos de arranque, construccion y despliegue:

```bash
# Script principal de Minikube
chmod +x minikube-setup.sh
./minikube-setup.sh

# O usando el helper de deployment
chmod +x scripts/deploy.sh
./scripts/deploy.sh
```


