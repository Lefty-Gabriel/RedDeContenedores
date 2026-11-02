<!--
Sube las ocho capturas a la misma carpeta del repositorio donde guardarás este informe, conservando sus nombres.
Las etiquetas <img> incluidas debajo las mostrarán en GitHub.

Si prefieres usar enlaces https://github.com/user-attachments/assets/..., arrastra cada captura al editor de GitHub y sustituye su etiqueta por la que GitHub genere. Esos enlaces se crean al subir las imágenes; no se pueden deducir de sus nombres.
-->

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
- Configuración mediante variables de entorno.
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
- Docker configurado para ejecutar contenedores Linux.
- Terminal PowerShell.
- Navegador web.
- Conexión a Internet para descargar las imágenes.
- Memoria y almacenamiento suficientes para ejecutar ambos servicios.

## 7. Material de apoyo

- Enunciado de la práctica.
- Documentación oficial de Docker.
- Documentación de las imágenes oficiales de MySQL y phpMyAdmin.
- Documentación sobre redes personalizadas.
- Capturas de pantalla del procedimiento.

## 8. Procedimiento

**Paso 1: Verificación del funcionamiento de Docker.**

Se inició Docker Desktop y se comprobó la disponibilidad del cliente y del servidor mediante:

```powershell
docker version
```

También se consultaron los contenedores existentes:

```powershell
docker ps -a
```

Este comando permite visualizar los contenedores en ejecución y los detenidos.

**Paso 2: Creación del contenedor MySQL.**

Se ejecutó el siguiente comando:

```powershell
docker run -d --name mysql_practica -e MYSQL_ROOT_PASSWORD=admin -e MYSQL_ROOT_HOST=% mysql:8.4
```

La opción `-d` ejecutó el contenedor en segundo plano y `--name` le asignó el nombre `mysql_practica`.

La variable `MYSQL_ROOT_PASSWORD` estableció la contraseña del usuario `root`. Para esta práctica local, `MYSQL_ROOT_HOST=%` permitió utilizar esa cuenta desde otro contenedor.

La contraseña `admin` se utilizó únicamente en el entorno de aprendizaje.

Como la imagen no estaba disponible localmente, Docker inició su descarga:

<img width="1000" alt="Creación del contenedor mysql_practica y descarga de MySQL 8.4" src="Captura de pantalla 2026-10-31 142941.png" />

*Figura 1. Comando de creación del contenedor MySQL y descarga de la imagen.*

Se consultaron los registros de inicialización:

```powershell
docker logs mysql_practica
```

Se esperó a que MySQL estuviera listo para aceptar conexiones. La captura muestra la descarga en curso; el acceso posterior desde phpMyAdmin confirmó que el servicio terminó funcionando.

**Paso 3: Creación del contenedor phpMyAdmin.**

Se creó el contenedor de administración web:

```powershell
docker run -d --name phpmyadmin_practica -e PMA_HOST=mysql_practica -e PMA_PORT=3306 -p 127.0.0.1:8080:80 phpmyadmin
```

La variable `PMA_HOST` indicó el nombre del contenedor MySQL y `PMA_PORT` estableció el puerto interno `3306`.

La opción `-p 127.0.0.1:8080:80` permitió acceder al puerto `80` del contenedor mediante el puerto `8080` del equipo local.

<img width="1000" alt="Descarga de la imagen y creación del contenedor phpmyadmin_practica" src="Captura de pantalla 2026-10-31 143320.png" />

*Figura 2. Descarga de la imagen y creación del contenedor phpMyAdmin.*

**Paso 4: Creación de la red personalizada.**

Se creó la red `red_practica`:

```powershell
docker network create red_practica
```

Posteriormente, se verificó su existencia:

```powershell
docker network ls
```

El resultado mostró `red_practica` con el controlador `bridge` y alcance `local`.

<img width="1000" alt="Listado de redes Docker con red_practica" src="Captura de pantalla 2026-10-31 143437.png" />

*Figura 3. Verificación de la existencia de la red personalizada.*

**Paso 5: Conexión de los contenedores a la red.**

Se conectó MySQL a la red:

```powershell
docker network connect red_practica mysql_practica
```

Luego se conectó phpMyAdmin:

```powershell
docker network connect red_practica phpmyadmin_practica
```

Estos comandos permitieron incorporar ambos contenedores a la red sin eliminarlos ni crearlos nuevamente.

**Paso 6: Verificación de la conexión a la red.**

Se inspeccionó la configuración:

```powershell
docker network inspect red_practica
```

En la sección `Containers` aparecieron `mysql_practica` y `phpmyadmin_practica`.

La salida también mostró la subred `172.20.0.0/16`, la puerta de enlace `172.20.0.1` y la dirección `172.20.0.2/16` del contenedor MySQL.

La dirección IPv4 de phpMyAdmin queda fuera del recorte de la captura, pero su nombre aparece entre los contenedores conectados.

<img width="1000" alt="Conexión de los contenedores e inspección de red_practica" src="Captura de pantalla 2026-10-31 143616.png" />

