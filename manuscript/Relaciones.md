# Relaciones

Para establecer una relación debemos considerar el dónde se guardará la información de la relación, es decir, la relación a nivel de tablas. Así mismo debemos generar los método correspondientes en cada modelo que estamos relacionando, ya que es mediante ellos que crearemos y accederemos a la relación.

## Relación muchos a muchos

Para ejemplificar esta relación, utilizaremos una relación de muchos a muchos entre las tablas `alumnos` y `materias`, por lo que asumimos la existencia de los modelos `Alumno` y `Materia`.

### Tabla Pivote

Para guardar la relación entre las dos tablas, requerimos crear una tercer tabla que se conoce como *pivote* donde se guardarán las llaves foraneas de las dos tablas que estamos relacionando.

El estandar para nombrar la tabla *pivote* es utilizar los nombres de las tablas que se relacionan en su forma singular separadas por un guión bajo y en orden alfabético. Esta tabla puede o no tener una columna id, ya que solo debe contener las dos llaves foraneas. Adicionalmente se pueden agregar columnas en caso de requerir guardar información reelevante a la relación.

En nuestro ejemplo la tabla pivote se llamaría `alumno_materia` con columnas `alumno_id` y `materia_id`.

### Relación entre modelos

La relación entre los modelos se realiza agregando un método en cada modelo llamado como el modelo que se está relacionando en su forma plural. Dentro del método se utilizará el método `belongsToMany()` cuyo primer parámetro es la clase del modelo que estamos relacionando.

Siguiendo con el ejemplo, dentro del modelo `Alumno` se agrega el método `materias()`:

```php
public function materias()
{
    return $this->belongsToMany(Materia::class);
}
```

Mediante este método se accederá a las materias que están relacionadas con el alumno.

De forma similar, en el modelo `Materia` agregamos un método `alumnos()`.

```php
public function alumnos()
{
    return $this->belongsToMany(Alumno::class);
}
```

### Uso de la relación

Generalmente la implementación se realiza dentro de los métodos de un controlador donde se realizan cambios a la base de datos, es decir, los métodos *store*, *update* y *destroy*, pero ésto no es obligatorio y depende de la lógica de nuestra aplicación.

Para crear o eliminar una relación entre los modelos, debemos tener una instancia de uno de los dos modelos, pudiendo ser cualquiera de los dos, en este caso lo veremos desde el punto de vista del alumno: `$alumno = Alumno::find($id);`

Esto no se requiere si se está inyectando una instancia como parte de los parámetros de la función en el controlador, `public function update (Alumno $alumno) { ... }`.

A partir de esa instancia utilizamos el método de la relación `$alumno->materias()` para poder:

- **Crear relación** mediante el método *attach()* con los siguientes parámetros:
	+ El ID del modelo a relacionar: `$alumno->materias()->attach($id_materia);`
	+ Un arreglo de IDs a relacionar: `$alumno->materias()->attach([$id_materia_1, $id_materia_2, ...]);`
	+ Un arreglo como segundo parámetro con las columnas y valores adicionales que existan en la tabla pivote: `$alumno->materias()->attach($id_materia, ['nombre_columna' => $valor_columna]);`
- **Eliminar una relación** con *detach()*:
	+ Sin parámetros, eliminará todas las materias del alumno: `$alumno->materias()->detach();`
	+ El id de la materia a eliminar la relación: `$alumno->materias()->detach($id_materia);`
	+ Un arreglo de ids: `$alumno->materias()->detach([$id_materia_1, $id_materia_2, ...]);`
- **Sincronizar** cambios con *sync()*:
	+ Agrega la relación a los IDs que se pasen a *sync()* y elimina cualquier relación existente que no se incluya `$alumno->materias()->sync([$id_materia_x, $id_materia_y]);`
	+ No elimina relaciones existente, solo agregaría las nuevas, pero también impide duplicar una relación: `$alumno->materias()->syncWithoutDetaching([$id_materia_x, $id_materia_y]);`

Los méteodos *attach*, *detach* y *sync* actualizan la información en la tabla pivote `alumno_materia`.
