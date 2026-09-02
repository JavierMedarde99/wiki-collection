# Deploy

## Estado Actual

El deploy aún no está configurado. El proyecto se ejecuta localmente con Maven y npm.

## Planificación Futura

### Backend

**Opción A: VPS (DigitalOcean, Hetzner)**
- Docker + Docker Compose
- Nginx como reverse proxy
- SSL con Let's Encrypt
- CI/CD con GitHub Actions

**Opción B: PaaS (Railway, Render)**
- Deploy automático desde GitHub
- Variables de entorno para configuración
- Escalado automático

### Frontend

**Opción A: Vercel/Netlify**
- Deploy automático desde GitHub
- CDN global
- Preview deployments por PR

**Opción B: S3 + CloudFront**
- Hosting estático en S3
- CDN con CloudFront
- Deploy con AWS CLI

### Base de Datos

**Opción A: MongoDB Atlas**
- Cluster gratuito (512MB)
- Backups automáticos
- Monitoreo integrado

**Opción B: MongoDB en VPS**
- Instalación manual
- Backups con cron
- Mayor control

## Variables de Entorno

```bash
# Backend
SPRING_MONGODB_URI=mongodb://localhost:27017/wiki_collection
SERVER_PORT=8080
GOOGLE_BOOKS_API_KEY=optional

# Frontend
VITE_API_URL=http://localhost:8080/api
```

## Docker (Planificado)

```yaml
# docker-compose.yml
version: "3.8"
services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_MONGODB_URI=mongodb://mongo:27017/wiki_collection
    depends_on:
      - mongo
  
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - VITE_API_URL=http://localhost:8080/api
  
  mongo:
    image: mongo:7
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```
