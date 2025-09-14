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
#version: '3.8'                    # Versión de Compose
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
| `docker compose up` | Inicia todos los servicios |
| `docker compose up -d` | Inicia en modo "detached" (segundo plano) |
| `docker compose down` | Detiene y elimina contenedores |
| `docker compose build` | Construye/reconstruye imágenes |
| `docker compose logs` | Muestra logs de todos los servicios |
| `docker compose logs [servicio]` | Logs de un servicio específico |
| `docker compose exec [servicio] [comando]` | Ejecuta comando en contenedor |
| `docker compose ps` | Lista contenedores activos |
| `docker compose restart` | Reinicia servicios |

---

# Ejemplo 1: Aplicación Web Simple (WordPress + MySQL)

Este ejemplo muestra cómo crear un blog WordPress con base de datos MySQL.

## Archivo: docker-compose.yml

```yaml
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
   docker compose up -d --build
   ```

3. **Verificar que funciona:**
   - API: `http://localhost:3000`
   - MongoDB Admin: `http://localhost:8081`
   - Probar cache: `http://localhost:3000/api/cached-data`

4. **Ver logs en tiempo real:**
   ```bash
   docker compose logs -f api
   ```

**¿Qué aprendemos aquí?**
- Construcción de imágenes personalizadas
- Desarrollo con hot-reload
- Integración de múltiples tecnologías
- Uso de caché para optimización

---

# Stack Completo de Monitoreo - Docker Compose
## ELK Stack + Prometheus + Grafana + Aplicación Web

### 🔧 Solución al Error de Montaje

El error que experimentas es común cuando Docker intenta montar un archivo que no existe. Vamos a crear la estructura completa paso a paso.

---

## Estructura de Directorios Requerida

Primero, crea esta estructura exacta de directorios:

```
monitoreo/
├── docker-compose.yml
├── html/
│   └── index.html
├── nginx/
│   └── nginx.conf
├── filebeat/
│   └── filebeat.yml
├── logstash/
│   └── logstash.conf
├── prometheus/
│   └── prometheus.yml
└── grafana/
    ├── dashboards/
    │   └── dashboard.yml
    └── datasources/
        └── datasource.yml
```

---

## Paso a Paso: Creación del Proyecto

### 1. Crear el directorio principal y navegar a él
```bash
mkdir monitoreo
cd monitoreo
```

### 2. Crear todos los subdirectorios necesarios
```bash
mkdir -p html nginx filebeat logstash prometheus grafana/dashboards grafana/datasources
```

### 3. Crear el archivo docker-compose.yml
```bash
touch docker-compose.yml
```

---

## Archivos de Configuración

### docker-compose.yml
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
      - cluster.name=docker-cluster
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Logstash para procesar logs
  logstash:
    image: logstash:8.8.0
    volumes:
      - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5044:5044"
    environment:
      - "LS_JAVA_OPTS=-Xmx256m -Xms256m"
    depends_on:
      elasticsearch:
        condition: service_healthy

  # Kibana para visualizar datos
  kibana:
    image: kibana:8.8.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=""
    depends_on:
      elasticsearch:
        condition: service_healthy

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
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'

  # Grafana para dashboards
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
      - ./grafana/datasources:/etc/grafana/provisioning/datasources:ro

volumes:
  es_data:
  prometheus_data:
  grafana_data:
