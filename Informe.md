

## 1. Título

Implementación y comunicación de contenedores MySQL y phpMyAdmin mediante una red personalizada en Docker.

## 2. Tiempo de duración

2 horas.

## 3. Fundamentos

Docker permite ejecutar aplicaciones en contenedores que incluyen las dependencias necesarias para su funcionamiento. Los contenedores se crean a partir de imágenes y permiten ejecutar servicios independientes dentro de un mismo equipo.

MySQL es un sistema gestor de bases de datos relacionales que permite almacenar, organizar y consultar información mediante instrucciones SQL. phpMyAdmin es una herramienta web que facilita la administración de MySQL mediante una interfaz gráfica.

Para establecer la comunicación entre ambos servicios se puede utilizar una red personalizada de Docker. Las redes de tipo `bridge` permiten conectar contenedores ejecutados en un mismo motor de Docker y ofrecen resolución de nombres. Así, phpMyAdmin puede encontrar el servidor MySQL mediante el nombre de su contenedor.

Las variables de entorno permiten configurar aspectos como las credenciales de acceso, el servidor de destino y el puerto de conexión. La publicación de puertos permite acceder desde el equipo anfitrión a los servicios de un contenedor.

En esta práctica se publicó phpMyAdmin en el puerto local `8080`, mientras que la conexión interna con MySQL se realizó mediante el puerto `3306`.

El comando `docker network inspect` permite consultar los detalles de una red y sus contenedores conectados. La creación y consulta de datos desde phpMyAdmin permite comprobar, adicionalmente, que la comunicación con MySQL funciona correctamente.

## 4. Conocimientos previos

- Conceptos básicos de imágenes y contenedores Docker.
- Manejo básico de Docker Desktop.
- Ejecución de comandos en PowerShell.
- Conceptos de redes, direcciones IP y puertos.
- Conceptos básicos de bases de datos relacionales.
- Uso de instrucciones SQL como `CREATE TABLE`, `INSERT` y `SELECT`.

## 5. Objetivos a alcanzar

**Objetivo general**

Implementar dos contenedores, uno para MySQL y otro para phpMyAdmin, y comprobar su comunicación mediante una red personalizada en Docker.

**Objetivos específicos**

- Crear un contenedor MySQL y configurar sus credenciales.
- Crear un contenedor phpMyAdmin y establecer sus parámetros de conexión.
- Crear una red personalizada y conectar ambos contenedores.
- Verificar la pertenencia de los contenedores a la red.
- Crear una base de datos de prueba desde phpMyAdmin.
- Comprobar la comunicación mediante la creación de una tabla, la inserción de registros y su consulta.

## 6. Equipo necesario

- Computadora con sistema operativo Windows.
- Docker Desktop instalado y en ejecución.
- Terminal PowerShell.
- Navegador web.


## 7. Material de apoyo

- Enunciado de la práctica.
- Documentación oficial de Docker.
- Imágenes oficiales de MySQL y phpMyAdmin.


## 8. Procedimiento

**Paso 1: Creación del contenedor MySQL.**

Se ejecutó el siguiente comando:

```powershell
docker run -d --name mysql_practica -e MYSQL_ROOT_PASSWORD=admin -e MYSQL_ROOT_HOST=% mysql:8.4
```

La opción `-d` ejecutó el contenedor en segundo plano y `--name` le asignó el nombre `mysql_practica`.

La variable `MYSQL_ROOT_PASSWORD` estableció la contraseña del usuario `root`. Para esta práctica local, `MYSQL_ROOT_HOST=%` permitió utilizar esa cuenta desde otro contenedor.

<img width="1426" height="425" alt="Captura de pantalla 2026-10-31 142941" src="https://github.com/user-attachments/assets/6619e743-15f0-4a51-a40c-db79b3fc9a26" />


**Paso 2: Creación del contenedor phpMyAdmin.**

Se creó el contenedor de administración web:

```powershell
docker run -d --name phpmyadmin_practica -e PMA_HOST=mysql_practica -e PMA_PORT=3306 -p 127.0.0.1:8080:80 phpmyadmin
```

<img width="1506" height="671" alt="Captura de pantalla 2026-10-31 143320" src="https://github.com/user-attachments/assets/2a145037-e7ab-4d0f-b7d2-9e25e90e2792" />


**Paso 3: Creación de la red personalizada.**

Se creó la red `red_practica`:

```powershell
docker network create red_practica
```

Posteriormente, se verificó su existencia:

```powershell
docker network ls
```

El resultado mostró `red_practica` con el controlador `bridge` y alcance `local`.

<img width="968" height="193" alt="Captura de pantalla 2026-10-31 143437" src="https://github.com/user-attachments/assets/51fa5599-5cdf-4a1f-9d44-4e26b0c73562" />


**Paso 4: Conexión de los contenedores a la red.**

Se conectó MySQL a la red:

```powershell
docker network connect red_practica mysql_practica
```

Luego se conectó phpMyAdmin:

```powershell
docker network connect red_practica phpmyadmin_practica
```

<img width="1251" height="965" alt="Captura de pantalla 2026-10-31 143616" src="https://github.com/user-attachments/assets/264672af-2533-47d1-9dbe-96807d2918f2" />

**Paso 5: Creación de la base de datos de prueba.**

Dentro de phpMyAdmin se seleccionó **Nueva** o **Bases de datos**. Se ingresó el nombre `base_prueba`, se seleccionó el cotejamiento `utf8mb4_unicode_ci` y se pulsó **Crear**.

<img width="487" height="312" alt="Captura de pantalla 2026-10-31 144023" src="https://github.com/user-attachments/assets/1735ee24-5669-48ab-a6f5-10e9b0dc44a1" />

**Paso 6: Creación de la tabla de prueba.**

Se seleccionó `base_prueba`, se abrió la pestaña **SQL** y se ejecutó:

```sql
USE base_prueba;

CREATE TABLE estudiantes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    correo VARCHAR(150) NOT NULL,
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

<img width="512" height="327" alt="Captura de pantalla 2026-10-31 144630" src="https://github.com/user-attachments/assets/f487bd89-61df-4f93-8cb3-ab0d2eb27caa" />


**Paso 7: Consulta de registros.**

<img width="945" height="185" alt="Captura de pantalla 2026-10-31 144642" src="https://github.com/user-attachments/assets/d343ddef-0ee6-460b-89db-8e25e7c360e9" />



## 9. Resultados esperados

- Contenedor `mysql_practica` creado y en ejecución.
- Contenedor `phpmyadmin_practica` creado y accesible desde el navegador.
- Red personalizada `red_practica` disponible.
- Ambos contenedores conectados a la misma red.
- Conexión de phpMyAdmin con MySQL mediante el nombre `mysql_practica`.
- Base de datos `base_prueba` creada.
- Tabla `estudiantes` creada.
- Registros insertados y consultados correctamente.


## 10. Bibliografía

1. Docker Official Images. (s. f.). *MySQL*. Docker Hub.  
   https://hub.docker.com/_/mysql

2. phpMyAdmin. (s. f.). *phpMyAdmin: Docker Official Image*. Docker Hub.  
   https://hub.docker.com/_/phpmyadmin

3. Docker. (s. f.). *Bridge network driver*. Docker Docs.  
   https://docs.docker.com/engine/network/drivers/bridge/

4. Docker. (s. f.). *docker network inspect*. Docker Docs.  
   https://docs.docker.com/reference/cli/docker/network/inspect/
