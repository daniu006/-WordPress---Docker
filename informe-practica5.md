# Práctica No. 5 — Despliegue de WordPress con Docker (Red y Volúmenes)

## 1. Título

Creación de contenedores para un sitio WordPress utilizando Docker: red personalizada, volúmenes persistentes, MySQL y PhpMyAdmin

---

## 2. Tiempo de duración

Aproximadamente **60 minutos**

---

## 3. Fundamentos

### Docker y la contenerización
Docker es una plataforma de código abierto que permite crear, desplegar y ejecutar aplicaciones dentro de contenedores. Un **contenedor** es una unidad estándar de software que empaqueta el código y todas sus dependencias, de modo que la aplicación se ejecute de manera rápida y confiable en cualquier entorno. A diferencia de las máquinas virtuales, los contenedores comparten el kernel del sistema operativo anfitrión, lo que los hace mucho más ligeros y eficientes.

### Redes en Docker
Una **red Docker** permite que los contenedores se comuniquen entre sí de forma aislada del resto del sistema. Cuando dos contenedores están en la misma red personalizada (tipo `bridge`), pueden referenciarse mutuamente usando sus **nombres de contenedor** como hostname, sin necesidad de conocer sus direcciones IP. Esto es fundamental para que WordPress pueda conectarse a MySQL usando el nombre `mysql_db` como host de base de datos.

Los principales tipos de red en Docker son:
- **bridge** (por defecto): red privada interna en el host, ideal para comunicación entre contenedores en el mismo host.
- **host**: el contenedor comparte directamente la red del anfitrión.
- **none**: sin acceso a red.

### Volúmenes en Docker
Un **volumen Docker** es el mecanismo recomendado para persistir datos generados y usados por contenedores. A diferencia del sistema de archivos interno del contenedor (que se destruye cuando el contenedor se elimina), los volúmenes existen independientemente del ciclo de vida del contenedor. Esto significa que aunque se elimine y recree un contenedor, los datos almacenados en el volumen permanecen intactos.

Para esta práctica se utilizan dos volúmenes:
- **mysql-data**: persiste los archivos de la base de datos MySQL en `/var/lib/mysql`
- **wordpress-data**: persiste los archivos del sitio WordPress en `/var/www/html`

### WordPress, MySQL y PhpMyAdmin
- **WordPress** es el CMS (Sistema de Gestión de Contenidos) más popular del mundo, escrito en PHP y que usa MySQL como base de datos.
- **MySQL** es un sistema gestor de bases de datos relacional de código abierto, ampliamente usado en aplicaciones web.
- **PhpMyAdmin** es una herramienta web escrita en PHP que permite administrar bases de datos MySQL desde el navegador de forma visual e intuitiva.

La combinación de estos tres servicios representa una arquitectura web clásica y es la base de millones de sitios web en producción.

---

## 4. Conocimientos previos

Para realizar esta práctica el estudiante necesita tener claros los siguientes temas:

- Comandos básicos de Linux (WSL)
- Conceptos de contenedores e imágenes en Docker
- Comandos `docker run`, `docker network`, `docker volume`
- Variables de entorno en Docker (`-e`)
- Mapeo de puertos en Docker (`-p`)
- Conceptos básicos de bases de datos relacionales
- Navegación web para verificar resultados

---

## 5. Objetivos a alcanzar

- Crear una red personalizada en Docker para comunicar contenedores
- Crear volúmenes Docker para persistir datos de MySQL y WordPress
- Desplegar un contenedor MySQL con credenciales configuradas por variables de entorno
- Desplegar un contenedor PhpMyAdmin conectado a MySQL
- Desplegar un contenedor WordPress conectado a la red y a la base de datos
- Verificar el correcto funcionamiento del sitio desde el navegador

---

## 6. Equipo necesario

- Computador con sistema operativo Windows 10/11 con WSL2 habilitado
- Ubuntu WSL instalado (Ubuntu 20.04 o superior)
- Docker Desktop instalado y funcionando con integración WSL2 activa
- Docker versión 24.x o superior
- Navegador web
- Conexión a internet para descargar imágenes de Docker Hub

