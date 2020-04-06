# Modelos (Eloquent ORM)

Un modelo es una clase que representará a una tabla en la base de datos, mediante el modelo se podrán crear, modificar o eliminar los registros en la tabla correspondiente.

La convención para nombrar un modelo es que sea la forma **singular** del nombre de la tabla, así un modelo para la tabla *alumnos* se llamará *Alumno*. 
En caso que la tabla conste de dos palabras la convención sería llamar a la tabla *domicilio_alumnos* y a modelo *DomicilioAlumno*

Para crear un modelo podemos utilizar:

`php artisan make:model Alumno`

Adicionalmente podemos agregar algunas opciones útiles:

- `--migration` o `-m` para la migración de creación de la tabla.
- `-c` para crear un controlador.
- `-r` para que el controlador sea tipo resource.

Así `php artisan make:model Alumno -mcr` creará un modelo Alumno, una migración para crear la tabla *alumnos* y un controlador *AlumnoController* tipo resource (con todos los método para un CRUD).

## Configuración básica

Existen varios atributos que nos sirven para configurar nuestro modelo:

- Table: En caso que el nombre modelo/tabla no siga el estandar, se puede indicar el nombre de la tabla mediante `protected $table = 'nombre_tabla';`
- Timestamps: `public $timestamps = false;` indica que la tabla no tendrá las columnas `created_at` `updated_at`.
- Fillable: Recibe un arreglo con las columnas que queramos **permitir** sean asignadas de forma masiva `protected $fillable = ['nombre', 'codigo', 'correo'];`
- Guarded: Recibe un arreglo con las columnas que queramos **impedir** que sean asignadas de forma masiva `protected $guarded = ['id', 'created_at', 'updated_at'];`
- Dates: Si tenemos columnas tipo *date* o *dateTime* puede ser util que al recuperarlas sean instancias de Carbon, para lo cual definimos `protected $dates = ['fecha_nacimiento'];`

## Consultas

Para realizar consultas utlizaremos el modelo como punto de partida, ya que Eloquent viene a ser una extensión más poderosa del Query Builder. Por este motivo podemos hacer uso de métodos como `where()`, `sum()`, `orderBy()`, etc.

Cuando hacemos una consulta, tenemos tres posibles resultados:

1. Una instancia del modelo.
2. Una colección de modelos. 
3. Una consulta sin resultados *(null)*.

### Consultar una instancia del modelo (un solo registro)

Caundo queremos recuperar un solo registro o instancia de un modelo podemos utilizar los métodos `find()` para buscar por *ID* o `first()` cuando agreguemos condiciones con `where()`, por ejemplo:

`$alumno = Alumno::find($idAlumno);`

`$alumno = Alumno::where('codigo', 112233)->first();`

En ambos casos obtenemos una instancia del modelo *Alumno* a partir del cual podemos hacer uso de las propiedades y métodos contenidos en ese modelo.

### Consultar muchos registros.

Los métodos `get()` o `all()` nos devolverán una colección (una instancia de `Collection`).

`$alumnos = Alumno::where('activo', 1)->get();`

Para recorrer esta colección podemos utilizar métodos para ciclos como:

```php
foreach ($alumnos as $alumno) {
    echo $alumno->nombre;
}
```

## Borrado lógico (Soft Deletes)

Se puede implementar de forma muy sencilla un borrado lógico de registros, es decir, que el registro se marcará como borrado, excluyéndolo de cualquier consulta de forma automática, pero en la base de datos se conservará el registro.

Para su implementación:

1. Utilizar la clase tipo "Trait" `Illuminate\Database\Eloquent\SoftDeletes`:
	
	```php
	namespace App;
	
	use Illuminate\Database\Eloquent\Model;
	use Illuminate\Database\Eloquent\SoftDeletes;
	
	class Alumno extends Model
	{
	    use SoftDeletes;
	}
	```

2. Agregar la columna `deleted_at` en la tabla correspindiente al modelo mediante el método `softDeletes()`:

	```php
	Schema::table('alumnos', function (Blueprint $table) {
	    $table->softDeletes();
	});
	```

Ahora cuando se llame al método `delete()` se asignará el *timestamp* de cuando se elimina el registro en la columna `deleted_at` y el registro será ignorado por cualquier consulta.

### Métodos útiles

- `trashed()`: Determina si un modelo ha sido eliminado.
- `restore()`: Restaura un modelo eliminado. También se puede aplicar dentro de una consulta para restaurar muchos registros.
- `forceDelete()`: Elimina un registro permanentemente, es decir, lo elimina de la base de datos.
- `withTrashed()`: Al realizar una consulta, también incluye los registros eliminados.
- `onlyTrashed()`: Consulta solo los registros eliminados.

## Accessors & Mutators

Los accessors y mutators permiten modificar los valores de los atributos al momento de su consulta o su asignación.

### Accessor

Los accessors generan una especie de atributos "virtuales", es decir que no pertenecen a las columnas de la tabla que representa nuestro modelo.

Para crear un accesor, en el modelo se crea un método con la estructura `getVariableAttribute()` donde *Variable* será el atributo que queremos recuperar.

#### Ejemplos

