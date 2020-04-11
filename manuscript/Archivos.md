# Carga de Archivos

Cuando se cargan archivos a un sistema web, éstos se pueden almacenar dentro de la base de datos mediante una columna tipo *binary* o dentro de la estructura de archivos, nosotros nos enfocaremos en ésta última forma, ya que nos es más útil el mantener una separación entre archivos y la información de la base de datos.

Para cargar archivos requeriremos:

1. Formulario para seleccionar el o los archivos a cargar.
2. Recibir y guardar archivo.
3. Opcionalmente, crear un registro con información del archivo en una tabla para tal propósito.

## Tabla de información

Iniciaremos con el diseño de una tabla donde almacenaremos información referente al archivo, ésta es opcional, pero de mucha utilidad cuando queremos permitir la descarga de archivos.

La tabla se llamará *archivos* y contendrá las siguientes coclumnas:

- id
- nombre_original, string(255)
- nombre_hash, string(255)
- tamaño, integer
- mime, string(32)

**nombre_original** y **nombre_hash**

El funcionamiento de Laravel, es que cuando guardamos un archivo, éste es guardado con un nombre "hash" que permite que un usuario pueda subir dos archivos con el mismo nombre, pero al guardarlos mediante este "hash" evita que uno sobreescriba al otro, por esto agregarmos las columnas de *nombre_original* y *nombre_hash*.

**tamaño**

Conservar el tamaño del archivo, nos puede ser útil para mostrarlo como información al usuario, o llevar registro del espacio utilizado por los usuarios o sistema.

**mime**

Multipurpose Internet Mail Extensions o MIME es una convención que utilizaremos para cuando proporcionemos la descarga de los archivos, ya que como parte de la descarga se envían cabeceras con el tipo de contenido que se está descargando, lo cual ayuda al navegador a saber qué tipo de archivo es y qué acción realizar de acuerdo a la configuración del usuario.

## Formulario de carga

La carga de archivos se realiza mediante un formulario donde se incluya la propiedad `enctype="multipart/form-data"` y que tenga un `<input type="file">`.

```html
<form method="POST" action="{{ route('archvo.store') }}" enctype="multipart/form-data">
  @csrf
  <label for="archivo">Carga de Archivo</label>
  <input name="mi_archivo" type="file">
  <button type="submit" class="btn btn-primary">Enviar</button>
</form>
```

En caso de querer realizar una carga múltiple de archivos deberemos modificar nuestro *input* de la siguiente forma:

`<input name="mis_archivos[]" type="file" multiple>`

De esta forma se enviarán los archivos como parte de un arreglo *mis_archivos*.

## Recepción y guardado

### Recibir y validar el archivo

Cuando cargamos un archivo y lo queremos recuperar mediante PHP, éste archivo es guardado de forma temporal en un directorio interno de PHP, por lo que necesitamos moverlo a una ubicación permanente para guardarlo.

Para acceder al archivo temporal utilizamos cualquiera de las siguientes dos opciones:

`$file = $request->file('mi_archivo');`

`$file = $request->mi_archivo;`

Podemos verificar que la carga haya sido exitosa mediante `$request->file('mi_archivo')->isValid()` lo que nos regresa un booleano.


### Guardar el archivo

Para guardar el archivo utilizamos el método `store()` que por default guardará el archivo a partir de `storage/app`.

`$rutaArchivo = $request->mi_archivo->store('mis_archivos')`

