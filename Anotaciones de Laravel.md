
# Anotaciones de Laravel

## RouteIs permite acceder al path de la ruta laravel/laruta

`{{ request()->routeIs('laruta') ?? 'active' : ''}}`

---

### Usado en la validacion en el controlador

`request()->validate([])`

---

### errores en un formulario

`{{ $errors->any()}}` -> De un solo error

`{{ $errors->all()}}` -> para mostrar todos

---

### Para solo usar ciertos métodos por una ruta

`Route::resource('projects', 'PortfolioController')->only(['index', 'create']);`

---

### Para no usar los métodos no mencionados

`Route::resource('projects', 'PortfolioController')->except(['index', 'create']);`

---

### Para cambiar la ruta de edit a editar

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Route; //Importante colocar este Facades

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar'
        ]);
    }
}
```

---

### Para cargar un archivo con funciones

Está en composer.json y se debe correr luego composer dumpautoload

```json
  "autoload": {
        "psr-4": {
            "App\\": "app/"
        },
        "classmap": [
            "database/seeds",
            "database/factories"
        ],
        "files" : ["app/helpers.php"]
    },
```

---

### Para cambiar la configuración de en a es

'locale' => 'es', //en config/app y posteriormente crear una carpeta en resources/lang

---

### Traducción en en front

`<h1>{{__('Contact')}}</h1>`

Y crear en lang/es.json

### Como obtener resultados ordenados

```$portfolio = Project::orderBy('created_at', 'DESC')->get();
$portfolio = Project::latest('updated_at')->get();
```

### Creando un formrequest para validar campos por fuera del controlador

```terminal
php artisan make:request CreateProjectRequest
```

### Names cambia el nombre de las rutas y parameters cambia los nombres en los parametros de las rutas

```php
Route::resource('portafolio', 'ProjectController')
    ->parameters(['portafolio' => 'project'])
    ->names('projects');
```

### Crear otro campo en una table existente

```terminal
php artisan make:migration add_phone_to_messages_table --table=messages
```

### Cambiar las asignaciones en una tabla por medio del modelo

```php
class Post extends Model {
    protected $primaryKey = "mi_otro_id_con_primary_key";
    public $incrementing = false; //Evita el valor a incrementar
    protected $keyType = "string"; //Para colocar el tipo strin el primary key
    protected $timestamp = false; //Para no usar los timestamp 
}
```

### Relacion 1 a 1

Modelo: Address -> Método: belongsTo

``` php
    public function user() {
        return $this->belongsTo();
    }
```

Modelo: User -> Método: hasOne

``` php
    public function address() {
        return $this->hasOne(Address::class);
    }
```

### Relacion 1 a n

Modelo: Post -> Método: belongsTo

``` php
    public function user() {
        return $this->belongsTo();
    }
```

Modelo: User -> Método: hasMany

``` php
    public function post() {
        return $this->hasMany(U::class);
    }
```

### Relaciones en la tablas

```php
    //Relacion en las tablas
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('country_id');
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();

            $table->foreign('country_id')->references('id')->on('countries');
        });
    }
    //Creacion de la tabla pivote create_role_user_table
     public function up()
    {
        Schema::create('role_user', function (Blueprint $table) {
            $table->id();
            #Esta es la tabla pivote para las dos relaciones entre el usuario y el rol cuando varios roles estan asignados a varios usuarios
            $table->unsignedBigInteger('role_id');
            $table->unsignedBigInteger('user_id');
            $table->timestamps();

            $table->foreign('role_id')->references('id')->on('roles');
            $table->foreign('user_id')->references('id')->on('users');
        });
    }
```

### Resolviendo el problema de 1 + N

https://styde.net/instalar-barra-de-debug-en-laravel/ Revisar el tutorial para implementar el debugbar

```terminal
composer require barryvdh/laravel-debugbar --dev
```

Para que Laravel Debugbar funcione la variable APP_DEBUG en el archivo .env debe ser igual a true.

```php
    #Usar with para reducir los tiempos de carga donde se pasan los modelos relacionados
    $post = Post::with(['user', 'category'])->get();
```

### Para generar o usar componentes de blade

```php
#Forma antigua en la vista
@section('jumbotron')
    @component('components.jumbotron')
        @slot('title', 'Bienvenido a About')
        @slot('body')
        Lorem ipsum dolor, sit amet consectetur adipisicing elit. Nulla, voluptas?
        @endslot
        @slot('button', 'About')
    @endcomponent
@endsection

#php artisan make:component Field el comando para generar el componente
<x-field title="Contact" body="Este es el cuerpo del Contacto" button="Conctact"></x-field>
#los atributos se pasan al constructor de app/view/components/Field
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Field extends Component
{
    public $title;
    public $button;
    /**
     * Create a new component instance.
     *
     * @return void
     */
    public function __construct(string $title,  string $button)
    {
        $this->title = $title;
        $this->button = $button;
    }

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|string
     */
    public function render()
    {
        return view('components.field');
    }
}

```

### Publicar la carpeta que tiene el modelo del correo para resetear la contraseña y editarla

php artisan vendor:publish y luego seleccionar Tag: laravel-notifications

### Agrupación de rutas

```php
    # namespace('Admin') si comparten el mismo nombre de la ruta en carpeta
    # middleware('auth') si comparten el mismo middleware
    # name('admin.') englobar los nombres de las rutas si se está usando la misma pero cambiando el valor después del punto
    # prefix('admin') el nombre del prefijo para el grupo de rutas

    Route::prefix('admin')->namespace('Admin')->middleware('auth')->name('admin.')->group(function() {

        # Si se quiere usar otro name y la ruta / apunta directo a admin y no a la ruta principial
        Route::get('/', 'AdminController@index')->name('home');

        Route::resource('users', 'UsersController');

    });

```