---

## 7. Material de apoyo

- Imagen oficial de WordPress en Docker Hub
- Imagen oficial de MySQL en Docker Hub
- Imagen oficial de PhpMyAdmin
- Guía de la asignatura

---

## 8. Procedimiento

### Paso 1: Crear la red personalizada de Docker

Abre la terminal WSL y ejecuta el siguiente comando para crear una red tipo `bridge` que permitirá la comunicación entre los tres contenedores:

<img width="783" height="57" alt="image" src="https://github.com/user-attachments/assets/c2532f60-be38-42e0-9c33-ca0f80aa2b78" />

---

### Paso 2: Crear los volúmenes para persistencia de datos

Crea el volumen para MySQL y WordPress:


<img width="619" height="99" alt="image" src="https://github.com/user-attachments/assets/f126e310-5dce-4d87-8fa0-22322fc1f44f" />

---

Verifica que ambos volúmenes existen:

```bash
docker volume ls
```
<img width="336" height="68" alt="image" src="https://github.com/user-attachments/assets/151849f5-ab21-405b-acac-54a64e18aa3e" />

---

### Paso 3: Crear el contenedor MySQL

Ejecuta el siguiente comando para crear el contenedor de base de datos con las credenciales necesarias y el volumen montado:

```bash
docker run -d \
  --name mysql_db \
  --network wordpress-red \
  -e MYSQL_ROOT_PASSWORD=rootpass123 \
  -e MYSQL_DATABASE=wordpress_db \
  -e MYSQL_USER=wp_user \
  -e MYSQL_PASSWORD=wp_pass123 \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

Las variables de entorno configuradas son:
| Variable | Valor | Descripción |
|---|---|---|
| MYSQL_ROOT_PASSWORD | rootpass123 | Contraseña del usuario root |
| MYSQL_DATABASE | wordpress_db | Base de datos que se crea automáticamente |
| MYSQL_USER | wp_user | Usuario para WordPress |
| MYSQL_PASSWORD | wp_pass123 | Contraseña del usuario WordPress |

---

### Paso 4: Crear el contenedor PhpMyAdmin

```bash
docker run -d \
  --name phpmyadmin \
  --network wordpress-red \
  -e PMA_HOST=mysql_db \
  -e PMA_USER=root \
  -e PMA_PASSWORD=rootpass123 \
  -p 8081:80 \
  phpmyadmin:latest
```

PhpMyAdmin quedará accesible en el puerto `8081` del anfitrión. La variable `PMA_HOST=mysql_db` indica el nombre del contenedor MySQL dentro de la red Docker.

---

### Paso 5: Crear el contenedor WordPress

```bash
docker run -d \
  --name wordpress_cms \
  --network wordpress-red \
  -e WORDPRESS_DB_HOST=mysql_db:3306 \
  -e WORDPRESS_DB_USER=wp_user \
  -e WORDPRESS_DB_PASSWORD=wp_pass123 \
  -e WORDPRESS_DB_NAME=wordpress_db \
  -v wordpress-data:/var/www/html \
  -p 8080:80 \
  wordpress:latest
