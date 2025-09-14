# **User**
 Eres un DevOps enginmeer y se ta ha solicitado crear un pipeline usando github actions que, tras el trigger "push a una rama con un Pull Request abierto", siga los siguientes pasos:

Pase unos tests de backend.
Genere un build del backend.
Despliegue el backend en un EC2. 


Tienes que configurar el workflow de GitHub Actions en un archivo .github/workflows/pipeline.yml.

Explicame lo que has entendio antes de comenzar a generar código.

# **GitHub Copilot**
 He entendido que necesitas crear un pipeline de CI/CD usando GitHub Actions con las siguientes características:

## Contexto entendido:
- **Trigger**: Se ejecuta cuando hay un push a una rama que tiene un Pull Request abierto
- **Aplicación**: Backend en TypeScript/Node.js (veo que tienes un backend con package.json, tests con Jest, y Prisma como ORM)
- **Pasos del pipeline**:
  1. **Tests**: Ejecutar tests del backend (veo que tienes archivos `*.test.ts`)
  2. **Build**: Generar el build del backend
  3. **Deploy**: Desplegar en una instancia EC2

## Consideraciones técnicas que veo en tu proyecto:
- Backend en TypeScript con dependencias en package.json
- Tests configurados con Jest (`jest.config.js`)
- Base de datos con Prisma (migrations y schema)
- Estructura organizada con servicios, controladores y modelos

## Lo que necesitaré configurar:
- Workflow que se active en push a ramas con PR abierto
- Job para tests (instalar deps, ejecutar tests)
- Job para build (compilar TypeScript)
- Job para deploy (conectar a EC2, transferir archivos, reiniciar servicios)
- Variables de entorno y secretos necesarios para EC2

¿Es correcta mi comprensión? ¿Hay algún detalle específico sobre el entorno de EC2 (tipo de instancia, servicios ya configurados, etc.) o configuraciones adicionales que deba considerar?

# **User**
 si, peudes comenzar

# **GitHub Copilot**
 Perfecto, voy a crear el pipeline de GitHub Actions. Primero voy a crear un plan para organizar el trabajo: 

Created 5 todos

Ahora voy a comenzar con el primer paso: 

Starting (1/5) *Crear directorio .github/workflows*

