# Odoo 17 con Docker Compose + Nginx

Este repositorio contiene un despliegue de **Odoo 17** utilizando **Docker Compose**, con:

- PostgreSQL como base de datos
- Nginx como reverse‑proxy
- Soporte para módulos personalizados
- Persistencia de datos
- Preparado para subir archivos grandes


---

## 📦 Estructura del repositorio

```
.
├─ docker-compose.yml
├─ Dockerfile
├─ requirements.txt
├─ addons/                # Módulos personalizados (community)
├─ enterprise/            # Módulos enterprise (si aplica)
├─ config/                # Configuración de Odoo (odoo.conf)
├─ nginx/
│   └─ nginx.conf         # Configuración de Nginx
├─ data/
│   └─ filestore/         # Filestore persistente de Odoo
└─ .dockerignore
```


---

## 🧩 Servicios incluidos

### 🔹 Nginx

Funciona como proxy inverso delante de Odoo.

- Publica el servicio en el puerto 80
- Reenvía el tráfico a Odoo (puerto 8069 interno)
- Soporta WebSockets (bus de Odoo)
- Permite subidas de archivos grandes


### 🔹 Odoo 17

Se construye usando el `Dockerfile` del repositorio.

Incluye:

- Configuración montada desde `./config`
- Addons personalizados desde `./addons`
- Filestore persistente en `./data/filestore`


### 🔹 PostgreSQL

Base de datos del sistema.

- Versión: PostgreSQL 17
- Datos persistidos en un volumen Docker


---

## 🗂️ Volúmenes

| Servicio | Ruta en contenedor | Origen |
------|------------------|-------|
| Odoo | /var/lib/odoo/filestore | ./data/filestore |
| Odoo | /mnt/extra-addons | ./addons |
| Odoo | /etc/odoo | ./config |
| PostgreSQL | /var/lib/postgresql/data | volumen Docker |


---

## 🚀 Cómo utilizar este repositorio

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd <carpeta-del-repositorio>
```


---

### 2. Preparar carpetas necesarias

Desde la raíz del proyecto:

```bash
mkdir -p data/filestore
```


---

### 3. Ajustar permisos (muy importante)

Odoo se ejecuta con el usuario interno `uid 101`.

```bash
sudo chown -R 101:101 data
```

Esto evita errores de tipo:

```
PermissionError: [Errno 13] Permission denied
```


---

### 4. Revisar configuración de Odoo

Editar:

```
config/odoo.conf
```

Debe incluir como mínimo:

```
[options]
proxy_mode = True
limit_request_body = 16106127360
```

(Este valor permite subidas grandes a través de Nginx).


---

### 5. Construir e iniciar los contenedores

```bash
docker compose up -d --build
```


---

### 6. Acceso a Odoo

Desde el navegador:

```
http://IP_DEL_SERVIDOR
```

El acceso se realiza siempre a través de Nginx.


---

## ⚠️ Nota sobre el puerto 8069

El servicio de Odoo no se expone directamente al host.

Todo el tráfico debe pasar por Nginx.

Esto es el comportamiento recomendado para producción.


---

## 📁 Cómo añadir módulos personalizados

Coloca tus módulos en:

```
addons/
```

Después reinicia Odoo:

```bash
docker compose restart odoo
```


---

## 📂 Filestore de Odoo

El filestore se almacena en:

```
./data/filestore
```

Internamente Odoo crea una carpeta por cada base de datos:

```
./data/filestore/<nombre_base_datos>
```

No se debe montar directamente una subcarpeta concreta.


---

## 📦 Cómo se distribuye este repositorio

Este repositorio está pensado para distribuirse como:

- Proyecto Docker Compose autocontenido
- Preparado para ser desplegado en servidores Linux
- Compatible con entornos de desarrollo y producción


### El repositorio no incluye:

- Bases de datos
- Filestore real
- Módulos enterprise con licencia


Los datos reales se generan en tiempo de ejecución dentro de:

```
./data/
```

y en los volúmenes Docker.


---

## 🔁 Actualización del proyecto

Para actualizar el código o los módulos:

```bash
git pull
docker compose up -d --build
```


---

## 🛑 Parar el entorno

```bash
docker compose down
```


---

## 💡 Recomendaciones

- No subir el directorio `data/` al repositorio
- No subir backups ni filestore a Git
- Mantener los módulos enterprise fuera del repositorio público
- Utilizar HTTPS en producción (Nginx + Let's Encrypt)


---

## 🧪 Comprobación rápida del estado

```bash
docker compose ps
```


---

## 📌 Resumen

Este repositorio proporciona un entorno completo para ejecutar Odoo 17 con:

- Proxy inverso Nginx
- Persistencia de datos
- Soporte de módulos personalizados
- Preparación para cargas de archivos grandes

Listo para desplegar en servidor o entorno de pruebas.

