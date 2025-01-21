# Deploying-an-app-in-Kubernetes-cluster
Tarea asignada para desplegar una app sencilla en un clúster Kubernetes con Deployments y Services. 

Actividad Kubernetes: Desplegando una Aplicación con Despliegues y Servicios

**Paso 1: Preparar la aplicación**

En este paso, crearemos una aplicación web sencilla basada en Nginx y la empaquetaremos en una imagen Docker.  Usaremos un archivo `index.html` personalizado para mostrar un mensaje "Hola Mundo" desde Kubernetes.

**1.1 Crear el archivo `index.html`:**

Crea un archivo llamado `index.html` con el siguiente contenido:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hola Mundo Kubernetes</title>
</head>
<body>
    <h1>¡Hola desde Kubernetes!</h1>
</body>
</html>
```

Este archivo HTML mostrará el mensaje "Hola desde Kubernetes" cuando se acceda a la aplicación.

**1.2 Crear el Dockerfile:**

Crea un archivo llamado `Dockerfile` con el siguiente contenido:

```dockerfile
FROM nginx:alpine  # Usamos la imagen oficial de Nginx basada en Alpine Linux (más ligera)

COPY index.html /usr/share/nginx/html  # Copiamos nuestro archivo index.html al directorio raíz del servidor web Nginx
```

Este Dockerfile utiliza la imagen `nginx:alpine` como base. Alpine Linux es una distribución de Linux muy ligera, lo que resulta en imágenes Docker más pequeñas y eficientes. El comando `COPY` copia el archivo `index.html` al directorio correcto dentro de la imagen para que Nginx lo sirva.

**1.3 Construir la imagen Docker:**

Abre una terminal en el directorio donde se encuentran los archivos `index.html` y `Dockerfile`. Ejecuta el siguiente comando para construir la imagen Docker:

```bash
docker build -t tu-usuario/nginx-hola-mundo:1.0 .
```

* `docker build`: Comando para construir una imagen.
* `-t tu-usuario/nginx-hola-mundo:1.0`: Etiqueta la imagen con un nombre y una versión.  Reemplaza `tu-usuario` con tu nombre de usuario de Docker Hub (si lo vas a subir).
* `.`: Indica que el contexto de construcción es el directorio actual.

**1.4  Subir la imagen a un registro (opcional pero recomendado):**

Si estás usando un clúster Kubernetes en la nube o quieres compartir tu imagen, es recomendable subirla a un registro de contenedores como Docker Hub.

```bash
docker push tu-usuario/nginx-hola-mundo:1.0
```

Reemplaza `tu-usuario` con tu nombre de usuario de Docker Hub. Asegúrate de haber iniciado sesión en Docker Hub con `docker login`.

**1.5 Verificar la imagen (local):**

Puedes verificar que la imagen se ha creado localmente con:

```bash
docker images
```

Deberías ver tu imagen `tu-usuario/nginx-hola-mundo:1.0` en la lista.

---

### **Paso 2: Crear un Deployment**

### **2.1 Crear el archivo `deployment.yaml`**
Crea un archivo llamado `deployment.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1          # Versión de la API para Deployments
kind: Deployment             # Tipo de recurso (Deployment)
metadata:
  name: nginx-deployment     # Nombre del Deployment
  labels:
    app: nginx               # Etiqueta para identificar el Deployment
spec:
  replicas: 3                # Número de réplicas (pods) a mantener
  selector:
    matchLabels:
      app: nginx             # Selector que coincide con las etiquetas de los pods
  strategy:
    type: RollingUpdate      # Estrategia de actualización (sin downtime)
    rollingUpdate:
      maxUnavailable: 1      # Máximo de pods no disponibles durante la actualización
      maxSurge: 1            # Máximo de pods nuevos creados durante la actualización
  template:
    metadata:
      labels:
        app: nginx           # Etiqueta aplicada a los pods creados
    spec:
      containers:
      - name: nginx          # Nombre del contenedor
        image: tu-usuario/nginx-hola-mundo:1.0  # Imagen Docker a usar
        ports:
        - containerPort: 80  # Puerto expuesto por el contenedor
