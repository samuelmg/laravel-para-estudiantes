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