```

---

## Archivos de Configuración Individuales

### html/index.html
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Aplicación Monitoreada</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .container {
            background: rgba(255,255,255,0.1);
            padding: 30px;
            border-radius: 15px;
            backdrop-filter: blur(10px);
        }
        .status {
            background: #4CAF50;
            padding: 10px;
            border-radius: 5px;
            margin: 20px 0;
            text-align: center;
        }
        .links {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-top: 30px;
        }
        .link-card {
            background: rgba(255,255,255,0.2);
            padding: 20px;
            border-radius: 10px;
            text-decoration: none;
            color: white;
            transition: transform 0.3s;
        }
        .link-card:hover {
            transform: translateY(-5px);
            background: rgba(255,255,255,0.3);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Aplicación Monitoreada</h1>
        <div class="status">✅ Sistema Funcionando Correctamente</div>
        <p>Esta aplicación está siendo monitoreada por un stack completo de observabilidad:</p>
        
        <ul>
            <li><strong>ELK Stack</strong> - Para logs centralizados</li>
            <li><strong>Prometheus</strong> - Para métricas del sistema</li>
            <li><strong>Grafana</strong> - Para dashboards visuales</li>
        </ul>

        <div class="links">
            <a href="http://localhost:5601" class="link-card" target="_blank">
                <h3>📊 Kibana</h3>
                <p>Análisis y búsqueda de logs</p>
            </a>
            <a href="http://localhost:9090" class="link-card" target="_blank">
                <h3>📈 Prometheus</h3>
                <p>Métricas del sistema</p>
            </a>
            <a href="http://localhost:3000" class="link-card" target="_blank">
                <h3>📋 Grafana</h3>
                <p>Dashboards y alertas</p>
                <small>admin / admin123</small>
            </a>
        </div>
    </div>

    <script>
        // Generar logs para demostración
        console.log('🚀 Aplicación iniciada:', new Date().toISOString());
        
        let logCounter = 0;
        setInterval(() => {
            logCounter++;
            console.log(`💓 Heartbeat #${logCounter}:`, new Date().toISOString());
            
            // Simular diferentes tipos de logs
            if (logCounter % 5 === 0) {
                console.warn('⚠️  Advertencia simulada - Cache miss');
            }
            if (logCounter % 10 === 0) {
                console.info('ℹ️  Info - Cleanup completado');
            }
            if (logCounter % 15 === 0) {
                console.error('❌ Error simulado - Timeout de conexión');
            }
        }, 10000); // Cada 10 segundos

        // Simular actividad de usuario
        document.addEventListener('click', (e) => {
            console.log('👆 Click detectado en:', e.target.tagName);
        });
    </script>
