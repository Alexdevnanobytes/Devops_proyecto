<p align="center">
  <img src="https://img.icons8.com/color/96/000000/docker.png" alt="Docker"/>
  <img src="https://img.icons8.com/color/96/000000/kubernetes.png" alt="Kubernetes"/>
  <img src="https://img.icons8.com/color/96/000000/amazon-web-services.png" alt="AWS"/>
  <img src="https://img.icons8.com/color/96/000000/spring-logo.png" alt="Spring Boot"/>
  <img src="https://img.icons8.com/color/96/000000/react-native.png" alt="React"/>
  <h1 align="center">Evaluación Final Transversal (EFT) - Arquitectura DevOps & Cloud en EKS</h1>
</p>


## 📝 Resumen de la Arquitectura
Este repositorio contiene la solución técnica automatizada para el ciclo de integración y despliegue continuo (CI/CD) de la plataforma E-commerce "Tienda de Perritos". El ecosistema ha evolucionado a una **arquitectura real de microservicios** contenerizada, desplegada en AWS y orquestada en producción con **Amazon EKS (Elastic Kubernetes Service)**.

Se compone de:
1. **Frontend**: Interfaz de usuario (React/Vite).
2. **Backend Ventas**: Microservicio encargado del core de ventas (Java/Spring Boot).
3. **Backend Despachos**: Microservicio encargado de la logística (Java/Spring Boot).
4. **Base de Datos**: Motor relacional (MySQL 8.0) con persistencia (PVC).

---

## 🏗 1. Estrategia de Contenedorización
Los `Dockerfile` de los 3 microservicios (Frontend, Ventas, Despachos) cumplen estrictamente con las reglas Cloud Native:
- **Multi-stage Build**: Separamos la fase de construcción (`node`/`maven`) de la ejecución (`nginx`/`jre-alpine`), optimizando radicalmente el peso y la velocidad de transferencia.
- **Imágenes Minimalistas**: Utilizamos imágenes base tipo `alpine` para reducir la superficie de ataque.
- **Usuario Non-Root**: Los contenedores backend no se ejecutan como `root`. Emplean un usuario de sistema dedicado `appuser` previniendo brechas de seguridad (principio de mínimo privilegio).

## 🚀 2. Flujo CI/CD y Pipeline Inmutable (GitHub Actions)
Todo el flujo de despliegue opera automáticamente a través de la rama `deploy` mediante nuestro archivo `.github/workflows/eft-deploy.yml`:
1. **Build & Push**: Construye las 3 imágenes en paralelo y las empuja al registro privado de Amazon ECR, etiquetándolas con un tag único basado en el commit (`github.sha`).
2. **Renderizado de Manifiestos**: Usando utilidades GNU, reemplaza dinámicamente las etiquetas de imagen dentro de los manifiestos de Kubernetes (`k8s/*.yml`).
3. **Deploy a EKS**: Aplica la infraestructura sobre el clúster usando `kubectl`.
4. **Rollout Status**: El pipeline bloquea su finalización exitosa hasta verificar que los pods escalaron y están en estado *Running*, garantizando que no hay "falsos positivos" en la integración.

## 🌐 3. Orquestación, Nodos y Balanceo de Carga (AWS EKS)
El proyecto confía en Amazon EKS para el aprovisionamiento.
- **Estructura del Clúster y Nodos**: El clúster delega la ejecución de contenedores sobre *Worker Nodes* (instancias EC2 autoescalables integradas al control plane de EKS). Esto distribuye la carga eficientemente si un nodo falla.
- **Balanceo de Carga (LoadBalancer)**: El Frontend se expone al exterior mediante un servicio de Kubernetes tipo `LoadBalancer`. AWS aprovisiona automáticamente un *Classic/Application Load Balancer* que distribuye el tráfico HTTP entrante hacia los pods del frontend vivos. En cambio, los Backends y la Base de Datos usan servicios de tipo `ClusterIP`, permaneciendo inaccesibles y seguros desde internet.

## ⚙️ 4. Autoscaling (HPA) y Métricas
Para garantizar alta disponibilidad frente a fluctuaciones de tráfico, implementamos el **Horizontal Pod Autoscaler (HPA)**.
- **Justificación del HPA en el Backend**: El backend de ventas tiene configurado un límite de peticiones de CPU (`requests: cpu 250m`). El HPA monitorea esto, y está configurado para que, si el promedio del uso de CPU supera el **50%**, dispare el escalamiento horizontal. Crecerá dinámicamente desde **1 hasta 4 pods** para absorber el estrés y luego destruirá los pods extra cuando el tráfico baje, ahorrando costos.
- **Observabilidad (CloudWatch)**: Los logs de los despliegues de EKS y los nodos de trabajo son monitoreados mediante Amazon CloudWatch.

## 📊 Diagrama de Arquitectura
![Diagrama de Arquitectura EKS](https://via.placeholder.com/800x400.png?text=Reemplazar+con+Imagen+de+Draw.io)
*(Reemplazar con exportación JPG/PNG desde Draw.io)*

---

## 🛠 Instrucciones de Ejecución Local (Pruebas)
Para levantar la orquestación simulada localmente con Docker Compose:

1. Clona el repositorio y ubícate en la raíz.
2. Ejecuta la construcción multi-contenedor:
   ```bash
   docker-compose up -d --build
   ```
3. Explora la aplicación:
   - **Frontend**: http://localhost:8082
   - **Backend Ventas**: http://localhost:8080/swagger-ui.html
   - **Backend Despachos**: http://localhost:8081/swagger-ui.html
   - **Base de Datos**: Puerto 3306.

Para apagar el entorno limpiamente:
```bash
docker-compose down
```
