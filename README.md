<p align="center">
  <img src="https://img.icons8.com/color/96/000000/docker.png" alt="Docker"/>
  <img src="https://img.icons8.com/color/96/000000/git.png" alt="Git"/>
  <img src="https://img.icons8.com/color/96/000000/amazon-web-services.png" alt="AWS"/>
  <img src="https://img.icons8.com/color/96/000000/spring-logo.png" alt="Spring Boot"/>
  <img src="https://img.icons8.com/color/96/000000/react-native.png" alt="React"/>
  <h1 align="center">Innovatech Chile - Arquitectura DevOps & Cloud</h1>
</p>

## 📝 Resumen del Encargo
Este repositorio contiene la solución técnica al requerimiento de la "Evaluación Parcial 2" para la empresa Innovatech Chile. El proyecto demuestra la **contenedorización, orquestación y despliegue automatizado** de una plataforma de E-commerce (Tienda de Perritos) utilizando prácticas DevOps modernas, Amazon Web Services (VPC, EC2, SSM, ECR) y flujos CI/CD mediante GitHub Actions.

## 🏗 1. Estrategia de Contenedorización *(IE1, IE6)*
Cada servicio dentro de este ecosistema ha sido aislado mediante contenedores propios:

- **Frontend (React/Vite)**
- **Backend Ventas (Java/Spring Boot)**
- **Backend Despachos (Java/Spring Boot)**
- **Base de Datos (MySQL)**

### Buenas Prácticas en Dockerfile
Los `Dockerfile` construidos cumplen con las reglas estándar de la industria Cloud:
- **Multi-stage Build**: Separamos la fase de construcción pesada (`node`/`maven`) de la ejecución ligera (`nginx`/`jre`), optimizando drásticamente el peso final de la imagen a desplegar.
- **Usuario Non-Root**: Respetamos el principio del menor privilegio. Los comandos se ejecutan mediante usuarios de tipo `appuser` o `nginx`, previniendo vulnerabilidades y *container escapes*.
- **Optimización de Capas**: Ubicación inteligente de dependencias `.json` o `.xml` previas a la copia del código fuente, permitiendo a Docker usar su memoria caché para ahorrar tiempo de compilación.

## 💾 2. Persistencia de Datos (Volúmenes) *(IE2)*
La información de nuestros despachos y usuarios está protegida en la instancia contra cualquier fallo, usando volúmenes integrados a la infraestructura mediante `docker-compose.yml`:
- **Named Volume (`dbdata`)**: Se mapeó la ruta de MySQL nativa a un volumen en EC2. 
- **Justificación**: Se optó por un *Named Volume* administrado directamente por el Docker Daemon en lugar de un *Bind Mount* manual. Esto asegura facilidad de respaldo y total continuidad operativa; si nuestro contenedor MySQL se destruye, los datos persistirán intocables listos para acoplarse al arranque del siguiente motor.

## 🚀 3. Flujo CI/CD y Entrega Continua *(IE3, IE7, IE8)*
Orquestamos un Workflow inmutable en **GitHub Actions**:
1. **Trigger**: Se evalúan los *commits* hacia la rama `deploy`.
2. **Build**: Un *runner* genera temporalmente la nueva imagen con los últimos cambios del equipo, según el directorio operado (*paths*).
3. **Push a ECR**: La imagen se almacena resguardada utilizando *Amazon ECR* como registro privado para evitar la exposición externa.
4. **Deploy Serverless**: Implementando **AWS SSM**, GitHub ordena a la instancia privada EC2 detener lo viejo y traccionar el contenedor refrescado (*Zero-downtime* sin abrir puertos SSH/22). 
- **Manejo de Secrets**: Absolutamente ninguna contraseña, IP remota, token ECR o IAM Key está expuesta o harcodeada. Toda inyección transita cifrada usando Secretos de GitHub.

## 🌐 4. Infraestructura EC2 y Ruteo de Datos *(IE4, IE9)*
La aplicación final se despliega resguardada bajo la topología Virtual Private Cloud (VPC):
- **Frontend** opera nativamente desde su EC2 Pública.
- **Backend APIs** y **Base de Datos** yacen confinados en sus instancias EC2 en Subredes Privadas.
- **Restricción y Acceso**: El Security Group (SG) de los Backends le entrega permiso exclusivo por puerto 8080/8081 únicamente al grupo originado por la EC2 Web Frontend, frenando cualquier intrusión originada directamente desde el internet público.

---

## 🛠 Instrucciones de Ejecución Local (Cómo usar y probar)
Cualquier desarrollador del equipo u observador técnico puede simular este ecosistema Cloud utilizando su propio computador. 

### Prerrequisitos
1. Tener instalado [Docker Desktop](https://www.docker.com/products/docker-desktop) en su computador (o cualquier Demonio Docker).
2. Tener instalado [Git](https://git-scm.com/).

### Instalación en 3 Pasos 

**1. Clonar el repositorio y acceder a él**
```bash
git clone https://github.com/Alexdevnanobytes/Devops_proyecto.git
cd Devops_proyecto
```

**2. Asignar Variables de Entorno Locales**
El cluster orquestado (`docker-compose.yml`) está programado simulando seguridad empresarial y exige Variables de Entorno. Córrelas en tu terminal de comandos *(Si estás en Windows usa `set` en vez de `export`):*
```bash
export DB_ENDPOINT=db
export DB_PORT=3306
export DB_NAME=tienda_db
export DB_USERNAME=root
export DB_PASSWORD=root
```

**3. Levantar la Orquestación Global**
Procede a empaquetar, descargar e iniciar simultáneamente los cuatro micro-contenedores con:
```bash
docker-compose up -d --build
```
> *Nota: La etapa inicial con `--build` descargará Java, Maven y librerías Node, lo que podría tomar unos minutos dependiendo de la conexión a internet.*

### Verificar el Funcionamiento Local
Una vez listados en consola con el estatus de "*Started*", puedes acceder interactuando con ellos desde tu navegador local:

- 🎮 **Interfaz del Usuario (Frontend)**: [http://localhost:80](http://localhost:80)
- 🛒 **Backend Ventas (Swagger UI)**: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html) 
- 🚚 **Backend Despachos (Swagger UI)**: [http://localhost:8081/swagger-ui.html](http://localhost:8081/swagger-ui.html) 

### Detener los servicios
Cuando finalices tus pruebas, asegúrate de apagar y remover los contenedores huérfanos correctamente con:
```bash
docker-compose down
```
*(Y puedes agregar `docker-compose down -v` si también deseas destruir y limpiar los datos del volumen persistente `dbdata` guardado localmente)*.
