## 1. Título

Implementación y comunicación de contenedores MySQL y phpMyAdmin mediante una red personalizada en Docker.

## 2. Tiempo de duración

2 horas.

## 3. Fundamentos

Docker permite ejecutar aplicaciones en contenedores, proporcionando entornos que incluyen las dependencias necesarias para su funcionamiento. Los contenedores se crean a partir de imágenes y pueden ejecutar servicios independientes, como una base de datos o una aplicación web.

MySQL es un sistema gestor de bases de datos relacionales que permite almacenar, organizar y consultar información mediante SQL. Por su parte, phpMyAdmin es una herramienta web que facilita la administración de bases de datos MySQL mediante una interfaz gráfica.

Para que ambos servicios trabajen conjuntamente, necesitan comunicarse a través de una red. Las redes personalizadas de tipo `bridge` permiten conectar contenedores ejecutados en un mismo motor de Docker y ofrecen resolución de nombres. Esto permite que phpMyAdmin encuentre el servidor MySQL utilizando el nombre del contenedor, sin configurar manualmente su dirección IP.

Las variables de entorno permiten establecer parámetros de configuración durante la creación de los contenedores. En esta práctica se utilizaron para definir la contraseña de MySQL y configurar el servidor y el puerto al que se conectaría phpMyAdmin.

La publicación de puertos permite acceder a un servicio del contenedor desde el equipo anfitrión. Se utilizó el puerto local `8080` para acceder al puerto `80` del contenedor de phpMyAdmin. MySQL utilizó su puerto interno `3306` para la comunicación entre los contenedores.

Finalmente, el comando `docker network inspect` permite consultar los detalles de una red y los contenedores conectados. Sin embargo, pertenecer a la misma red no garantiza por sí solo el funcionamiento del servicio; también es necesario comprobar la autenticación y ejecutar operaciones sobre la base de datos.

## 4. Conocimientos previos

- Conceptos básicos de imágenes y contenedores Docker.
- Uso de comandos en PowerShell.
- Manejo básico de Docker Desktop.
- Conceptos de redes, direcciones IP y puertos.
- Configuración mediante variables de entorno.
- Conceptos básicos de bases de datos relacionales.
- Uso de instrucciones SQL como `CREATE TABLE`, `INSERT` y `SELECT`.

## 5. Objetivos a alcanzar

**Objetivo general**

Implementar dos contenedores, uno para MySQL y otro para phpMyAdmin, y establecer su comunicación mediante una red personalizada en Docker.

**Objetivos específicos**

- Crear un contenedor MySQL y configurar sus credenciales de acceso.
- Crear un contenedor phpMyAdmin y establecer sus parámetros de conexión.
- Crear una red personalizada y conectar ambos contenedores.
- Verificar la conexión de los contenedores a la red.
- Crear una base de datos de prueba desde phpMyAdmin.
- Comprobar la comunicación mediante la creación de una tabla, la inserción de registros y su consulta.

## 6. Equipo necesario

- Computadora con sistema operativo Windows.
- Docker Desktop instalado y en ejecución.
- Entorno de Docker configurado para ejecutar contenedores Linux.
- Terminal PowerShell.
- Navegador web.
- Conexión a Internet para descargar las imágenes.
- Memoria y almacenamiento suficientes para ejecutar ambos contenedores.

## 7. Material de apoyo

- Enunciado de la práctica.
- Documentación oficial de Docker.
- Documentación de las imágenes oficiales de MySQL y phpMyAdmin.
- Documentación sobre redes personalizadas en Docker.
- Capturas de pantalla del procedimiento realizado.

## 8. Procedimiento

<!-- Para visualizar las imágenes en GitHub, subir las ocho capturas a la misma carpeta que este archivo Markdown, conservando sus nombres originales. -->

**Paso 1: Verificación del funcionamiento de Docker.**

Se inició Docker Desktop y se comprobó la disponibilidad del cliente y del servidor mediante:

```powershell
docker version
```

También se consultaron los contenedores existentes:

```powershell
docker ps -a
```

Este último comando permite visualizar tanto los contenedores en ejecución como los detenidos.

**Paso 2: Creación del contenedor MySQL.**

Se ejecutó el siguiente comando para crear el contenedor `mysql_practica`:

```powershell
docker run -d --name mysql_practica -e MYSQL_ROOT_PASSWORD=admin -e MYSQL_ROOT_HOST=% mysql:8.4
```

