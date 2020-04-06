# Correo Electrónico

Para el envío de correos electrónicos no necesitaremos realizar la configuración de un servidor de correos, sino que Laravel provee una API que nos servirá para conectarnos con proveedores de correo. Los servicios a los que nos podemos conectar son Mailgun, Postmark, Amazon SES, así como SMTP y sendmail.

Adicionalmente, para realizar pruebas durante el desarrollo nos podemos contectar al servicio de Mailtrap, el cual provee un sistema donde podemos ver todos los correos electrónicos que se envían desde nuestro sistema.

## Prerequisitos

Para poder utilizar los controladores de conexión debemos instalar la librería *Guzzle HTTP* mediante el comando:

`composer require guzzlehttp/guzzle`

Adicionalmente, cada servicio tiene algunas configuraciones que debemos seguir de la documentación.

## Verificación de correo

Laravel provee un sistema de verificación de correo electrónico para usuarios recién registrados.

La migración para la tabla *users* incluida con Laravel incluye la columna `email_verified_at` la cual tiene como función registrar si el correo del usuario ya fue verificado.

Así mismo se tienen rutas y vistas para propósitos de validación , las cuales se habilitan en el archivo *web* de la siguiente forma:

`Auth::routes(['verify' => true]);`

Lo anterior habilita el mecanismo de validación de correo electrónico para usuarios recién registrados, sin embargo aún es necesario que indiquemos las rutas que requerirán que el correo haya sido verificado, para lo cual se tiene el middleware *verified*:

```php
Route::get('profile', function () {
    // Only verified users may enter...
})->middleware('verified');
```

Una vez verificado el correo, el usuario será redireccionado a la ruta `/home`, para cambiar este comportamiento, en el controlador `VerificationController` se puede agregar la propiedad:

`protected $redirectTo = '/ruta-redireccionamiento';`

## Crear un correo electrónico

Para realizar el envío de un correo electrónico necesitamos tres elementos:

1. Una clase tipo `Mailable` que utilizaremos para "armar" el correo.
2. Una vista que será lo que reciba el destinatario.
3. Enviar el correo mediante la clase `Mail`.

### 1. Mailable

Para crear una clase `Mailable` utilizaremos:

`php artisan make:mail Reporte`

Una vez creada la clase, en el método `build()` podremos realizar la configuración de nuestro correo mediante los métodos:

* `from()`: Correo que aparecerá como remitente.
* `subject()`: Asunto del correo.
* `view()`: La vista que utilizaremos para el contenido del correo.
* `attach()`: Archivos adjuntos que queramos enviar.

#### Pasar datos a la vista (contenido) del correo

Para pasar información a la vista desde nuestra clase *Mailable* podemos pasar variables de dos formas:

1. Definir propiedades públicas, mismas que asignaremos en el constructor de la clase.
2. Pasar las variables mediante el método `with()`.


### 2. Vista (contenido) del correo

El contenido del correo lo podemos elaborar como una vista normal mediante un archivo `.blade` lo que nos generará un correo en html.

Opcionalmente podemos enviar el correo en texto plano mediante el método `text()` que podemos utilizar en vez de `view()` o incluso podemos utilizar ambos, lo que nos permite tener una versión de html con *view* y una de texto plano mediante *text*.

#### Vista con markdown

Podemos crear el contenido del correo utilizando plantillas y componentes preconstruidos mediante markdown. Además podemos generar una plantilla de este tipo al mismo tiempo que nuestro mailable agregando la opción `--markdown` y especificaremos el archivo a utilizar:

`php artisan make:mail Reporte --markdown=emails.reporte`

Componentes disponibles:

* Contenido del mensaje

	```
	@component('mail::message')
	
	# Contenido
	# Aquí podemos agregar los otros componentes
	
	@endcomponent
	```
* Botón

	```
	@component('mail::button', ['url' => $url, 'color' => 'success'])
	Texto del botón
	@endcomponent
	```
* Panel

	```
	@component('mail::panel')
	Contenido del panel
	@endcomponent
	```

* Tabla

	```
	@component('mail::table')
	| Laravel       | Table         | Example  |
	| ------------- |:-------------:| --------:|
	| Col 2 is      | Centered      | $10      |
	| Col 3 is      | Right-Aligned | $20      |
	@endcomponent
	```

### 3. Envío de correo

Para enviar el correo utilizamos la clase `Mail` done podemos especificar el destinatario, cc, bcc. Finalmente mediante el método `send()` le pasamos la instancia de nuestra clase *Mailable*.

```php
use Illuminate\Support\Facades\Mail;

Mail::to('correo@destino.com')
    ->cc('correo@cc.com')
    ->bcc('correo@bcc.com')
    ->send(new Reporte());
```

Esto significa que podemos utilizar `Mail` de forma muy versatil, pudiéndolo llamar desde un método en un controlador, dentro de algún ciclo para generar correos a muchos usuarios, en un scheduler o en un comando.