1. Un caso sencillo puede ser que necesitemos mostrar el nombre del alumno y su código como una sola cadena, por lo que dentro del modelo Alumno agregaríamos:

	```php
	public function getNombreCodigoAttribute()
	{
	    return $this->nombre . ' ' . $this->codigo;
	}
	```
	
	Para acceder al valor, solo necesitamos llamar al attributo "virtual" *nombre_codigo*:
	
	```php
	$alumno = App\Alumno::find(1);
	$nombreCodigo = $alumno->nombre_codigo;
	```

2. El segundo ejemplo es darle formato al nombre del alumno para que al ser consultado sea regresado en mayúsculas, por lo que en este caso se pasa como parámetro una variable que mantendrá el valor original del atributo:

	```php
	public function getNombreAttribute($value)
	{
	    return strtoupper($value);
	}
	```
	
	
	Ahora cuando se llame al atributo "nombre", siempre será regresado en mayúsculas.
	
	```php
	$alumno = App\Alumno::find(1);
	$nombre = $alumno->nombre;
	```

3. Otras aplicaciones pudiera ser el obtener el monto total de una compra:
	
	```php
	public function getMontoTotalAttribute()
	{
		return $this->costo * $this->cantidad  * $this->impuestos;
	}
	```	

Estos atributos son "calculados" al momento en que son requeridos, mediante la instancia del modelo, pero si queremos que siempre estén disponibles caundo realicemos cualquier consulta, lo deberemos indicar mediaten el atributo *appends*: `protected $appends = ['nombre_atributo']`.

### Mutator

El mutator nos permite modificar el atributo de forma automática antes de ser guardado, igualmente en el modelo crearemos un método con la estructura `setColumnaAttribute($value)` donde *Columna* es el nombre del atributo que corresponde a la columna en la tabla correspondiente.

#### Ejemplo

Agregando el siguiente método en el modelo Alumno, al guardar o modificar el atributno *nombre*, éste siempre será guardado en mayúscular, independientemente de cómo se llene en el formulario.

```php
public function setNombreAttribute($value)
{
	$this->attributes['nombre'] = strtoupper($value);
}
```

## Alcance de Consultas (Query Scopes)

Podemos tener situaciones donde queramos que se aplique un cierto alcance o filtrado automático de la información que se consulta, por ejemplo podemos querer que un profesor solo pueda interactuar con  alumnos que pertenecen a su materia.

Si bien podemos resolver esta consulta, puede que queramos aplicar el mismo filtro a diversos módulos como "asignar calificaciones", "asistencias", "listado de alumnos"; esto nos lleva a que en cada una de las consultas de los distintos módulos tuvieramos que implementar el mismo filtrado, es aquí donde el implementar scopes nos simiplifica el proceso.

Existen scopes globales y locales. 

### Scopes Globales

Los globales los podemos aplicar a mútliples modelos de nuestra aplicación, por ejemplo, si tenemos un sistema tipo SaaS (Software as a Service) que de servicio a muchas escuelas, es conveniente que cada modelo que contenga información de cada escuela que use nuestro sistema, tenga una columna `escuela_id` y que queramos que al consultar cualquier sección, la información mostrada esté limitada a la escuela correspondiente al usuario.

#### Crear scope global

Para crear un scope global debemos crear una clase que implemente la interface `Illuminate\Database\Eloquent\Scope`

En este ejemplo la clase la creamos dentro del directorio `app/scopes/`


```php
namespace App\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class EscuelaScope implements Scope
{
    /**
     * Aplica el scope al constructor de consultas de Eloquent.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @return void
     */
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('escuela_id', \Auth::user()->escuela_id);
    }
}
```

#### Implementar scope global en modelo

Para aplicar el scope a un modelo, sobreescribiremos el método `boot` y llamaremos al método `addGlobalScope()`

```php
namespace App;

use App\Scopes\EscuelaScope;
use Illuminate\Database\Eloquent\Model;

class Alumno extends Model
{
    /**
     * Método boot del modelo.
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new EscuelaScope);
    }
}
```

Esto genera que en cualquier consulta de *alumnos*, por ejemplo `Alumno::all();` que hagamos siempre se aplique el `EscuelaScope` de forma automática, lo que siempre filtraría a los alumnos de la escuela del usuario.

Esto mismo se podría replicar con otros modelos como "Aulas", "Maestros", "Laboratorios".

### Scopes Locales

El scope local se crea directamente dentro del modelo mediante métodos con prefijo *scope*.

En el siguiete ejemplo creamos dos scopes para los alumnos:

```
namespace App;

use Illuminate\Database\Eloquent\Model;

class Alumno extends Model
{
    /**
     * Consulta los alumnos sobresalientes.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeSobresalientes($query)
    {
        return $query->where('promedio', '>', 90);
    }

    /**
     * Consulta los alumnos aprobados.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeAprobados($query)
    {
        return $query->where('calificacion', '>', 60);
    }
}
```

Estos scopes se implementan al momento de la consulta, llamando al método sin el prefijo *scope*:

`$alumnosSobresalientes = Alumno::sobresalientes()->get();`

`$alumnosAprobados = Alumno::aprobados()->get();`

Los scopes de este ejemplo nos sirven a que nuestras consultas sean semánticamente mas sencillas de entender y la idea es que creemos scopes de consultas que puedan resultar repetitivas.