Los parámetros utilizados fueron:

| Parámetro | Función |
|---|---|
| `-d` | Ejecuta el contenedor en segundo plano. |
| `--name mysql_practica` | Asigna un nombre al contenedor. |
| `MYSQL_ROOT_PASSWORD=admin` | Establece la contraseña del usuario `root`. |
| `MYSQL_ROOT_HOST=%` | Permite la conexión de `root` desde otro contenedor. |
| `mysql:8.4` | Indica la imagen y versión utilizada. |

La contraseña `admin` corresponde únicamente al entorno local de esta práctica.

Como la imagen no estaba disponible localmente, Docker inició su descarga.

![Descarga de la imagen MySQL](<Captura de pantalla 2026-10-31 142941.png>)

*Figura 1. Comando de creación del contenedor MySQL y descarga de la imagen.*

Se consultaron los registros para comprobar la inicialización:

```powershell
docker logs mysql_practica
```

Se esperó a que MySQL estuviera listo para aceptar conexiones. Aunque la figura muestra la descarga en curso, el acceso posterior desde phpMyAdmin permitió comprobar que el servicio terminó funcionando.

**Paso 3: Creación del contenedor phpMyAdmin.**

Se creó el contenedor `phpmyadmin_practica`:

```powershell
docker run -d --name phpmyadmin_practica -e PMA_HOST=mysql_practica -e PMA_PORT=3306 -p 127.0.0.1:8080:80 phpmyadmin
```

Los parámetros de conexión fueron:

| Parámetro | Función |
|---|---|
| `PMA_HOST=mysql_practica` | Define el nombre del contenedor que ejecuta MySQL. |
| `PMA_PORT=3306` | Establece el puerto interno de MySQL. |
| `-p 127.0.0.1:8080:80` | Publica el servicio web en el puerto `8080` del equipo local. |
| `phpmyadmin` | Indica la imagen oficial utilizada. |

Las credenciales de MySQL se introdujeron posteriormente en la pantalla de inicio de sesión de phpMyAdmin.

![Creación del contenedor phpMyAdmin](<Captura de pantalla 2026-10-31 143320.png>)

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

El resultado mostró la red con el controlador `bridge` y alcance `local`.

![Listado de redes Docker](<Captura de pantalla 2026-10-31 143437.png>)

*Figura 3. Verificación de la existencia de `red_practica`.*

**Paso 5: Conexión de ambos contenedores a la red.**

Se conectó el contenedor MySQL:

```powershell
docker network connect red_practica mysql_practica
```

Luego se conectó el contenedor phpMyAdmin:

```powershell
docker network connect red_practica phpmyadmin_practica
```

Estos comandos permitieron incorporar los contenedores a la red sin eliminarlos ni crearlos nuevamente.

**Paso 6: Verificación de los contenedores conectados.**

Se inspeccionó la configuración de la red:

```powershell
docker network inspect red_practica
```

En la sección `Containers` aparecieron `mysql_practica` y `phpmyadmin_practica`.

También se identificaron los siguientes datos:

| Elemento | Valor observado |
|---|---|
| Nombre de la red | `red_practica` |
| Controlador | `bridge` |
| Subred | `172.20.0.0/16` |
| Puerta de enlace | `172.20.0.1` |
| Dirección de MySQL | `172.20.0.2/16` |

La dirección IPv4 de phpMyAdmin queda fuera del recorte de la captura, pero su nombre sí aparece entre los contenedores conectados.

![Conexión e inspección de la red](<Captura de pantalla 2026-10-31 143616.png>)

*Figura 4. Comandos de conexión e inspección de la red personalizada.*

Para consultar los contenedores en ejecución se utilizó:

```powershell
docker ps
```

**Paso 7: Acceso a phpMyAdmin.**

Desde el navegador del equipo donde se ejecutaba Docker se ingresó a:

```text
http://localhost:8080
```

Se inició sesión con las credenciales configuradas:

| Campo | Valor |
|---|---|
| Usuario | `root` |
| Contraseña de la práctica | `admin` |

El servidor MySQL ya estaba definido mediante la variable `PMA_HOST`, por lo que phpMyAdmin dirigió la conexión a `mysql_practica:3306`.

**Paso 8: Creación de la base de datos de prueba.**

