# Oauth2 en Canvas LMS

## 1.- Crear las credenciales
Las claves de acceso se crean en `Admin > Claves de Desarrollador`.
Una vez ahí deberá agregar una nueva **Clave API**, desde el apartado ubicado en la esquina superior derecha.

Se abrirá un formulario que debe completar, lo unico que debe entender acá es que la URI de redirección hace referencia al sitio externo que recibirá las claves de acceso creadas por canvas, si está utilizando un servidor local puede utilizar `localhost`

Una vez ha creado su credencial, asegurece de cambiar el estado a activo y **observe cuidadosamente la columna de detalles, ahí se encuentran las claves de integración y será vital para seguir con los siguientes pasos.** 




## 2.- Obtener el codigo de autorización del usuario
Para obtener el codigo de autorización debemos consumir la siguiente uri de canvas.

Verbo HTTP: `GET`

URI: `/login/oauth2/auth`

PARAMETROS `client_id, response_type, redirect_uri`

**client_id**:  Es el primer numero visible en la columna de detalles -*Columna mencionada en el paso 1*-, tiene un formato tipo `143540000000000035`. 

**response_type**: Solo tiene un valor posible y es `code`

**redirect_uri**: Es la ruta a la cual canvas devolverá por variables `GET` el codigo de autorización o la notificación de rechazo del usuario.

La url final deberia verse algo así: 
`https://<DOMINIO DE TU INSTANCIA DE CANVAS>/login/oauth2/auth?client_id=143540000000000035&response_type=code&redirect_uri=http://localhost/oauth2/oauth2_token.php`

El end point puede retornar dos posibles respuestas que son:
1) `[error] => access_denied`

2) `[code] => a4e9bbe90461898ff91ee0537e969e1c2248666df256bc36a5c053bf5d7ddbcc90922ba7ca108ef6be315cf5350f4d9acc781b21b5eadfca4a74b7ca6e209329`
   
   >Se asume que si viene el parametro `code`, el usuario ha autorizado a nuestra aplicación.

## 3.- Cambiar el codigo de autorización por el token de acceso final
Ahora utilizaremos el codigo de autorización recibido en la petición anterior para obtener un token de acceso final.
>El codigo solo será valido para una unica petición

Verbo HTTP: `POST`

URI: `/login/oauth2/token`

PARAMETROS `client_id, client_secret, code`

**client_id**:  Es el primer numero visible en la columna de detalles -*Columna mencionada en el paso 1*-, tiene un formato tipo `143540000000000035`. 

**client_secret**: Es la clave que se muestra al hacer clic en el botón **Mostrar Clave**, ubicado bajo el client_id -*Columna mencionada en el paso 1*-.

**code**: Es el codigo devuelto por canvas en el paso anterior

La respuesta que nos dará canvas será:

`{
    "access_token": "14354~DaO7ADOYLY1hGHHzJecJzNXN2KUlS3Ka0HM7sn2W7xA0wNl2v9iF0SUBXU6V97jp",
    "token_type": "Bearer",
    "user": {
        "id": 121,
        "name": "Cesar Mora",
        "global_id": "143540000000000121",
        "effective_locale": "es"
    },
    "refresh_token": "14354~qvYGnQ2Y6SWSWmNaWzcp02idIlDnZUuE3L1OkWDQFOWW9F1isCvHAStOVZjFlItw",
    "expires_in": 3600
}`

En caso de intentar obtener un nuevo token de acceso final con los mismos valores será: 

`{
    "error": "invalid_grant",
    "error_description": "authorization_code not found"
}`

Si algo sale mal acá, deberá solicitar la autorización nuevamente.

## 4.- Consumir end points de la API
Si haz llegado a este punto, ya tienes lo necesario para comenzar a consumir la API de canvas.
Debes tener en cuenta que todas las peticiones deben llevar el Bearer Token en los headers.

**Header de la autorización:**
`Authorization: Bearer <token>`

Donde `<token>` corresponde a la propiedad `access_token` en la respuesta del paso anterior.