Ésto guardará nuestro archivo con un nombre generado automáticamente (hash) en el directorio *storage/app/mis_archivos/*.

Si deseamos especificar un nombre para el archivo, lo podemos hacer mediante `storeAs()`:

`$rutaArchivo = $request->mi_archivo->storeAs('mis_archivos', 'archivo.jpg');`

En este caso, se guardará en el directorio *storage/app/mis_archivos/* con el nombre *archivo.jpg*

### Sistemas de archivos

Laravel provee "discos" (*disks*) que se pueden configurar desde `config/filesystems.php`, los discos preconfigurados son:

- local: `storage/app/` es donde por default se guardan los archivos cargados.
- public: `storage/app/public` es una ubicación para cargar archivos que queramos queden como públicos, es decir, expuestos para el navegador.
- s3: Almacenamiento en S3 de Amazon

Esto nos permite poder modificar en dónde se almacenarán nuestros archivos de forma muy sencilla. Se pueden agregar otros discos que nos permitan almacenar en otros servicios aunque se deben descargar los paquetes para realizar la conexión.

Con esto podemos al momento de guardar un archivo indicar el disco al cual queremos que se guarde:

`$ruta = $request->mi_archivo->store('mis_archivos', 's3');`

## Ejemplo

A continuación se muestra un ejemplo que permite la carga, listado y eliminación de archivos.

### Esquema para tabla *archivos*

```php
Schema::create('archivos', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('nombre_original');
    $table->string('nombre_hash');
    $table->string('tamaño');
    $table->string('mime');
});
```


### Formulario de carga

```php
<form method="POST" action="{{ route('archvo.upload') }}" enctype="multipart/form-data">
  @csrf
  <label for="archivo">Carga de Archivo</label>
  <input name="mi_archivo" type="file">
  <button type="submit" class="btn btn-primary">Enviar</button>
</form>
```

### Listado de archivos

```php
<div class="card">
  <div class="card-header">Archivos Cargados</div>
  <div class="card-body">
    <table class="table">
      <tr>
        <th>Archivo</th>
        <th>Tamaño</th>
        <th colspan="2">Acciones</th>
      </tr>

      @foreach($archivos as $archivo)
        <tr>
          <td>{{ $archivo->nombre_original }}</td>
          <td>{{ $archivo->tamaño }}</td>
          <td>
            <a href="{{ route('archivo.download', $archivo->id) }}" class="btn btn-sm btn-success">Descargar</a>
          </td>
          <td>
            <!-- Formulario para eliminar archivo -->
            {!! Form::open(['route' => ['archivo.delete', $archivo->id]]) !!}
            <button type="submit" class="btn btn-sm btn-danger">Borrar</button>
            {!! Form::close() !!}
          </td>
        </tr>
      @endforeach

    </table>
  </div>
</div>
```


### Controlador para archivos

```php

namespace App\Http\Controllers;

use App\Archivo;
use Illuminate\Http\Request;

class ArchivoController extends Controller
{
    public function upload(Request $request)
    {
        if ($request->mi_archivo->isValid()) { //Valida carga
            
            //Guarda en storage/app/archivos_cargados
            $nombreHash = $request->mi_archivo->store('archivos_cargados');
            
            //Crea registro en tabla archivos
            Archivo::create([
                'nombre_original' => $request->archivo->getClientOriginalName(),
                'hash' => $nombreHash,
                'mime' => $request->archivo->getClientMimeType(),
                'tamaño' => $request->archivo->getClientSize(),
            ]);
        }

        return redirect()->back();
    }

    public function download(Archivo $archivo)
    {
        //Obtiene ruta del archivo
        $rutaArchivo = storage_path('app/' . $archivo->hash);
        
        //La respuesta contiene el archivo con el tipo de documento
        return response()->download($rutaArchivo, $archivo->original, ['Content-Type' => $archivo->mime]);
    }

    public function delete(Archivo $archivo)
    {
        $rutaArchivo = storage_path($archivo->hash);

        //Verifica la existencia del archivo
        if (\Storage::exists($archivo->hash)) {
            \Storage::delete($archivo->hash); //Elimina archivo
            $archivo->delete(); //Elimina registro en tabla
        }

        return redirect()->back();
    }
}
```

### Rutas para manejo de archivos

```php
//Rutas para listado y carga de archivos
Route::get('archivo', function() {
    $archivos = App\Archivo::all();
    return view('archivos.archivoIndex', compact('archivos'));
});
Route::get('archivo/formulario', function() {
    return view('archivos.archivoForm');
});

Route::post('archivo/cargar', 'ArchivoController@upload')->name('archivo.upload');

Route::get('archivo/{archivo}/descargar', 'ArchivoController@download')->name('archivo.download');

Route::post('archivo/{archivo}/borrar', 'ArchivoController@delete')->name('archivo.delete');
```

### Implementación

El ejemplo funciona por sí solo, se tienen rutas para listado, carga, descarga y eliminación de archivos, sin embargo sería un mero repositorio de archivos, generalmente los archivos estarían relacionados con otro modelo, por ejemplo podemos querer que un usuario cargue archivos.

Para este caso lo único que tendríamos que hacer es agergar una relación 1:m entre User y Archivo, agregando la columna `user_id` en la tabla *archivos* y con esto poder utilizar las relaciones para listar los archivos de un cierto usuario.

Otra situación común puede ser que, además de querer cargar un documento y relacionarlo con un cierto usurio, queramos que se puedan cargar documentos relacionados con una materia. Para este caso, toda la lógica de los archivos es la misma y resultaría redundante tener modelos, tablas y controladores para archivos de *usuarios* y otro para archivos de *materias*. Para este caso, lo mejor es utilizar una relación polimórfica entre User, Materia y Archivo para con esto lograr que tanto User como Materia puedan tener archivos relacionados utilizando una misma estructura.

Adicionalmente, tanto el formulario como el listado se puden utilizar como vistas parciales, de tal forma que puedan ser incluidas en las vistas tanto de los usuarios como de las materias.