*Figura 4. Comandos de conexión e inspección de la red, mostrando ambos contenedores.*

También se utilizó el siguiente comando para consultar los contenedores en ejecución:

```powershell
docker ps
```

**Paso 7: Acceso a phpMyAdmin.**

Desde el navegador del equipo donde se ejecutaba Docker se ingresó a:

```text
http://localhost:8080
```

Se inició sesión con el usuario `root` y la contraseña `admin`, establecidos durante la configuración de MySQL.

El servidor de destino ya estaba definido mediante `PMA_HOST`, por lo que phpMyAdmin dirigió la conexión a `mysql_practica:3306`.

**Paso 8: Creación de la base de datos de prueba.**

Dentro de phpMyAdmin se seleccionó **Nueva** o **Bases de datos**. Se ingresó el nombre `base_prueba`, se seleccionó el cotejamiento `utf8mb4_unicode_ci` y se pulsó **Crear**.

La base apareció en el panel izquierdo de la interfaz.

<img width="761" alt="Base de datos base_prueba creada y seleccionada en phpMyAdmin" src="Captura de pantalla 2026-10-31 144137.png" />

*Figura 5. Base de datos creada, antes de incorporar tablas.*

Se comprobó su existencia mediante:

```sql
SHOW DATABASES;
```

<img width="487" alt="Resultado de SHOW DATABASES con base_prueba" src="Captura de pantalla 2026-10-31 144023.png" />

*Figura 6. Consulta de las bases de datos disponibles.*

Esta operación confirmó que phpMyAdmin podía autenticarse y ejecutar instrucciones sobre MySQL.

**Paso 9: Creación de la tabla de prueba.**

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

El campo `id` se configuró como clave primaria autoincremental. Los campos `nombre` y `correo` almacenaron los datos de cada estudiante, mientras que `fecha_registro` permitió guardar automáticamente la fecha y hora de inserción.

La siguiente captura presenta conjuntamente las instrucciones de creación, inserción y consulta:

<img width="512" alt="Código SQL para crear estudiantes, insertar registros y consultarlos" src="Captura de pantalla 2026-10-31 144630.png" />

*Figura 7. Instrucciones SQL utilizadas para comprobar el funcionamiento de la base de datos.*

**Paso 10: Inserción y consulta de registros.**

Se insertaron dos registros:

```sql
INSERT INTO estudiantes (nombre, correo)
VALUES
    ('Gabriel', 'gabriel@example.com'),
    ('Ana', 'ana@example.com');
```

Posteriormente, se consultaron los datos:

```sql
SELECT * FROM estudiantes;
```

El resultado mostró los registros de Gabriel y Ana, con sus identificadores, correos y fechas de registro.

<img width="945" alt="Registros de Gabriel y Ana almacenados en la tabla estudiantes" src="Captura de pantalla 2026-10-31 144642.png" />

*Figura 8. Resultado de la consulta de los registros almacenados.*

La inserción y consulta comprobaron que phpMyAdmin podía escribir y leer información en MySQL mediante la red personalizada.

**Paso 11: Comprobación final de la comunicación.**

Se relacionaron las siguientes evidencias:

- Ambos contenedores aparecieron en la sección `Containers` de `red_practica`.
- Se accedió correctamente a phpMyAdmin.
- Se creó la base de datos `base_prueba`.
- Se creó la tabla `estudiantes`.
- Se insertaron y consultaron los registros de prueba.

La inspección confirmó la pertenencia a la red. Las operaciones realizadas desde phpMyAdmin comprobaron la comunicación funcional con MySQL y la validez de las credenciales.

## 9. Resultados esperados

- Contenedor `mysql_practica` creado y en ejecución.
- Contenedor `phpmyadmin_practica` creado y accesible desde el navegador.
- Red personalizada `red_practica` disponible.
- Ambos contenedores conectados a la misma red.
- Conexión de phpMyAdmin con MySQL mediante el nombre `mysql_practica`.
- Autenticación satisfactoria con las credenciales configuradas.
- Base de datos `base_prueba` creada.
- Tabla `estudiantes` creada.
- Registros insertados y consultados correctamente.

Las evidencias obtenidas muestran el cumplimiento del objetivo principal: establecer la comunicación entre MySQL y phpMyAdmin mediante una red personalizada en Docker.

La creación de la base de datos, la tabla y los registros permitió comprobar el funcionamiento conjunto de ambos servicios.

## 10. Bibliografía

1. Docker Official Images. (s. f.). *MySQL*. Docker Hub.  
   https://hub.docker.com/_/mysql

2. phpMyAdmin. (s. f.). *phpMyAdmin: Docker Official Image*. Docker Hub.  
   https://hub.docker.com/_/phpmyadmin

3. Docker. (s. f.). *Bridge network driver*. Docker Docs.  
   https://docs.docker.com/engine/network/drivers/bridge/

4. Docker. (s. f.). *docker network inspect*. Docker Docs.  
   https://docs.docker.com/reference/cli/docker/network/inspect/
