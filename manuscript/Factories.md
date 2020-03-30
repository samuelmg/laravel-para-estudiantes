# Factories

Las factories son archivos que nos ayudan a generar información para pruebas, en sí son fábricas para modelos,file:///C:/Users/Samuel/Documents/laravel-para-estudiantes/manuscript/Recetas.md en ellos se define un "plano" con con el cual se construirán los datos de prueba. Usualmente se utilizan junto con [Faker](https://github.com/fzaninotto/Faker) que es una librería que genera datos por nosotros.

## Crear una factory

Para crear una factory utilizaremos el comando:

`php artisan make:factory NombreModeloFactory`

La opción `--model` nos ayuda a generar la factory con el modelo indicado:

`php artisan make:factory NombreModeloFactory --model=NombreModelo`

Las factories se guardarán en el directorio `database/factories`

## Ejemplo

Considerando la tabla *alumnos* una factory para el modelo Alumno se vería de la siguiente forma:

```php
use Faker\Generator as Faker;

$factory->define(App\Alumno::class, function (Faker $faker) {
    return [
    	'programa_educativo_id' => $faker->numberBetween(1, 2),
        'nombre' => $faker->name,
        'correo' => $faker->unique()->safeEmail,
        'codigo' => $faker->randomNumber(),
        'fecha_nacimiento' => $faker->date(),
    ];
});
```

En este ejemplo consideramos que se han generado dos programas educativos, por lo que mediante *faker* asignamos de forma aleatorea el programa educativo a cada alumno que se creará.

## Implementación

La factory por si sola no generará los registros, sino que deberá ser utilizada como instructivo en la generación de registros, por lo que debe ser implementada dependiendo del propósito.

### En un seeder

Se puede utilizar una factory para generar registros de prueba, en el siguiente ejemplo se generarían 20 registros en la tabla *alumnos*:

```php
public function run()
{
    factory(App\Alumno::class, 20)->create();
}
```

### Como auxiliares para realizar pruebas (Testing)