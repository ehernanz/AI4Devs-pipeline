# GitHub Actions CI/CD Pipeline

Este pipeline automatiza el proceso de testing, build y deploy del backend cuando se hace push a una rama con Pull Request abierto.

## 🚀 Configuración del Pipeline

### Estructura del Workflow

El pipeline consta de 4 jobs principales:

1. **Test**: Ejecuta los tests del backend
2. **Build**: Compila TypeScript y crea el paquete de despliegue
3. **Deploy**: Despliega en EC2 usando SSH
4. **Notify**: Notifica el resultado del despliegue

### Triggers

El pipeline se ejecuta en los siguientes eventos:
- Push a cualquier rama con PR abierto
- Cambios en los archivos del directorio `backend/`
- Cambios en los archivos de workflows

## ⚙️ Configuración Requerida

### 1. Secrets de GitHub

Debes configurar los siguientes secrets en tu repositorio de GitHub:
(`Settings` > `Secrets and variables` > `Actions`)

| Secret | Descripción | Ejemplo |
|--------|-------------|---------|
| `EC2_HOST` | IP pública de tu instancia EC2 | `54.123.456.789` |
| `EC2_USERNAME` | Usuario SSH (normalmente `ubuntu` o `ec2-user`) | `ubuntu` |
| `EC2_PRIVATE_KEY` | Clave privada SSH para conectar a EC2 | `-----BEGIN RSA PRIVATE KEY-----...` |

**Nota**: No se requiere `DATABASE_URL` como secret ya que el archivo `.env` debe estar pre-configurado en el servidor EC2.

### 2. Preparación de la Instancia EC2

Tu instancia EC2 debe tener instalado:

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Instalar PM2 globalmente
sudo npm install -g pm2

# Instalar PostgreSQL client (si usas Prisma)
sudo apt-get install -y postgresql-client

# Configurar PM2 para iniciar con el sistema
pm2 startup
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

**⚠️ IMPORTANTE: Configurar archivo .env**

Debes crear manualmente el archivo `.env` en `/home/ubuntu/app/.env` (o el directorio de tu usuario) con las variables de entorno necesarias:

```bash
# Conectarse a EC2
ssh -i tu-clave.pem ubuntu@tu-ec2-ip

# Crear el archivo .env
nano /home/ubuntu/app/.env
```

Contenido del archivo `.env`:
```bash
DATABASE_URL=postgresql://usuario:password@host:5432/nombre_bd
JWT_SECRET=tu_jwt_secret_aqui
API_KEY=tu_api_key_aqui
# Agregar otras variables que necesite tu aplicación
# NODE_ENV se actualizará automáticamente por el pipeline
# PORT se agregará automáticamente si no existe
```

### 3. Configuración del package.json del Backend

Asegúrate de que tu `backend/package.json` tenga estos scripts:

```json
{
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "start": "node dist/index.js"
  }
}
```

## 🔧 Personalización

### Modificar el Trigger de Deploy

Por defecto, el deploy solo se ejecuta si:
- El PR se mergea, O
- El PR tiene la etiqueta `deploy`

Para cambiar esto, modifica la condición en el job `deploy`:

```yaml
if: github.event.pull_request.merged == true || contains(github.event.pull_request.labels.*.name, 'deploy')
```

### Cambiar el Puerto de la Aplicación

Si tu aplicación usa un puerto diferente al 3000, modifica:

1. La variable de entorno `PORT` en el step de deploy
2. La URL del health check en el step de verificación

### Agregar Variables de Entorno Adicionales

En el step "Configurar variables de entorno" del job deploy, agrega:

```bash
echo "MI_VARIABLE=${{ secrets.MI_VARIABLE }}" >> .env
```

## 📊 Monitoreo y Logs

### Ver el Estado de la Aplicación en EC2

```bash
# Conectarse a EC2
ssh -i tu-clave.pem ubuntu@tu-ec2-ip

# Ver estado de PM2
pm2 status

# Ver logs
pm2 logs backend

# Restart manual si es necesario
pm2 restart backend
```

### Logs del Pipeline

Los logs de cada job están disponibles en:
`GitHub Repository` > `Actions` > `Tu Workflow Run`

## 🐛 Troubleshooting

### Error: "Host key verification failed"

Si el pipeline falla en el SSH, puede ser un problema de verificación de host. El pipeline incluye `-o StrictHostKeyChecking=no` pero si persiste, verifica que:

1. La clave SSH sea correcta
2. El usuario tenga permisos SSH
3. El security group de EC2 permita conexiones SSH desde GitHub Actions

### Error: "Permission denied (publickey)"

1. Verifica que `EC2_PRIVATE_KEY` sea la clave privada completa
2. Asegúrate de que la clave pública esté en `~/.ssh/authorized_keys` del usuario EC2

### Error: "DATABASE_URL not found"

Si el pipeline falla con este error:
1. Verifica que el archivo `.env` exista en `/home/tu-usuario/app/.env`
2. Asegúrate de que contenga todas las variables necesarias
3. Verifica permisos de lectura: `chmod 644 /home/tu-usuario/app/.env`

## 🔒 Seguridad

### Buenas Prácticas Implementadas

- ✅ Uso de secrets de GitHub para datos sensibles
- ✅ Conexión SSH con clave privada (no contraseñas)
- ✅ Limpieza de archivos temporales después del despliegue
- ✅ Variables de entorno separadas por ambiente
- ✅ Verificación de salud después del despliegue

### Recomendaciones Adicionales

1. **Rotate SSH Keys**: Cambia las claves SSH periódicamente
2. **Database Security**: Usa conexiones SSL para la base de datos
3. **Network Security**: Configura security groups restrictivos en EC2
4. **Monitoring**: Implementa logging y monitoring en producción

## 📝 Próximos Pasos

1. Configurar los secrets en GitHub
2. Preparar la instancia EC2
3. Crear un PR de prueba para validar el pipeline
4. Agregar health checks más robustos
5. Considerar blue-green deployments para zero-downtime