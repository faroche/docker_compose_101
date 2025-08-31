# Docker Compose: Orquestando Aplicaciones Multi-Contenedor
## ¿Qué es Docker Compose?

Docker Compose es una herramienta que permite **definir y ejecutar aplicaciones Docker multi-contenedor**. En lugar de manejar cada contenedor por separado, Compose te permite:

- Definir toda tu aplicación en un solo archivo YAML
- Iniciar todos los servicios con un solo comando
- Gestionar redes y volúmenes automáticamente
- Facilitar el desarrollo local y el despliegue

### Analogía
Piensa en Docker Compose como el **director de orquesta** de tu aplicación. Mientras que Docker maneja instrumentos individuales (contenedores), Compose coordina toda la sinfonía.

## Conceptos Fundamentales

### 1. **Servicios (Services)**
Cada contenedor en tu aplicación. Por ejemplo:
- Servicio web (frontend)
- Servicio API (backend)
- Servicio de base de datos

### 2. **Redes (Networks)**
Permite que los contenedores se comuniquen entre sí de forma segura.

### 3. **Volúmenes (Volumes)**
Almacenamiento persistente que sobrevive al reinicio de contenedores.

### 4. **Variables de Entorno**
Configuración que se pasa a los contenedores.

---

## Estructura del archivo docker-compose.yml

```yaml
version: '3.8'                    # Versión de Compose
services:                         # Define los contenedores
  nombre-servicio:
    image: imagen:tag             # O 'build: .' para construir
    ports:                        # Mapeo de puertos
      - "puerto-host:puerto-contenedor"
    environment:                  # Variables de entorno
      - VARIABLE=valor
    depends_on:                   # Dependencias
      - otro-servicio
    volumes:                      # Montaje de volúmenes
      - volumen:/ruta/contenedor

volumes:                          # Define volúmenes persistentes
  volumen:

networks:                         # Define redes personalizadas
  red-personalizada:
```

---

## Comandos Esenciales

| Comando | Descripción |
|---------|-------------|
| `docker-compose up` | Inicia todos los servicios |
| `docker-compose up -d` | Inicia en modo "detached" (segundo plano) |
| `docker-compose down` | Detiene y elimina contenedores |
| `docker-compose build` | Construye/reconstruye imágenes |
| `docker-compose logs` | Muestra logs de todos los servicios |
| `docker-compose logs [servicio]` | Logs de un servicio específico |
| `docker-compose exec [servicio] [comando]` | Ejecuta comando en contenedor |
| `docker-compose ps` | Lista contenedores activos |
| `docker-compose restart` | Reinicia servicios |

---

# Ejemplo 1: Aplicación Web Simple (WordPress + MySQL)

Este ejemplo muestra cómo crear un blog WordPress con base de datos MySQL.

## Archivo: docker-compose.yml

```yaml
version: '3.8'

services:
  # Servicio WordPress
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_password
    depends_on:
      - db
    volumes:
      - wordpress_data:/var/www/html

  # Servicio MySQL
  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wp_password
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  wordpress_data:
  db_data:
```

## Instrucciones de uso:

1. **Crear directorio del proyecto:**
   ```bash
   mkdir wordpress-app
   cd wordpress-app
   ```

2. **Crear el archivo docker-compose.yml** (copiar contenido anterior)

3. **Iniciar la aplicación:**
   ```bash
   docker compose up -d
   ```

4. **Acceder a WordPress:**
   - Abrir navegador en `http://localhost:8080`
   - Completar instalación de WordPress

5. **Ver logs:**
   ```bash
   docker compose logs wordpress
   docker compose logs db
   ```

6. **Detener la aplicación:**
   ```bash
   docker compose down
   ```

**¿Qué aprendemos aquí?**
- Comunicación entre contenedores (WordPress → MySQL)
- Persistencia de datos con volúmenes
- Variables de entorno para configuración
- Dependencias entre servicios

---

# Ejemplo 2: API REST con Node.js + MongoDB + Redis