```

WordPress quedará accesible en el puerto `8080`. La variable `WORDPRESS_DB_HOST=mysql_db:3306` indica que debe conectarse al contenedor MySQL por su nombre dentro de la red.

---

### Paso 6: Verificar que los tres contenedores están activos

```bash
docker ps
```
<img width="1901" height="125" alt="image" src="https://github.com/user-attachments/assets/350199a3-d6f3-4d64-be1c-8dd00fca4151" />

---

### Paso 7: Verificar la red Docker

```bash
docker network inspect wordpress-red
```
<img width="1320" height="537" alt="image" src="https://github.com/user-attachments/assets/bad2af0a-6db8-4531-916a-cf09c97439f0" />

Este comando muestra todos los contenedores conectados a la red `wordpress-red` y sus direcciones IP asignadas.

---

### Paso 8: Acceder a los servicios desde el navegador

Abre el navegador y accede a las siguientes URLs:

| Servicio | URL | Credenciales |
|---|---|---|
| WordPress | http://localhost:8080 | Configurar en el asistente |
| PhpMyAdmin | http://localhost:8081 | root / rootpass123 |

En WordPress, completa el asistente de instalación: selecciona el idioma, ingresa el nombre del sitio, usuario administrador y correo electrónico.
<img width="1919" height="1144" alt="image" src="https://github.com/user-attachments/assets/8e0c75e5-887e-4132-aa3e-888bfe235bd7" />

En PhpMyAdmin se podrá ver la base de datos `wordpress_db` creada automáticamente con todas las tablas generadas por WordPress.
<img width="1919" height="468" alt="image" src="https://github.com/user-attachments/assets/15809963-ec19-4e47-b702-80371178dd5a" />


---

## Figura 8-1. Diagrama de contenedores con puertos

```
┌─────────────────────────────────────────────────────────────────────┐
│                     SERVIDOR ANFITRIÓN (WSL)                        │
│                                                                     │
│  Puerto 8080          Puerto 8081                                   │
│      │                    │                                         │
│      ▼                    ▼                                         │
│  ┌─────────────┐    ┌─────────────┐                                 │
│  │ wordpress   │    │ phpmyadmin  │                                 │
│  │    _cms     │    │             │                                 │
│  │  :80 (HTTP) │    │  :80 (HTTP) │                                 │
│  │             │    │             │                                 │
│  │  Vol:       │    │  PMA_HOST=  │                                 │
│  │  wordpress  │    │  mysql_db   │                                 │
│  │  -data      │    │             │                                 │
│  └──────┬──────┘    └──────┬──────┘                                 │
│         │                  │                                        │
│         └────────┬─────────┘                                        │
│                  │                                                  │
│          ┌───────▼────────┐                                         │
│          │  wordpress-red │  (Docker Network - bridge)              │
│          └───────┬────────┘                                         │
│                  │                                                  │
│          ┌───────▼────────┐                                         │
│          │   mysql_db     │                                         │
│          │  MySQL 8.0     │                                         │
│          │  Puerto: 3306  │                                         │
│          │                │                                         │
│          │  Vol:          │                                         │
│          │  mysql-data    │                                         │
│          └────────────────┘                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9. Resultados esperados

Al finalizar la práctica se espera:

1. Los tres contenedores (`mysql_db`, `phpmyadmin`, `wordpress_cms`) aparecen con estado **Up** al ejecutar `docker ps`.
2. Accediendo a `http://localhost:8080` se muestra el asistente de instalación de WordPress o el sitio ya configurado.
3. Accediendo a `http://localhost:8081` se puede ingresar a PhpMyAdmin con las credenciales `root / rootpass123` y visualizar la base de datos `wordpress_db` con todas las tablas de WordPress.
4. Al detener y volver a crear los contenedores, los datos persisten gracias a los volúmenes `mysql-data` y `wordpress-data`.
5. La red `wordpress-red` conecta los tres contenedores y permite su comunicación interna por nombre.

---

## 10. Bibliografía

- Docker Inc. (2024). *Docker Documentation — Networking overview*. https://docs.docker.com/network/
- Docker Inc. (2024). *Docker Documentation — Volumes*. https://docs.docker.com/storage/volumes/
- Docker Hub. (2024). *wordpress — Official Docker Image*. https://hub.docker.com/_/wordpress
- Docker Hub. (2024). *mysql — Official Docker Image*. https://hub.docker.com/_/mysql
- Docker Hub. (2024). *phpmyadmin — Official Docker Image*. https://hub.docker.com/_/phpmyadmin
- WordPress Foundation. (2024). *WordPress.org — Documentation*. https://wordpress.org/documentation/
- Daniela, A. (2026). Despliegue de WordPress con Docker (Red y Volúmenes).
