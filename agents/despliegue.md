# 🚀 Agente Despliegue — De local a producción con dominio propio

> Usá este agente cuando quieras publicar un proyecto en internet con dominio y SSL

## Arquitectura objetivo (lo que vamos a armar)

```
Internet
    ↓
[Dominio: tuapp.com]  ←  Registrar en Namecheap / Porkbun (~$10/año)
    ↓
[Cloudflare]  ←  DNS + protección DDoS + SSL gratuito (opcional pero recomendado)
    ↓
[VPS]  ←  Hostinger / DigitalOcean / Contabo (~$4-6 USD/mes)
    ↓
[Nginx]  ←  Reverse proxy, SSL termination, sirve el frontend
    ↓
┌──────────────────────────────────────────┐
│  Docker Compose                          │
│  ┌──────────┐  ┌──────────┐  ┌───────┐  │
│  │ Frontend │  │ Backend  │  │  DB   │  │
│  │ Next.js  │  │ FastAPI/ │  │Postgres│ │
│  │ :3000    │  │ Express  │  │ :5432 │  │
│  │          │  │ :8000    │  │       │  │
│  └──────────┘  └──────────┘  └───────┘  │
└──────────────────────────────────────────┘
```

---

## Paso 0 — Preparar el proyecto localmente

### Docker Compose para desarrollo local
```yaml
# docker-compose.yml (en la raíz del proyecto)
version: '3.9'

services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      JWT_SECRET: ${JWT_SECRET}
    ports:
      - "8000:8000"
    depends_on:
      - db

  frontend:
    build: ./frontend
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:8000
    ports:
      - "3000:3000"
    depends_on:
      - backend

volumes:
  postgres_data:
```

```bash
# Comandos básicos
docker-compose up -d          # levantar todo en background
docker-compose down           # bajar todo
docker-compose logs backend   # ver logs de un servicio
docker-compose ps             # ver estado
```

---

## Paso 1 — Elegir y configurar el VPS

### Opciones recomendadas para estudiantes
| Proveedor | Plan mínimo | Precio | Recomendado para |
|---|---|---|---|
| **Hostinger VPS** | 1 CPU, 1GB RAM | ~$4/mes | Empezar, interfaz amigable |
| **DigitalOcean** | 1 CPU, 1GB RAM | $6/mes | Más profesional, mejor docs |
| **Contabo** | 4 CPU, 4GB RAM | €4.5/mes | Más potencia por el precio |

Para proyectos universitarios, **Hostinger** o **DigitalOcean** con el plan más básico es suficiente.

### Configuración inicial del servidor (Ubuntu 22.04)
```bash
# Conectarse al VPS
ssh root@IP_DEL_SERVIDOR

# Actualizar sistema
apt update && apt upgrade -y

# Instalar Docker
curl -fsSL https://get.docker.com | sh
systemctl enable docker
systemctl start docker

# Instalar Docker Compose
apt install docker-compose-plugin -y

# Instalar Nginx
apt install nginx -y
systemctl enable nginx

# Crear usuario no-root para trabajar
adduser tuusuario
usermod -aG sudo,docker tuusuario

# Configurar firewall
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

---

## Paso 2 — Subir el proyecto al servidor

```bash
# En el servidor: clonar el repo
git clone https://github.com/TU-USUARIO/TU-REPO.git /var/www/tuapp

# Crear archivo .env en producción (NUNCA subir al repo)
cd /var/www/tuapp
nano .env
# Pegar las variables de producción

# Levantar con Docker Compose
docker compose up -d --build
```

---

## Paso 3 — Configurar Nginx como reverse proxy

```nginx
# /etc/nginx/sites-available/tuapp.com
server {
    listen 80;
    server_name tuapp.com www.tuapp.com;

    # Frontend (Next.js)
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API
    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
# Activar el sitio
ln -s /etc/nginx/sites-available/tuapp.com /etc/nginx/sites-enabled/
nginx -t          # verificar que no hay errores
systemctl reload nginx
```

---

## Paso 4 — SSL gratuito con Let's Encrypt

```bash
# Instalar Certbot
apt install certbot python3-certbot-nginx -y

# Obtener certificado SSL (reemplazá con tu dominio)
certbot --nginx -d tuapp.com -d www.tuapp.com

# Renovación automática (ya la configura Certbot, verificar con:)
systemctl status certbot.timer
```

Después de esto tu sitio tiene HTTPS automáticamente. Certbot modifica el Nginx solo.

---

## Paso 5 — Dominio (DNS)

En tu registrador de dominios (Namecheap, Porkbun, etc.):

```
Tipo    Nombre    Valor               TTL
A       @         IP_DEL_SERVIDOR     Auto
A       www       IP_DEL_SERVIDOR     Auto
```

Si usás Cloudflare (recomendado):
1. Agregar el sitio en Cloudflare
2. Cambiar los nameservers en tu registrador
3. En Cloudflare: DNS → agregar los registros A
4. SSL/TLS → Full (strict)

---

## Paso 6 — CI/CD básico con GitHub Actions (opcional)

```yaml
# .github/workflows/deploy.yml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /var/www/tuapp
            git pull origin main
            docker compose up -d --build
            docker system prune -f
```

Secretos a configurar en GitHub → Settings → Secrets:
- `VPS_HOST`: IP del servidor
- `VPS_USER`: usuario SSH
- `VPS_SSH_KEY`: clave privada SSH

---

## Checklist antes de ir a producción

```markdown
### Seguridad
- [ ] Variables de entorno en .env, NO en el código
- [ ] .env está en .gitignore
- [ ] JWT_SECRET es una cadena larga aleatoria (no "secret123")
- [ ] CORS configurado solo para tu dominio
- [ ] Rate limiting en rutas de autenticación

### Infraestructura
- [ ] Firewall activo (ufw)
- [ ] SSL activo (HTTPS)
- [ ] Backups de PostgreSQL configurados
- [ ] Logs centralizados o accesibles con docker compose logs

### Aplicación
- [ ] NODE_ENV=production / ENVIRONMENT=production
- [ ] Sin console.log innecesarios
- [ ] Manejo de errores no expone stack traces
- [ ] Health check endpoint: GET /health → { status: "ok" }
```

---

## Comandos útiles en el servidor

```bash
# Ver logs en tiempo real
docker compose logs -f backend

# Ver uso de recursos
docker stats

# Reiniciar un servicio
docker compose restart backend

# Backup de la base de datos
docker compose exec db pg_dump -U $DB_USER $DB_NAME > backup_$(date +%Y%m%d).sql

# Restaurar backup
cat backup.sql | docker compose exec -T db psql -U $DB_USER $DB_NAME

# Ver espacio en disco
df -h
```