Este ejemplo muestra una arquitectura más compleja para una API moderna.

## Estructura del proyecto:
```
api-project/
├── docker-compose.yml
├── app/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
└── .env
```

## Archivo: docker-compose.yml

```yaml
version: '3.8'

services:
  # API Node.js
  api:
    build: ./app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://mongo:27017/myapi
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongo
      - redis
    volumes:
      - ./app:/usr/src/app
      - /usr/src/app/node_modules
    command: npm run dev

  # Base de datos MongoDB
  mongo:
    image: mongo:5.0
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password123
    volumes:
      - mongo_data:/data/db

  # Cache Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # Interfaz web para MongoDB
  mongo-express:
    image: mongo-express:latest
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: password123
      ME_CONFIG_MONGODB_SERVER: mongo
    depends_on:
      - mongo

volumes:
  mongo_data:
  redis_data:
```

## Archivo: app/Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

## Archivo: app/package.json

```json
{
  "name": "api-example",
  "version": "1.0.0",
  "description": "API example with Docker Compose",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.0.0",
    "redis": "^4.6.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.0"
  }
}
```

## Archivo: app/server.js

```javascript
const express = require('express');
const mongoose = require('mongoose');
const redis = require('redis');

const app = express();
const PORT = process.env.PORT || 3000;

// Conectar a MongoDB
mongoose.connect(process.env.MONGODB_URI)
  .then(() => console.log('Conectado a MongoDB'))
  .catch(err => console.error('Error conectando a MongoDB:', err));

// Conectar a Redis
const client = redis.createClient({
  url: process.env.REDIS_URL
});
client.connect().then(() => console.log('Conectado a Redis'));

app.use(express.json());

// Ruta simple
app.get('/', (req, res) => {
  res.json({ 
    message: 'API funcionando!',
    timestamp: new Date().toISOString()
  });
});

// Ruta con cache Redis
app.get('/api/cached-data', async (req, res) => {
  try {
    const cached = await client.get('data');
    if (cached) {
      return res.json({ 
        data: JSON.parse(cached), 
        source: 'cache' 
      });
    }
    
    const data = { value: Math.random(), timestamp: Date.now() };
    await client.setEx('data', 60, JSON.stringify(data)); // Cache por 60 segundos
    
    res.json({ data, source: 'database' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(PORT, () => {
  console.log(`Servidor corriendo en puerto ${PORT}`);
});
```

## Instrucciones de uso:

1. **Crear estructura del proyecto y archivos**

2. **Iniciar todos los servicios:**
   ```bash
   docker-compose up -d --build
   ```

3. **Verificar que funciona:**
   - API: `http://localhost:3000`
   - MongoDB Admin: `http://localhost:8081`
   - Probar cache: `http://localhost:3000/api/cached-data`

4. **Ver logs en tiempo real:**
   ```bash
   docker-compose logs -f api
   ```

**¿Qué aprendemos aquí?**
- Construcción de imágenes personalizadas
- Desarrollo con hot-reload
- Integración de múltiples tecnologías
- Uso de caché para optimización

---

# Ejemplo 3: Stack Completo de Monitoreo (ELK + Aplicación)

Este ejemplo muestra cómo implementar un sistema de monitoreo completo.

## Archivo: docker-compose.yml

```yaml
version: '3.8'

services:
  # Aplicación web de ejemplo
  web-app:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # Elasticsearch para almacenar logs
  elasticsearch:
    image: elasticsearch:8.8.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data

  # Logstash para procesar logs
  logstash:
    image: logstash:8.8.0
    volumes:
      - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  # Kibana para visualizar datos
  kibana:
    image: kibana:8.8.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  # Filebeat para enviar logs
  filebeat:
    image: elastic/filebeat:8.8.0
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - logstash
    command: filebeat -e -strict.perms=false

  # Prometheus para métricas
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus

  # Grafana para dashboards
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
      - ./grafana/datasources:/etc/grafana/provisioning/datasources:ro

volumes:
  es_data:
  prometheus_data:
  grafana_data:
```

