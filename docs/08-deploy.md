# Despliegue

Guía para desplegar el proyecto Wiki-Collection en producción usando **MongoDB Atlas**, **Render** (backend) y **Vercel** (frontend). Todas las opciones tienen planes gratuitos que cubren un proyecto personal/prototipo.

---

## Arquitectura de Despliegue

```
┌──────────────┐     ┌─────────────────┐     ┌────────────────┐
│   Frontend   │     │    Backend      │     │    MongoDB     │
│   (Vercel)   │────▶│    (Render)     │────▶│  (Atlas Free)  │
│              │◀────│                 │◀────│                │
└──────────────┘     └─────────────────┘     └────────────────┘
  React + Vite        Spring Boot 4.1.1        MongoDB 7+
  SPA estático        Java 25 + JAR            512 MB gratis
```

---

## 1. MongoDB Atlas (Base de Datos)

### Registro y Creación del Cluster

1. Acceder a [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register)
2. Crear cuenta (requiere email + tarjeta de crédito, pero el free tier no cobra)
3. En el dashboard: **Build a Database** → **Shared** (Free)
4. Elegir proveedor y región más cercana (ej. AWS irlandesa para Europa)
5. Nombrar el cluster: `wiki-collection`
6. Esperar a que el cluster esté listo (unos minutos)

### Configuración de Usuario y Red

1. **Database Access** → Crear usuario administrador:
   - Username: `wikiadmin`
   - Password: generar una contraseña fuerte (guárdala)
   - Privileges: **Atlas Admin** (o **Read/Write** para producción)

2. **Network Access** → Add IP Address:
   - **0.0.0.0/0** — permite conexión desde cualquier IP (solo para desarrollo/pruebas)
   - O añadir IPs específicas del servidor Render (si se conocen)
   - **Recomendado para producción**: IP del servidor Render + 0.0.0.0/0 solo si el servidor cambia de IP frecuentemente

### Obtener la URI de Conexión

1. En el dashboard del cluster → **Connect** → **Connect your application**
2. Seleccionar driver: **Java** y versión: **4.10 o superior**
3. Copiar la URI:
   ```
   mongodb+srv://wikiadmin:<password>@wiki-collection.xxxxx.mongodb.net/wiki_collection?retryWrites=true&w=majority
   ```
4. Reemplazar `<password>` por la contraseña real

### Base de Datos

- El nombre de la base de datos en la URI es `wiki_collection` (o el que se prefiera)
- Atlas crea la base de datos automáticamente al primer uso
- **Límites del free tier**:
  - 512 MB de almacenamiento
  - 3 conexiones concurrentes (suficientes para un backend + algunos clientes)
  - RAM compartida
  - Sin backups automatizados (solo snapshots manuales)

---

## 2. Backend en Render (Spring Boot + Java 25)

### Requisitos Previos