```

---

### **2.2 Explicación del YAML**

| Sección | Propósito | Ejemplo |
|---------|-----------|---------|
| **`apiVersion`** | Especifica la versión de la API de Kubernetes. Para Deployments, usa `apps/v1`. | `apps/v1` |
| **`kind`** | Define el tipo de recurso. Aquí es un `Deployment`. | `Deployment` |
| **`metadata`** | Proporciona metadatos como el nombre y las etiquetas del Deployment. | `name: nginx-deployment` |
| **`spec.replicas`** | Número de pods que el Deployment debe mantener en ejecución. | `replicas: 3` |
| **`spec.selector`** | Define cómo el Deployment encuentra los pods que gestiona. Debe coincidir con las etiquetas del pod. | `matchLabels: app: nginx` |
| **`spec.strategy`** | Configura cómo se actualizan los pods. `RollingUpdate` reemplaza pods gradualmente. | `type: RollingUpdate` |
| **`spec.template`** | Plantilla para crear nuevos pods. Incluye la configuración del contenedor. | `containers: - name: nginx` |
| **`spec.template.spec.containers`** | Define la imagen del contenedor y los puertos expuestos. | `image: tu-usuario/nginx-hola-mundo:1.0` |

---

### **2.3 Aplicar el Deployment**
1. **Ejecuta el comando para crear el Deployment:**
   ```bash
   kubectl apply -f deployment.yaml
   ```
   Salida esperada:
   ```
   deployment.apps/nginx-deployment created
   ```

2. **Verifica el estado del Deployment:**
   ```bash
   kubectl get deployments
   ```
   Salida esperada:
   ```
   NAME               READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-deployment   3/3     3            3           10s
   ```

3. **Verifica los pods creados:**
   ```bash
   kubectl get pods
   ```
   Salida esperada (3 pods en estado `Running`):
   ```
   NAME                                READY   STATUS    RESTARTS   AGE
   nginx-deployment-5c689d8bb6-abcde   1/1     Running   0          15s
   nginx-deployment-5c689d8bb6-fghij   1/1     Running   0          15s
   nginx-deployment-5c689d8bb6-klmno   1/1     Running   0          15s
   ```

---

### **2.4 Comandos clave para gestionar el Deployment**
| Comando | Propósito |
|---------|-----------|
| **`kubectl describe deployment nginx-deployment`** | Muestra detalles del Deployment (eventos, réplicas, etc.). |
| **`kubectl rollout status deployment nginx-deployment`** | Monitorea el progreso de una actualización. |
| **`kubectl logs <nombre-del-pod>`** | Muestra los logs de un pod específico. |
| **`kubectl scale deployment nginx-deployment --replicas=5`** | Escala el número de pods a 5. |

### **2.5 Errores comunes y solución**
| Error | Causa | Solución |
|-------|-------|----------|
| **`ImagePullBackOff`** | Kubernetes no puede descargar la imagen Docker. | Verifica que la imagen existe y la etiqueta es correcta. |
| **`CrashLoopBackOff`** | El contenedor falla al iniciarse. | Revisa los logs del pod con `kubectl logs <pod-name>`. |
| **`selector does not match template labels`** | Las etiquetas del pod no coinciden con el selector del Deployment. | Asegúrate de que `spec.selector.matchLabels` y `spec.template.metadata.labels` sean idénticos. |

---

### Paso 3: Crear un Service para exponer la aplicación

Usaremos un Service de tipo **NodePort**, que permite acceder a la aplicación desde fuera del clúster a través de un puerto específico en los nodos de Kubernetes. 

### **3.1 Crear el archivo `service.yaml`**

Crearemos un archivo llamado `service.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Service                # Tipo de recurso (Service)
metadata:
  name: nginx-service        # Nombre del Service