Dentro de phpMyAdmin se seleccionó la opción **Nueva** o **Bases de datos**. Se ingresó el nombre `base_prueba`, se seleccionó el cotejamiento `utf8mb4_unicode_ci` y se pulsó **Crear**.

La base apareció en el panel izquierdo de la interfaz.

![Base de datos creada](<Captura de pantalla 2026-10-31 144137.png>)

*Figura 5. Base de datos `base_prueba` seleccionada en phpMyAdmin.*

Para comprobar su existencia se ejecutó:

```sql
SHOW DATABASES;
```

![Consulta de las bases de datos](<Captura de pantalla 2026-10-31 144023.png>)

*Figura 6. Resultado de `SHOW DATABASES`, incluyendo `base_prueba`.*

La creación de la base desde phpMyAdmin permitió comprobar que la autenticación y la comunicación con MySQL funcionaban correctamente.

**Paso 9: Creación de la tabla de prueba.**

Se seleccionó `base_prueba`, se abrió la pestaña **SQL** y se preparó la siguiente instrucción:

```sql
USE base_prueba;

CREATE TABLE estudiantes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    correo VARCHAR(150) NOT NULL,
    fecha_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

La tabla se definió con los siguientes campos:

| Campo | Descripción |
|---|---|
| `id` | Identificador autoincremental y clave primaria. |
| `nombre` | Nombre del estudiante. |
| `correo` | Correo electrónico del estudiante. |
| `fecha_registro` | Fecha y hora de inserción del registro. |

La captura presenta conjuntamente las instrucciones de creación, inserción y consulta utilizadas en los pasos 9 y 10.

![Instrucciones SQL de prueba](<Captura de pantalla 2026-10-31 144630.png>)

*Figura 7. Instrucciones SQL para crear la tabla, insertar registros y consultar los datos.*

**Paso 10: Inserción y consulta de registros.**

Se insertaron dos registros:

```sql
INSERT INTO estudiantes (nombre, correo)
VALUES
    ('Gabriel', 'gabriel@example.com'),
    ('Ana', 'ana@example.com');
```

Posteriormente, se consultó el contenido de la tabla:

```sql
SELECT * FROM estudiantes;
```

Los resultados mostraron los registros de Gabriel y Ana, junto con sus identificadores, correos y fechas de registro.

![Registros de la tabla estudiantes](<Captura de pantalla 2026-10-31 144642.png>)

*Figura 8. Visualización de los registros almacenados en `estudiantes`.*

Esta operación comprobó que phpMyAdmin podía escribir y consultar información en MySQL mediante la red personalizada.

**Paso 11: Comprobación final de la comunicación.**

Se relacionaron dos evidencias:

- La presencia de ambos contenedores en la sección `Containers` de `red_practica`.
- La creación de la base de datos, la tabla y los registros desde phpMyAdmin.

La inspección confirmó la pertenencia de los contenedores a la red. Las operaciones SQL comprobaron la comunicación funcional entre los servicios y la validez de las credenciales.

## 9. Resultados esperados

- Contenedor `mysql_practica` creado y en ejecución.
- Contenedor `phpmyadmin_practica` creado y accesible desde el navegador.
- Red personalizada `red_practica` disponible.
- Ambos contenedores conectados a la misma red.
- Conexión de phpMyAdmin con MySQL mediante el nombre `mysql_practica`.
- Autenticación satisfactoria con las credenciales configuradas.
- Base de datos `base_prueba` creada.
- Tabla `estudiantes` creada con los campos definidos.
- Registros de Gabriel y Ana insertados y consultados correctamente.

Las evidencias obtenidas muestran el cumplimiento del objetivo principal: establecer la comunicación entre MySQL y phpMyAdmin mediante una red personalizada en Docker. La creación y consulta de datos permitió verificar el funcionamiento conjunto de ambos servicios.

## 10. Bibliografía

1. Docker Official Images. (s. f.). *MySQL*. Docker Hub.  
   https://hub.docker.com/_/mysql

2. phpMyAdmin. (s. f.). *phpMyAdmin: Docker Official Image*. Docker Hub.  
   https://hub.docker.com/_/phpmyadmin

3. Docker. (s. f.). *Bridge network driver*. Docker Docs.  
   https://docs.docker.com/engine/network/drivers/bridge/

4. Docker. (s. f.). *docker network inspect*. Docker Docs.  
   https://docs.docker.com/reference/cli/docker/network/inspect/