</body>
</html>
```

### nginx/nginx.conf
```nginx
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Formato de log personalizado
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    # Logs de acceso
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        # Endpoint de health check
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }

        # API simulada para generar logs
        location /api/test {
            return 200 '{"status": "ok", "timestamp": "$time_iso8601"}';
            add_header Content-Type application/json;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```

### filebeat/filebeat.yml
```yaml
filebeat.inputs:
- type: container
  paths:
    - '/var/lib/docker/containers/*/*.log'

processors:
- add_docker_metadata:
    host: "unix:///var/run/docker.sock"

- decode_json_fields:
    fields: ["message"]
    target: ""
    overwrite_keys: true

output.logstash:
  hosts: ["logstash:5044"]

logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644

# Configuración adicional para debugging
logging.selectors: ["*"]
```

### logstash/logstash.conf
```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  # Procesar logs de Docker
  if [container][name] {
    mutate {
      add_field => { "service_name" => "%{[container][name]}" }
    }
  }

  # Parsear logs JSON si existen
  if [message] =~ /^\{.*\}$/ {
    json {
      source => "message"
    }
  }

  # Agregar timestamp procesado
  mutate {
    add_field => { "processed_at" => "%{[@timestamp]}" }
  }

  # Detectar nivel de log por contenido
  if [message] =~ /(?i)error/ {
    mutate {
      add_field => { "log_level" => "ERROR" }
    }
  } else if [message] =~ /(?i)warn/ {
    mutate {
      add_field => { "log_level" => "WARN" }
    }
  } else if [message] =~ /(?i)info/ {
    mutate {
      add_field => { "log_level" => "INFO" }
    }
  } else {
    mutate {
      add_field => { "log_level" => "DEBUG" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "docker-logs-%{+YYYY.MM.dd}"
  }

  # Para debugging, también mostrar en stdout
  stdout {
    codec => rubydebug
  }
}
```

### prometheus/prometheus.yml
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['elasticsearch:9200']
    scrape_interval: 30s
    metrics_path: /_prometheus/metrics

  - job_name: 'nginx'
    static_configs:
      - targets: ['web-app:80']
    scrape_interval: 10s
    metrics_path: /metrics

# Configuración de alertas (opcional)
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093
```

### grafana/datasources/datasource.yml
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true

  - name: Elasticsearch
    type: elasticsearch
    access: proxy
    url: http://elasticsearch:9200
    database: "docker-logs-*"
    interval: Daily
    timeField: "@timestamp"
    editable: true
```

### grafana/dashboards/dashboard.yml
```yaml
apiVersion: 1

providers:
  - name: 'Docker Monitoring'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

---

## Instrucciones de Instalación

### 1. Verificar la estructura
Ejecuta este comando para verificar que todos los archivos estén presentes:

```bash
find . -type f -name "*.yml" -o -name "*.conf" -o -name "*.html" | sort
```

Deberías ver:
```
./docker-compose.yml
./filebeat/filebeat.yml
./grafana/dashboards/dashboard.yml
./grafana/datasources/datasource.yml
./html/index.html
./logstash/logstash.conf
./nginx/nginx.conf
./prometheus/prometheus.yml
```

### 2. Verificar permisos
```bash
chmod 644 nginx/nginx.conf
chmod 644 filebeat/filebeat.yml
chmod 644 logstash/logstash.conf
chmod 644 prometheus/prometheus.yml
```

### 3. Iniciar los servicios
```bash
# Iniciar en modo detached
docker-compose up -d

# Ver logs en tiempo real (opcional)
docker-compose logs -f
```

### 4. Verificar que los servicios estén funcionando
```bash
# Verificar estado
docker-compose ps

# Verificar logs específicos si hay errores
docker-compose logs elasticsearch
docker-compose logs kibana
docker-compose logs grafana
```

---

## Acceso a las Aplicaciones

Una vez que todos los contenedores estén funcionando:

| Servicio | URL | Credenciales |
|----------|-----|--------------|
| **Aplicación Web** | http://localhost:8080 | - |
| **Kibana** | http://localhost:5601 | - |
| **Prometheus** | http://localhost:9090 | - |
| **Grafana** | http://localhost:3000 | admin / admin123 |
| **Elasticsearch** | http://localhost:9200 | - |

---

## Configuración Post-Instalación

### En Kibana:
1. Ir a "Stack Management" → "Index Patterns"
2. Crear un nuevo index pattern: `docker-logs-*`
3. Seleccionar `@timestamp` como time field
4. Ir a "Discover" para ver los logs

### En Grafana:
1. Las fuentes de datos ya están configuradas automáticamente
2. Puedes crear dashboards personalizados
3. Explorar métricas de Prometheus

---

## Solución de Problemas Comunes

### Error: "file not found"
```bash
# Verificar que todos los archivos existen
ls -la nginx/nginx.conf
ls -la filebeat/filebeat.yml
ls -la logstash/logstash.conf
```

### Error: "permission denied"
```bash
# Corregir permisos
sudo chown -R $USER:$USER .
chmod -R 644 .
```

### Elasticsearch no inicia
```bash
# Verificar límites de memoria virtual
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Servicios no se comunican
```bash
# Verificar red de Docker
docker network ls
docker network inspect monitoreo_default
```

---

## Comandos Útiles para Monitoreo

```bash
# Ver logs en tiempo real de un servicio específico
docker-compose logs -f elasticsearch

# Reiniciar un servicio específico
docker-compose restart kibana

# Ver recursos utilizados
docker stats

# Limpiar volúmenes (CUIDADO: elimina datos)
docker-compose down -v

# Ver redes creadas
docker network ls

# Acceder a un contenedor
docker-compose exec elasticsearch bash
```

---

## Próximos Pasos

1. **Personalizar dashboards** en Grafana
2. **Configurar alertas** en Prometheus
3. **Agregar más aplicaciones** al monitoreo
4. **Implementar alertas** por email/Slack
5. **Optimizar índices** en Elasticsearch

Este stack completo te proporciona una base sólida para monitorear aplicaciones en contenedores de forma profesional.

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