- Tener el proyecto backend clonado: `backend-collection`
- Tener un cuenta en [Render](https://render.com)
- Tener la URI de MongoDB Atlas

### Paso 1: Preparar el Backend para Despliegue

#### application.properties para producción

El backend ya tiene soporte para perfiles Spring. Crear `application-prod.properties`:

```properties
# server.port=8080 (Render asigna el puerto mediante PORT env var)

# MongoDB - se inyecta vía variable de entorno
spring.mongodb.uri=${MONGODB_URI}

# Server
server.address=0.0.0.0

# Activar perfil prod
spring.profiles.active=prod

# Activar CORS para el frontend en Vercel
app.cors.allowed-origins=https://tu-proyecto.vercel.app

# Swagger/OpenAPI solo en desarrollo
springdoc.api-docs.enabled=false
springdoc.swagger-ui.enabled=false

# Logging
logging.level.org.springframework.security=WARN
logging.level.com.wikicollection=INFO
```

#### Variables de Entorno Requeridas

Render permite configurar variables de entorno en la dashboard. Estas son las necesarias:

| Variable | Valor | Descripción |
|----------|-------|-------------|
| `MONGODB_URI` | `mongodb+srv://wikiadmin:XXXXX@cluster0.xxxxx.mongodb.net/wiki_collection?retryWrites=true&w=majority` | URI de Atlas |
| `SERVER_PORT` | `8080` (o usar el puerto que Render asigna) | Puerto del servidor |
| `JWT_SECRET` | Cadena aleatoria de 256 bits (base64) | Secreto para JWT |
| `JWT_ACCESS_TOKEN_EXPIRATION` | `900000` (15 min en ms) | Expiración access token |
| `JWT_REFRESH_TOKEN_EXPIRATION` | `604800000` (7 días en ms) | Expiración refresh token |
| `ADMIN_USERNAME` | `admin` | Usuario admin por defecto |
| `ADMIN_PASSWORD` | Contraseña admin | Password admin por defecto |
| `ADMIN_EMAIL` | `admin@wiki-collection.local` | Email admin por defecto |
| `CATBOX_USERHASH` | (opcional) Hash de usuario Catbox | Para poder borrar imágenes subidas |
| `RAWG_API_KEY` | (opcional) API key RAWG | Para búsqueda de juegos |
| `GOOGLE_BOOKS_API_KEY` | (opcional) API key Google Books | Para búsqueda de libros |
| `STEAM_API_KEY` | (opcional) API key Steam Web API | Para logros |
| `TMDB_API_KEY` | (opcional) API key TMDB | Para búsqueda de películas/series |
| `BGG_USERNAME` | (opcional)Usuario BoardGameGeek | Para XML API |
| `BGG_PASSWORD` | (opcional) Password BGG | Para XML API |

**Generar JWT_SECRET:**
```bash
# En Linux/Mac
openssl rand -base64 32
```

### Paso 2: Crear el Servicio en Render

1. En el dashboard de Render: **New +** → **Web Service**
2. **Connect repository**: conectar la cuenta GitHub y seleccionar `JavierMedarde99/backend-collection`
3. Configurar el servicio:

| Campo | Valor |
|-------|-------|
| Name | `wiki-collection-backend` |
| Region | 동일한 지역 선택 (ej. Ireland si Atlas está ahí) |
| Branch | `main` |
| Root Directory | (vacío, si el repo es solo backend) |
| Runtime | `Java` (si está disponible) o usar Dockerfile |
| Build Command | `./mvnw clean package -DskipTests` |
| Start Command | `java -jar target/backend-collection-*.jar` |
| Instance Type | **Free** |

**Nota sobre Java 25:** Render puede no tener JDK 25 nativo. Si no está disponible, crear un `Dockerfile` en la raíz del backend:

```dockerfile
FROM eclipse-temurin:25-jdk-alpine

WORKDIR /app

COPY . .

RUN ./mvnw clean package -DskipTests

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "target/backend-collection-*.jar"]
```

Y en Render seleccionar **Docker** como runtime en lugar de Java.

### Paso 3: Variables de Entorno en Render

En la pestaña **Environment** del servicio en Render, añadir todas las variables de la tabla anterior. Marcar las variables sensibles (passwords, secretos) como **Sensitive** para que no se muestren en logs.

### Paso 4: Despliegue

1. Hacer clic en **Create Web Service**
2. Render clonará el repo, ejecutará el build y desplegará
3. El primer despliegue puede tardar varios minutos (descarga de dependencias Maven + compilación)
4. Una vez desplegado, Render asigna una URL: `https://wiki-collection-backend.onrender.com`

### Paso 5: Verificar el Backend

```bash
# Probar que el backend responde
curl -I https://wiki-collection-backend.onrender.com/api/v1/auth/me

# Debería devolver 401 (no autenticado) o 200 si hay sesión activa
```

### Consideraciones del Free Tier de Render

| Aspecto | Detalle |
|---------|---------|
| **Idle timeout** | El servicio se duerme después de 15 min de inactividad |
| **Cold start** | Primer request tras dormir tarda ~30-60 segundos |
| **Recursos** | CPU y RAM compartidos, limitados |
| **Uptime** | No hay SLA para free tier |
| **SSL** | HTTPS automático con certificado de Render |
| **Dominio** | `*.onrender.com` (puede personalizarse con dominio propio) |

**Mejores prácticas para minimizar cold starts:**
- Mantener el servicio "despierto" haciendo un ping periódico (ej. con UptimeRobot o cron)
- O migrar a un plan pago cuando el proyecto necesite mayor disponibilidad

### Configuración de Dominio Personalizado (Opcional)

1. En Render → Settings → Custom Domain
2. Añadir el dominio (ej. `api.tudominio.com`)
3. Configurar DNS en el registrars: `CNAME` o `A` según instrucciones de Render
4. Render gestiona automáticamente el certificado SSL

---

## 3. Frontend en Vercel (React + Vite)

### Requisitos Previos

- Tener el proyecto frontend clonado: `frontend-collection`
- Tener cuenta en [Vercel](https://vercel.com)
- Tener la URL del backend desplegado en Render

### Paso 1: Preparar el Frontend para Producción

#### Variables de Entorno para Vite

Vite necesita variables con prefijo `VITE_` para exponerlas al cliente. Crear `.env.production`:

```env
VITE_API_URL=https://wiki-collection-backend.onrender.com/api/v1
VITE_APP_NAME=Wiki Collection
```

**Nota:** Si se usa dominio personalizado en Render, actualizar `VITE_API_URL` con la URL real.

#### build.sh o configuración de build

El frontend usa Vite. Vercel detecta automáticamente Vite y ejecuta `npm install && npm run build`. No requiere configuración extra si `package.json` tiene el script `build`.

Verificar que `package.json` tenga:
```json
{
  "scripts": {
    "build": "vite build"
  }
}
```

#### Asegurar que Vite no incluya `.env.local`

Vite ya filtra `.env.local` del commit. Añadir `.env` y `.env.production` a `.gitignore`:
```
.env
.env.production
```

Los valores de producción se configuran en Vercel, no en ficheros (ver paso 3).

### Paso 2: Configurar API URL en Vercel

**No hardcodear la URL del backend en el código del frontend.** Configurarla como variable de entorno de Vercel:

1. Desplegar el frontend: **Add New** → **Project** → importar `JavierMedarde99/frontend-collection`
2. En la configuración del proyecto Vercel → **Environment Variables**:
   - Añadir `VITE_API_URL` = `https://wiki-collection-backend.onrender.com/api/v1`
   - Modo: **Production** (y también Development si se hace preview deployments)
3. Guardar y volver a desplegar para que tome efecto

### Paso 3: Despliegue Automático

1. Vercel sincroniza con el repo GitHub
2. Cada push a `main` desencadena un nuevo despliegue
3. Vercel ejecuta `npm install && npm run build` y publica los estáticos
4. URL de producción: `https://tu-proyecto.vercel.app`

### Configuración de Dominio Personalizado (Opcional)

1. En Vercel → Settings → Domains
2. Añadir el dominio (ej. `wiki-collection.vercel.app` o dominio propio)
3. Sigue las instrucciones de DNS (normalmente `CNAME` o `ALIAS`)
4. SSL automático

---

## 4. Conectar Frontend y Backend

### CORS en el Backend

Spring Boot debe aceptar requests del frontend. En `WebConfig.java`:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Value("${app.cors.allowed-origins}")
    private String allowedOrigins;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins(allowedOrigins.split(","))
                .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS")
                .allowedHeaders("*")
                .exposedHeaders("Authorization")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

Y en `application-prod.properties`:
```properties
app.cors.allowed-origins=https://tu-proyecto.vercel.app,https://wiki-collection-backend.onrender.com
```

### Autenticación entre Frontend y Backend

1. El frontend hace login via `POST /api/v1/auth/login`
2. El backend devuelve `accessToken` y `refreshToken`
3. El frontend guarda los tokens (localStorage) y los envía en headers:
   ```
   Authorization: Bearer <accessToken>
   ```
4. El backend valida el JWT vía `JwtAuthenticationFilter`

### Handling del Refresh de Token

El frontend debe manejar el refresh automático cuando el access token expire (15 min). La implementación actual en `authFetch.ts` debería:

1. Interceptar respuestas 401
2. Intentar refresh con el refresh token
3. Si el refresh funciona, reintentar la petición original con el nuevo access token
4. Si el refresh falla, redirigir al login

---

## 5. Variables de Entorno Resumen

### Backend (Render Environment Variables)

```
MONGODB_URI=mongodb+srv://wikiadmin:XXXXX@cluster0.xxxxx.mongodb.net/wiki_collection?retryWrites=true&w=majority
SERVER_PORT=8080
JWT_SECRET=<openssl rand -base64 32>
JWT_ACCESS_TOKEN_EXPIRATION=900000
JWT_REFRESH_TOKEN_EXPIRATION=604800000
ADMIN_USERNAME=admin
ADMIN_PASSWORD=<contraseña-segura>
ADMIN_EMAIL=admin@wiki-collection.local
CATBOX_USERHASH=<opcional>
RAWG_API_KEY=<opcional>
GOOGLE_BOOKS_API_KEY=<opcional>
STEAM_API_KEY=<opcional>
TMDB_API_KEY=<opcional>
BGG_USERNAME=<opcional>
BGG_PASSWORD=<opcional>
APP_CORS_ALLOWED_ORIGINS=https://tu-proyecto.vercel.app
SPRING_PROFILES_ACTIVE=prod
SPRINGDOC_API_DOCS_ENABLED=false
SPRINGDOC_SWAGGER_UI_ENABLED=false
```

### Frontend (Vercel Environment Variables)

```
VITE_API_URL=https://wiki-collection-backend.onrender.com/api/v1
```

---

## 6. Despliegue Inicial Paso a Paso

### Orden recomendado

1. **MongoDB Atlas** (paso 1 del documento)
2. **Backend en Render** (paso 2 del documento)
3. **Frontend en Vercel** (paso 3 del documento)

### Verificación Post-Despliegue

```bash
# 1. Verificar que el backend está online
curl -I https://wiki-collection-backend.onrender.com/api/v1/stats/global

# Debería devolver 401 (no autenticado) con headers CORS correctos

# 2. Verificar CORS
curl -I -X OPTIONS https://wiki-collection-backend.onrender.com/api/v1/books \
  -H "Origin: https://tu-proyecto.vercel.app" \
  -H "Access-Control-Request-Method: GET"

# Debería devolver 200 OK con Access-Control-Allow-Origin

# 3. Probar registro de usuario (desde frontend o curl)
curl -X POST https://wiki-collection-backend.onrender.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@wiki.local",
    "password": "TestPass123!",
    "displayName": "Test User"
  }'

# Debería devolver 201 con tokens

# 4. Verificar frontend
curl -I https://tu-proyecto.vercel.app

# Debería devolver 200 con index.html
```

---

## 7. CI/CD con GitHub Actions (Opcional)

### Backend — Build + Test antes de deploy

`.github/workflows/backend.yml`:

```yaml
name: Backend CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 25
        uses: actions/setup-java@v4
        with:
          java-version: '25'
          distribution: 'temurin'

      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2

      - name: Build with Maven
        run: ./mvnw clean verify

      - name: Upload JAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: backend-jar
          path: target/*.jar
```

### Frontend — Build + Preview

Vercel ya hace build automático en cada push. No requiere GitHub Actions para despliegue básico. Si se quiere, se puede añadir un workflow que ejecute `npm run test` antes de permitir el merge.

---

## 8. Costes Reales

| Servicio | Plan Gratis | Límites | Cuándo pagar |
|----------|-------------|---------|--------------|
| MongoDB Atlas | M0 (Shared) | 512 MB, 3 conexiones | Si la BD crece >512 MB o se necesitan backups automatizados |
| Render | Free | Duerme tras 15 min, recursos limitados | Si se necesita uptime continua o más recursos |
| Vercel | Hobby | Proyectos personales OK, límites de bandwidth | Si el tráfico crece mucho o se necesita dominio propio en team |

**Coste mensual estimado (solo planes gratis): $0**

**Coste mensual si se upgradea backend a Render Paid (starter): ~$7/mes**
**Coste mensual si se upgradea MongoDB a Atlas M10: ~$57/mes**

---

## 9. Migración de Datos Locales a Atlas

Si ya hay datos en MongoDB local y se quiere migrar a Atlas:

### Opción A: mongodump + mongorestore

```bash
# 1. Dump local
mongodump --uri="mongodb://localhost:27017/wiki_collection" \
  --out=/tmp/wiki-dump

# 2. Restore a Atlas
mongorestore --uri="mongodb+srv://wikiadmin:XXXXX@cluster0.xxxxx.mongodb.net/wiki_collection" \
  /tmp/wiki-dump/wiki_collection
```

### Opción B: MongoDB Compass (GUI)

1. Conectar Compass a MongoDB local
2. Conectar Compass a Atlas
3. Copiar colecciones manualmente o exportar/importar

### Opción C: Script de migración en Java

Si los datos son pocos, se puede escribir un script Spring Boot que lea de la BD local y escriba en Atlas (usando dos `MongoTemplate` con diferentes URIs).

---

## 10. Rollback y Troubleshooting

### Backend no arranca en Render

1. Revisar **Logs** en la dashboard de Render
2. Errores comunes:
   - `MONGODB_URI` incorrecta → verificar usuario, password, IP whitelist
   - `JWT_SECRET` demasiado corto → debe ser al menos 256 bits (32 bytes base64)
   - Puerto incorrecto → Render usa la variable `PORT` (o `SERVER_PORT`)
   - Faltan variables de entorno → revisar lista completa

### Frontend no puede conectar al Backend

1. Verificar `VITE_API_URL` en Vercel (incluye `/api/v1` al final)
2. Verificar CORS en backend
3. Probar desde el navegador: abrir DevTools → Network y ver la petición fallida
4. Si el backend está "dormido" (Render free), el primer request tardará 30-60s

### MongoDB Atlas conectar y fallar

1. Verificar **Database Access**: usuario y password correctos
2. Verificar **Network Access**: IP del servidor Render permitida (o 0.0.0.0/0)
3. Verificar **URI**: que sea `mongodb+srv://` y no `mongodb://` para Atlas

### Tokens JWT y autenticación

1. Si `JWT_SECRET` cambia entre despliegues, todos los tokens existentes dejan de ser válidos
2. Los usuarios tendrán que hacer login de nuevo
3. Guardar `JWT_SECRET` en un lugar seguro (no en git, sino en secrets de Render)

---

## 11. Checklist de Despliegue

- [ ] Cuenta MongoDB Atlas creada
- [ ] Cluster free creado
- [ ] Usuario de base de datos creado
- [ ] Network access configurado (0.0.0.0/0 o IP de Render)
- [ ] URI de conexión copiada
- [ ] Cuenta Render creada y conectada a GitHub
- [ ] Repositorio `backend-collection` enlazado
- [ ] Variables de entorno configuradas en Render (todas las requeridas)
- [ ] build command y start command configurados
- [ ] Backend desplegado y online (URL asignada)
- [ ] Backend respondiendo (curl de prueba)
- [ ] CORS configurado para dominio Vercel
- [ ] Cuenta Vercel creada y conectada a GitHub
- [ ] Repositorio `frontend-collection` enlazado
- [ ] Variable `VITE_API_URL` configurada en Vercel
- [ ] Frontend desplegado y online
- [ ] Login/registro probados desde frontend
- [ ] JWT funcionando (access + refresh)
- [ ] Dominios personalizados configurados (opcional)
- [ ] Variables sensibles marcadas como secrets en Render y Vercel
- [ ] Backups de Atlas configurados (manuales para free tier)

---

## Notas Importantes

- **Render free tier idle**: El backend se duerme tras 15 min sin actividad. El primer request después tarda ~30-60s. Para un proyecto personal puede ser aceptable, pero si se necesita disponibilidad inmediata, considerar un ping keepalive o un plan upgradeado.
- **JDK 25 en Render**: Si Render no soporta JDK 25 nativamente, usar un Dockerfile con `eclipse-temurin:25-jdk-alpine`. Verificar que el runtime Docker esté disponible en el plan free.
- **MongoDB Atlas 512 MB**: Para una colección personal de libros, juegos, cartas y películas, 512 MB suele ser suficiente. Si se suben muchas imágenes (Catbox las almacena externo, no en MongoDB), el espacio sigue siendo suficiente.
- **CORS**: Es el problema más común al conectar frontend y backend en producción. Configurar `app.cors.allowed-origins` correctamente y testear con curl antes de abrir el frontend.
- **JWT_SECRET**: Cambiar el secreto invalida todos los tokens. Mantenerlo estable entre despliegues (almacenarlo como variable persistente en Render).
- **API keys externas**: Scryfall no requiere clave. Para RAWG, Google Books, Steam, TMDB y Catbox, las claves son opcionales en desarrollo pero recomendables en producción para rate limits más altos. No commitearlas en el repo; usar variables de entorno.