spec:
  type: NodePort             # Tipo de Service (NodePort expone la aplicación externamente)
  selector:
    app: nginx               # Selector que coincide con las etiquetas de los pods del Deployment
  ports:
    - protocol: TCP          # Protocolo de red (TCP o UDP)
      port: 80               # Puerto interno del Service
      targetPort: 80         # Puerto del contenedor al que se redirige el tráfico
      nodePort: 30007        # Puerto del nodo (rango 30000-32767)
```

---

### **3.2 Explicación detallada del YAML**

| Campo | Propósito | Ejemplo |
|-------|-----------|---------|
| **`apiVersion`** | Versión de la API para Services. | `v1` |
| **`kind`** | Tipo de recurso. Aquí es un `Service`. | `Service` |
| **`metadata.name`** | Nombre del Service. | `nginx-service` |
| **`spec.type`** | Tipo de Service. **NodePort** expone la aplicación en un puerto del nodo. | `NodePort` |
| **`spec.selector`** | Etiquetas para seleccionar los pods gestionados por el Deployment. Debe coincidir con las etiquetas del Deployment. | `app: nginx` |
| **`spec.ports`** | Configuración de los puertos. | `port: 80` (Service), `targetPort: 80` (contenedor) |
| **`nodePort`** | Puerto del nodo para acceso externo. | `30007` (rango 30000-32767) |

---

### **3.3 Aplicar el Service**

1. **Ejecuta el comando para crear el Service:**
   ```bash
   kubectl apply -f service.yaml
   ```
   Salida esperada:
   ```
   service/nginx-service created
   ```

2. **Verifica el estado del Service:**
   ```bash
   kubectl get services
   ```
   Salida esperada:
   ```
   NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
   nginx-service   NodePort   10.96.123.123   <none>        80:30007/TCP   10s
   ```

---

### **3.4 Acceder a la aplicación**

Dependiendo de tu entorno Kubernetes:

#### **En Minikube:**
```bash
minikube service nginx-service --url
```
Este comando abrirá automáticamente la aplicación en tu navegador.  
Ejemplo de salida:
```
http://192.168.49.2:30007
```

#### **En un clúster en la nube (AWS, GCP, Azure):**
1. Obtén la **IP externa** de un nodo del clúster:
   ```bash
   kubectl get nodes -o wide
   ```
   Ejemplo de salida:
   ```
   NAME       STATUS   EXTERNAL-IP
   node-01    Ready    34.123.45.67
   ```

2. Accede a la aplicación usando la IP del nodo y el `nodePort`:
   ```
   http://34.123.45.67:30007
   ```

---

### **3.5 Comandos clave para gestionar el Service**

| Comando | Propósito |
|---------|-----------|
| **`kubectl describe service nginx-service`** | Muestra detalles del Service (IP, puertos, eventos). |
| **`kubectl delete service nginx-service`** | Elimina el Service. |
| **`kubectl edit service nginx-service`** | Edita el Service en tiempo real. |

---

### **3.6 Errores comunes y solución**

| Error | Causa | Solución |
|-------|-------|----------|
| **`No endpoints available for service`** | El selector del Service no coincide con las etiquetas de los pods. | Verifica que `spec.selector` en el Service coincida con las etiquetas del Deployment. |
| **`Connection refused`** | Los pods no están escuchando en el `targetPort`. | Asegúrate de que el contenedor expone el puerto correcto (`containerPort` en el Deployment). |
| **`nodePort fuera de rango`** | El `nodePort` no está entre 30000-32767. | Usa un puerto dentro del rango permitido. |

---

### **3.7 Diagrama de flujo del tráfico**

```
Usuario Externo
       ↓
[Nodo Kubernetes:30007]  # Service (NodePort)
       ↓
[Pod 1] ←→ [Pod 2] ←→ [Pod 3]  # Réplicas del Deployment
```


---

### **Paso 4: Acceder a la aplicación y escalarla**

Profundicemos en las formas de acceder a la aplicación y verificar su funcionamiento.

**a) Usando Minikube:**
```bash
minikube service nginx-service --url
```
Este comando devolverá una URL como `http://192.168.49.2:30007` y abrirá automáticamente la aplicación en tu navegador. Verás el mensaje "¡Hola desde Kubernetes!".