## Archivos de configuración necesarios:

### filebeat/filebeat.yml
```yaml
filebeat.inputs:
- type: container
  paths:
    - '/var/lib/docker/containers/*/*.log'

processors:
- add_docker_metadata:
    host: "unix:///var/run/docker.sock"

output.logstash:
  hosts: ["logstash:5044"]

logging.level: info
```

### logstash/logstash.conf
```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  if [container][name] {
    mutate {
      add_field => { "service_name" => "%{[container][name]}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "docker-logs-%{+YYYY.MM.dd}"
  }
}
```

### prometheus/prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

### html/index.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>Mi Aplicación Monitoreada</title>
</head>
<body>
    <h1>¡Aplicación funcionando!</h1>
    <p>Esta aplicación está siendo monitoreada por el stack ELK y Prometheus/Grafana.</p>
    <script>
        // Generar algunos logs
        setInterval(() => {
            console.log('Heartbeat: ' + new Date().toISOString());
            fetch('/api/test').catch(e => console.error('API Error:', e));
        }, 30000);
    </script>
</body>
</html>
```

## Instrucciones de uso:

1. **Crear estructura de archivos y configuraciones**

2. **Iniciar todo el stack:**
   ```bash
   docker-compose up -d
   ```

3. **Acceder a las interfaces:**
   - Aplicación web: `http://localhost:8080`
   - Kibana (logs): `http://localhost:5601`
   - Prometheus: `http://localhost:9090`
   - Grafana: `http://localhost:3000` (admin/admin123)

4. **Configurar Kibana:**
   - Crear index pattern: `docker-logs-*`
   - Explorar logs en Discover

**¿Qué aprendemos aquí?**
- Arquitectura de microservicios compleja
- Monitoreo y logging centralizado
- Configuración avanzada con múltiples archivos
- Stack de observabilidad moderno

---

# Mejores Prácticas

## 1. **Organización de Archivos**
```
proyecto/
├── docker-compose.yml
├── docker-compose.prod.yml
├── .env
├── services/
│   ├── web/
│   └── api/
└── configs/
    ├── nginx/
    └── database/
```

## 2. **Variables de Entorno**
```yaml
# Usar archivo .env
services:
  db:
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_USER=${DB_USER}
```

## 3. **Múltiples Entornos**
```bash
# Desarrollo
docker-compose up

# Producción
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

## 4. **Healthchecks**
```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## 5. **Límites de Recursos**
```yaml
services:
  web:
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
```

---

# Actividades Prácticas

## Ejercicio 1: Modificar WordPress
- Cambiar el puerto de WordPress a 8090
- Agregar un servicio phpMyAdmin
- Configurar red personalizada

## Ejercicio 2: Extender la API
- Agregar servicio de email (MailHog)
- Implementar balanceador de carga (nginx)
- Agregar base de datos de pruebas

## Ejercicio 3: Debugging
- Usar `docker-compose logs` para encontrar errores
- Conectarse a contenedores con `exec`
- Modificar configuraciones sin recrear contenedores

---

# Recursos Adicionales

## Comandos Útiles de Troubleshooting
```bash
# Ver estado de servicios
docker-compose ps

# Seguir logs en tiempo real
docker-compose logs -f [servicio]

# Reiniciar un servicio específico
docker-compose restart [servicio]

# Acceder a un contenedor
docker-compose exec [servicio] bash

# Ver redes creadas
docker network ls

# Inspeccionar red
docker network inspect [nombre-red]
```

## Referencias
- [Documentación oficial de Docker Compose](https://docs.docker.com/compose/)
- [Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Awesome Docker Compose](https://github.com/docker/awesome-compose)

---

# Conclusiones

Docker Compose es fundamental para:
- **Desarrollo local**: Replicar entornos de producción
- **Microservicios**: Orquestar aplicaciones complejas
- **DevOps**: Automatizar despliegues
- **Aprendizaje**: Experimentar con diferentes tecnologías