Created [](file:///c%3A/Users/ehm19/Documents/GitHub/AI4Devs-pipeline_2/.github/workflows)

Starting (2/5) *Configurar workflow pipeline.yml*

Ahora voy a crear el archivo de workflow con la configuración completa: 

Created [](file:///c%3A/Users/ehm19/Documents/GitHub/AI4Devs-pipeline_2/.github/workflows/pipeline.yml)

Los errores son avisos normales de lint porque los secrets no están definidos aún. Vamos a completar los todos restantes: 

Completed (2/5) *Configurar workflow pipeline.yml*

Perfecto! He creado el pipeline completo de GitHub Actions. Ahora voy a crear un archivo de documentación para explicar cómo configurarlo: 

Created [](file:///c%3A/Users/ehm19/Documents/GitHub/AI4Devs-pipeline_2/PIPELINE_SETUP.md)

## ✅ Pipeline Completado

He creado exitosamente el pipeline de GitHub Actions con las siguientes características:

### 📋 Lo que se ha implementado:

1. **Workflow completo** en pipeline.yml que incluye:
   - **Trigger**: Se activa con push a ramas con PR abierto
   - **Tests**: Ejecuta tests de Jest con base de datos PostgreSQL
   - **Build**: Compila TypeScript y crea paquete de despliegue
   - **Deploy**: Despliega en EC2 usando SSH y PM2
   - **Verificación**: Valida que el despliegue sea exitoso

2. **Documentación completa** en PIPELINE_SETUP.md con:
   - Instrucciones de configuración paso a paso
   - Lista de secrets requeridos
   - Preparación de instancia EC2
   - Troubleshooting y mejores prácticas

### 🔧 Características técnicas:

- **Tests automatizados** con base de datos PostgreSQL en contenedor
- **Build optimizado** con cache de dependencias
- **Deploy seguro** con SSH y verificación de salud
- **Gestión de procesos** con PM2 para alta disponibilidad
- **Variables de entorno** configurables por secrets
- **Rollback automático** en caso de fallo

### 📚 Próximos pasos:

1. **Configurar secrets** en GitHub (EC2_HOST, EC2_USERNAME, EC2_PRIVATE_KEY, DATABASE_URL)
2. **Preparar EC2** con Node.js, PM2 y dependencias
3. **Crear PR de prueba** para validar el pipeline
4. **Revisar y ajustar** configuraciones específicas de tu entorno

El pipeline está listo para usar y seguirá las mejores prácticas de DevOps con testing automatizado, builds reproducibles y deploys seguros. 🚀

Made changes.

# **User**
 la variable DATABASE_URL está definida en el fichero .env del backend, poque la tengo que añadir como secreto en github?

# **GitHub Copilot**
 Excelente pregunta. Hay una diferencia importante entre el entorno de desarrollo local y el entorno de producción en EC2:

## 🔍 **¿Por qué DATABASE_URL como secret?**

### **En tu entorno local:**
- Usas el archivo .env del backend para desarrollo
- La base de datos probablemente es local o de desarrollo
- Los datos no son críticos y las credenciales pueden ser simples

### **En producción (EC2):**
- La base de datos de producción tiene credenciales diferentes y más seguras
- La URL incluye datos sensibles (usuario, contraseña, host de producción)
- **El archivo .env local NO debe contener credenciales de producción**

## 🔧 **Lo que hace el pipeline:**

Mira esta parte del código que seleccionaste:

```yaml
# Configurar variables de entorno
echo "DATABASE_URL=$DATABASE_URL" > .env
echo "NODE_ENV=production" >> .env
echo "PORT=3000" >> .env
```

El pipeline **sobrescribe** el archivo .env en el servidor EC2 con:
1. `DATABASE_URL` desde el secret de GitHub (base de datos de producción)
2. `NODE_ENV=production` 
3. `PORT=3000`

## 💡 **Alternativas más seguras:**

Si prefieres no usar secrets de GitHub, puedes:

### **Opción 1: .env ya configurado en EC2**
```yaml
# En lugar de crear el .env, solo asegurarse que existe
ssh -i private_key.pem -o StrictHostKeyChecking=no $EC2_USERNAME@$EC2_HOST << 'EOF'
  cd /home/$EC2_USERNAME/app/deployment-package/
  
  # Verificar que existe .env en el servidor
  if [ ! -f .env ]; then
    echo "ERROR: .env file not found in EC2!"
    exit 1
  fi
  
  # Solo actualizar NODE_ENV
  sed -i 's/NODE_ENV=.*/NODE_ENV=production/' .env
EOF
```

### **Opción 2: Usar AWS Systems Manager Parameter Store**
```yaml
# Obtener DATABASE_URL desde AWS Parameter Store
aws ssm get-parameter --name "/app/database-url" --with-decryption --query 'Parameter.Value' --output text > .env
```

## 🎯 **Mi recomendación:**

**Mantén el secret de GitHub** porque:
- ✅ Separa configuraciones de desarrollo y producción
- ✅ Más seguro (credenciales no quedan en código)
- ✅ Más fácil rotar credenciales
- ✅ Sigue mejores prácticas de DevOps

¿Te gustaría que modifique el pipeline para usar alguna de las alternativas, o prefieres mantener la configuración actual con secrets?

# **User**
 usa la Opción 1 por favor

# **GitHub Copilot**
 Perfecto! Voy a modificar el pipeline para usar la Opción 1, donde el archivo .env ya está configurado en EC2 y solo actualizamos el `NODE_ENV`. 

Using "Replace String in File"


```

```
Ahora voy a actualizar la documentación para reflejar este cambio: 

Using "Replace String in File"


```

```


Using "Replace String in File"


```

```


Using "Replace String in File"


```

```
## ✅ Pipeline Actualizado - Opción 1 Implementada

He modificado el pipeline para usar la **Opción 1** como solicitaste. Aquí están los cambios principales:

### 🔄 **Cambios realizados:**

1. **Eliminado el secret `DATABASE_URL`** de las variables de entorno del job deploy
2. **Nueva lógica de configuración:**
   - Verifica que existe `/home/$EC2_USERNAME/app/.env`
   - Copia el .env existente al directorio de despliegue
   - Solo actualiza `NODE_ENV=production` usando `sed`
   - Agrega `PORT=3000` si no existe

3. **Actualizada la documentación** con instrucciones claras sobre cómo crear el .env en EC2

### 📋 **Secrets necesarios ahora:**
- ✅ `EC2_HOST`
- ✅ `EC2_USERNAME` 
- ✅ `EC2_PRIVATE_KEY`
- ❌ ~~`DATABASE_URL`~~ (ya no necesario)

### 🛠️ **Pasos que debes seguir:**

1. **Conectarte a tu EC2:**
   ```bash
   ssh -i "Ejercicio_CICD.pem" ec2-user@ec2-3-120-237-131.eu-central-1.compute.amazonaws.com
   ```

2. **Crear el directorio y archivo .env:**
   ```bash
   mkdir -p /home/ec2-user/app
   nano /home/ec2-user/app/.env
   ```

3. **Configurar el .env con tus variables:**
   ```bash
   DATABASE_URL=postgresql://usuario:password@host:5432/bd_produccion
   JWT_SECRET=tu_secret_jwt
   # Otras variables que necesite tu app
   ```

4. **Configurar solo los 3 secrets en GitHub** (sin DATABASE_URL)

### 🎯 **Ventajas de este enfoque:**
- ✅ Más seguro (credenciales solo en el servidor)
- ✅ Menos secrets de GitHub que gestionar
- ✅ Fácil cambio de configuración sin redeployar
- ✅ Separación clara entre desarrollo y producción

¡El pipeline está listo para usar con esta configuración más segura!

Made changes.