**b) En un clúster en la nube:**
1. Obtén la IP externa del nodo y el puerto del Service:
   ```bash
   kubectl get service nginx-service -o wide
   ```
   Ejemplo de salida:
   ```
   NAME            TYPE       CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
   nginx-service   NodePort   10.96.123.123   34.123.45.67    80:30007/TCP   5m
   ```
2. Accede a la aplicación usando la IP externa y el `nodePort`:
   ```
   http://34.123.45.67:30007
   ```

**c) Verificar conectividad interna (dentro del clúster):**
Si quieres probar la comunicación entre pods, ejecuta:
```bash
kubectl exec <nombre-del-pod> -- curl http://nginx-service
```

Esto simulará una solicitud desde un pod hacia el Service.

---
#### **4.2 Escalar la aplicación**
El Deployment permite escalar horizontalmente el número de réplicas (pods) de la aplicación.

**a) Escalar manualmente:**
```bash
kubectl scale deployment nginx-deployment --replicas=5
```
Verifica el escalamiento:
```bash
kubectl get pods
```
Verás 5 pods en ejecución:
```
nginx-deployment-abcde   1/1     Running   0          10s
nginx-deployment-fghij   1/1     Running   0          10s
...
```

**b) Escalado automático (Horizontal Pod Autoscaler - HPA):**
Si tienes métricas configuradas, puedes usar:
```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=3 --max=10
```

Este comando ajustará automáticamente las réplicas según el uso de CPU.

---

#### **4.3 Monitorear el tráfico y los logs**
**a) Ver logs de un pod específico:**
```bash
kubectl logs <nombre-del-pod>
```

**b) Monitorear el tráfico en tiempo real:**
```bash
kubectl get pods -w  # Muestra cambios en los pods (creación/eliminación)
```

**c) Ver métricas de recursos:**
```bash
kubectl top pods  # Requiere instalación de Metrics Server
```

---

#### **4.4 Diagrama de arquitectura actual**
```
Usuario Externo
       │
       ▼
   [Service] (NodePort:30007)
       │
       ▼
   [Deployment]
       ├── Pod 1 (nginx)
       ├── Pod 2 (nginx)
       ├── Pod 3 (nginx)
       ├── Pod 4 (nginx)  # Después del escalamiento
       └── Pod 5 (nginx)
```

--- 

¡Perfecto! Vamos al **Paso 5: Actualizar la aplicación** para implementar una nueva versión sin interrumpir el servicio. Usaremos la estrategia de actualización `RollingUpdate` definida en el Deployment.

---

### **Paso 5: Actualizar la Aplicación**

#### **5.1 Modificar la imagen del contenedor**

1. **Actualiza la versión de la imagen en el `deployment.yaml`:**
   ```yaml
   containers:
   - name: nginx
     image: tu-usuario/nginx-hola-mundo:2.0  # Cambia de 1.0 a 2.0
   ```

2. **Aplica los cambios:**
   ```bash
   kubectl apply -f deployment.yaml
   ```

---

#### **5.2 Monitorear la actualización**

1. **Verifica el estado de la actualización:**
   ```bash
   kubectl rollout status deployment nginx-deployment
   ```
   Salida esperada:
   ```
   deployment "nginx-deployment" successfully rolled out
   ```

2. **Observa los pods en tiempo real:**
   ```bash
   kubectl get pods --watch
   ```
   Verás cómo los pods antiguos se eliminan gradualmente y se crean nuevos.

---

#### **5.3 Verificar la nueva versión**

1. **Accede a la aplicación:**
   ```bash
   minikube service nginx-service --url  # O usa la IP del nodo en la nube
   ```
   Asegúrate de que la nueva versión muestra los cambios esperados (ej: "¡Hola desde Kubernetes v2.0!").

---

#### **5.4 Revertir una actualización fallida**

Si la nueva versión tiene errores, puedes revertir al estado anterior:

```bash
kubectl rollout undo deployment nginx-deployment
```

---

### **Recursos Adicionales**
- [Documentación oficial de Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Guía de Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
