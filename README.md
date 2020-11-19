

# Laravel 20-2 

### Crear proyecto
Se puede usar cualquiera de los dos comandos
> laravel new MiProyecto 
> composer create-project --prefer-dist laravel/laravel MiProyecto
---
### Instalar Bootstrap y Autenticación

>  composer require laravel/ui
>  php artisan ui bootstrap --auth
>  npm install
>  npm run dev
---
## Anallely
### Creación de base de datos

Abrimos phpmyadmin y creamos una nueva base de datos que tenga el mismo nombre que nuestro proyecto.

Revisamos que los datos del archivo .env sean correctos.

Corremos las migraciones de autenticación
>php artisan migrate

Probamos haciendo un nuevo registro 

---
### Modificar tabla users

Explicar teoría de Eloquent y Bases de Datos

En el archivo de la migración haremos los cambios pertinentes
 
> /database/migrations/create_users_table

Agregamos una columna de tipo booleano que por default tendrá falso, es decir, que no es administrador. 

```php
$table->boolean('admin')->default(false); 
```
Hacemos otra de tipo string que contendrá la ruta de la imagen que el usuario elija y como será opcional puede tener un valor nulo.

```php
$table->string('avatar')->nullable(); 
```

Como estamos sobreescribiendo en la misma migración hay que correr todo desde cero

>php artisan migrate:fresh

Podemos revisar que nuestra BD se haya actualizado correctamente y haremos un nuevo usuario.

----
### Integrando vista welcome 

El comando de Bootstrap que hicimos al principio, jala los archivos de ```bootstrap.min.css``` y ```bootstrap.min.js```, así como jquery y otras dependencias.

Además, necesitamos nuestros archivos styles.css y las imágenes. Las copiamos de nuestra carpeta de front y las pegamos en nuestra carpeta ```/public```

Empezamos con la vista de welcome, borramos todo el contenido y pegamos el de ```index.html```

Vemos que le hacen falta algunos estilos, es porque necesitamos cambiar la ruta de referencia.

Para no tener problema usando los archivos de la carpeta public, definiremos las rutas con ayuda de assets de la siguiente manera. 
 ```html
 {{asset('css/app.css')}}
 {{asset('css/styles.css')}}
 {{asset('css/mediaqueries.css')}}
```

Eso indica que el archivo se buscará directamente dentro de la carpeta 
>/Public

El archivo de ```bootstrap.css```  ya está referenciado en el archivo app.css dentro de la misma carpeta

Y el de js lo incluiremos de la misma forma
```html
<script src="{{asset('js/app.js')}}"></script>
```
Ya tenemos la vista, pero si damos click en los links estos no van a funcionar. En Laravel, en lugar de poner el archivo de destino, se pone la ruta de destino, esto es porque no unicamente se muestra el archivo sino muchas veces se hacen acciones detrás de. 

---
### Rutas

*Explicar teoría de rutas*

Para las rutas podemos hacerlo mediante *url* si esta no tiene nombre, o si lo tiene, podemos hacer uso de *route*, de la siguiente manera:

```html
<a href="{{ route('register') }}">
 ```

Para saber el nombre de nuestras rutas usamos el comando
> php artisan route:list
---
## Juan Pablo

### Formularios

El formulario tampoco funcionará porque al dar click en el botón no le estamos dando una acción.

*Explicar teoría de Formularios*

Cuando usamos formularios hay 4 cosas indispensables

 1. Action: pondremos la ruta a la enviaremos al usuario al dar click en el botón de submit
 2. Method: POST o GET
 3. @csrf: por cuestiones de seguridad
 4. Todos los inputs deben tener name: para identificar los campos que se están mandando

### Vistas de autentificación

Algunas cosas las copiaremos tal cual están ya que el punto es usar lo que Laravel ya nos da hecho. 
Copiamos los inputs y los mensajes de error. Por el momento solo omitimos el apartado de restaurar la contraseña. 
*Revisamos que funcione*

Lo mismo haremos con la vista de ```register.blade.php``` y probamos

---
## Gabriel

### Layouts

Revisamos nuestro archivo ```home.blade.php``` y vemos que ya no está nuestro código completo, esto es por que Blade nos permite reciclar las partes del código que no cambian, como lo es la etiqueta head y normalmente la navbar. Estas "plantillas" están en la carpeta de ````/public/layouts````, vamos a borrar todo.

Copiamos el head de nuestra vista welcome, el script y el header de layout.app.html

El contenido dinámico lo agregamos con la siguiente linea

```php
@yield('content')
```
Recargamos la página y ya está nuestra navbar

Vemos que nos sale la opción de admin aun cuando no lo somos, para ocultar esa opción a los usuarios normales lo hacemos con una condicional

```php
@if(Auth::user()->admin)
	<a href="#">Admin</a>
@endif
 ````

https://laravel.com/docs/8.x/blade#if-statements 
*Podemos ingresar a la base de datos y hacerlo administrador desde ahí*

En donde dice 'mi nombre' ponemos la variable 'name' del usuario autenticado

 ```php
 {{ Auth::user()->name }}
 ```

Modificamos las ruta de Logout

 ```html
  <a class="dropdown-item" href="{{route('logout')}}" onclick="event.preventDefault();document.getElementById('logout-form').submit();">Cerrar sesión</a>
````

Con este formulario
```html
<form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
@csrf
</form>
```` 

Probamos

---
### Integrando vista principal (home)

Todos los archivos que usen la navbar serán como una extensión de ese archivo, por lo que para ligarlos pondremos lo siguiente:

```php
@extends('layouts.app')
@section('content')
<!-- Aquí va el contenido -->
@endsection
```
Copiamos el contenido de ````<main>```` del archivo `home.html` y lo pegamos en
>Resources/views/home.blade.php
---
### Perfil de administrador

#### Middleware
*Explicar teoría de Middleware*

Creamos un nuevo middleware
> php artisan make:middleware AdminMiddleware

Abrimos el nuevo archivo
> \app\Http\Middleware\AdminMiddleware
>
Y ponemos dentro de la función:
```php
if(auth()->user()->admin){
  return $next($request);
}
return redirect('home');
```
Esto permitirá que unicamente los usuarios administradores tengan acceso, en caso de que no lo sean los redirige a la vista home.

Tenemos que darle un alias a ese Middleware para que sea reconocido
>App\Http\Kernel.php
````php
'admin'=>\App\Http\Middleware\AdminMiddleware::class
````

Hacemos una ruta de prueba en ```web.php```

```php
Route::get('prueba', function(){
	return "Soy admin";
})->middleware('admin');
````
---
*Tarea:*
Podemos hacer otro para que una vez que el usuario inicie sesión ya no pueda ir  a la ruta raiz, entonces aplico un Middleware que haga que si el usuario está autenticado lo redirija igual a esta vista de home. 

>php artisan make:middleware WelcomeMiddleware

```php
 if(!auth()->guest()){
   return redirect('home');
}
return $next($request);
```` 
Lo aplicamos a nuestra ruta raíz
```php
Route::get('/', function () {
    return view('welcome');
})->middleware('welcome');
````
Y le damos el alias
```php
'welcome' => \App\Http\Middleware\WelcomeMiddleware::class
````
---
También podemos agrupar rutas y aplicarles el mismo Middleware

````php
Route::middleware(['admin'])->group(function () {

});
````

Y ahora todo lo que esté dentro de esa función, no será accesible para usuarios que no sean administradores. 

Podemos probar con la ruta de ejemplo que hicimos antes
https://laravel.com/docs/8.x/middleware

#### Layout admin
Haremos un nuevo layout para que los administradores tengan su propia barra de navegación.

Cómo es muy parecida a la que ya tenemos, duplicaremos ese archivo bajo el nombre de `admin.blade.php` en la misma carpeta
>Resources/views/layouts/admin.blade.php

Solo cambiamos las opciones de 'Calificaciones' por la de 'Usuarios' y la de 'Admin' por 'Home', a esta última le ponemos la ruta correspondiente. 

Para verificar, en la ruta de prueba que habíamos hecho cambiamos el valor de regreso por la vista de nuestra nueva plantilla. El nombre que va dentro del parentesis hace referencia a la ruta del archivo. 

````php
Route::get('prueba', function () {
	    return view('layouts.admin');
});
````
---
## Juan Pablo
### Tareas

Ya tenemos la estructura principal de nuestra página y los permisos, ahora pasamos a la sección de tareas, donde los usuarios administradores podrán subir una tarea a la plataforma, y esta será visible para los alumnos. 

Hacemos modelo y migración. Estas dos siempre van juntas.

> php artisan make:model Tarea
> php artisan make:migration create_tareas_table

o para correr los dos en uno

> php artisan make:model Tarea -m

*El nombre del modelo deberá empezar con mayúscula*

#### Tabla de Tareas en nuestra Base de Datos

Definimos los datos que contendrá nuestra tabla de tareas,  vamos al archivo de migración que acabamos de crear

>Database/migrations/create_tareas_table.php

Y definimos qué columnas llevará nuestra tabla, tomando el cuenta el tipo de dato.

```php
$table->string('title');
$table->text('description')->nullable();
$table->date('date');
$table->time('time');
$table->string('file')->nullable();
````

Los tipos de datos disponibles y modificadores los encontramos aquí https://laravel.com/docs/8.x/migrations#available-column-types 

Hacemos la migración
>php artisan migrate

## Anallely

#### Controlador de Tareas

Para controlar todo lo que haremos con las tareas como agregar, editar, borrar, etc. Haremos un controlador de recursos que contiene las operaciones fundamentales que se pueden hacer con una tabla. https://laravel.com/docs/8.x/controllers#actions-handled-by-resource-controller 

> php artisan make:controller TareaController -r

Lo primero es especificar la ruta en el archivo
>Routes/web.php

Esta ruta estará dentro de nuestro Middleware de administrador puesto que unicamente ellos podrán subir tareas a la plataforma

````php
use App\Http\Controllers\TareaController;
Route::resource('admintareas',TareaController::class);
````

La primera linea es para importar el controlador, sin eso no funciona

El primer parámetro corresponde al nombre que nosotros definimos para esa ruta y el segundo corresponde al controlador

#### Tareas index
Creamos una nueva carpeta para las vistas de tareas
> Resources/views/admin/tareas

Y dentro creamos un nuevo archivo `index.blade.php`
Lo primero será heredar nuestra plantilla con la barra de navegación

```php
@extends('layouts.admin')
@section('content')
...
@endsection
````

Y dentro de la sección pegamos el código de nuestro archivo `tareas_index.html`

Para ingresar a esta vista pasaremos por el controlador de Tareas y la sub ruta index

Dentro del controlador
>App/Http/Controllers/TareaController.php

en la funcion **index** regresamos nuestra vista 

```php
return view('admin.tareas.index');
````

En la plantilla `layouts/admin.blade.php`ponemos su

```php
<a href="{{ route('admintareas.index') }}">Tareas</a>
````

También la pondremos en `layouts/app.blade.php` para que al dar click en la opción de 'admin' sea la página principal

````php
<a href="{{ route('admintareas.index') }}">Administrador</a>
````

#### Crear Tareas
Hacemos nuestra nueva vista `create.blade.php`
>Resources/views/admin/tareas/create.blade.php

Heredamos la plantilla
```php
@extends('layouts.admin')
@section('content')
...
@endsection
````

Y dentro de la section pegamos el contenido del archivo `tareas-create.html`

Dentro del controlador
>App/Http/Controllers/TareaController.php

en la funcion **create** regresamos nuestra vista 
```php
return view('admin.tareas.create');
````

Y en nuestro archivo `tareas/index.blade.php`buscamos el botón de 'Crear Tarea' y le ponemos la ruta
````php
<a href="{{ route('admintareas.create') }}">Agregar Tarea</a>
````

La función crear nos regresa la vista con el formulario donde podremos llenar los datos, pero la función que guardará esos datos es **@store**

#### Guardar Tareas

En la etiqueta `<form>`definimos los siguientes atributos:

- `action="{{route('admintareas.store')}}"`
Para poder usar el formulario hay que definirle una acción, en este caso nos mandará a la funcion **@store** que es la que guardará la tarea. 
- `enctype="multipart/form-data"`
Para que nos permita enviar archivos
- `method="POST"`
- ```@csrf```

Adicional, en el campo de fecha podemos darle valor de la fecha actua
```php
value="{{ date('Y-m-d')}}"
````  

Solo falta definir en donde se guarda cada dato

Dentro del controlador
>App/Http/Controllers/TareaController.php
>
en la función **store** empezamos creando una nueva instancia de nuestro modelo Tarea, que se traduce como una nueva fila en nuestra tabla

```php
$tarea=new Tarea();
````

*Cada que hagamos uso de un modelo, hay que importarlo con `use App\Models\Tarea;` ya que no se encuentran dentro de la misma carpeta*

Columna por columna especificamos cual es el dato que se almacena ahí, esto haciendo uso del atributo 'name' de cada input

```php
$tarea->title=$request->title;
$tarea->description=$request->description;
$tarea->date=$request->date;
$tarea->time=$request->time;
````

Los archivos no se guardan directamente en la base de datos, sino como un archivo en nuestro proyecto, por lo que en la base unicamente se guardará la ruta a ese archivo.

Como el archivo es opcional usaremos una condicional if

```php
if ($request->hasFile('file')) {
}
```
Hacemos variable donde se guardará el documento
```php
 $file = $request->file;
```
Hacemos otra variable donde se guardará el nombre de ese archivo
```php
$name=$file->getClientOriginalName();
```
Hacemos una nueva carpeta donde se almacenarán nuestros archivos
>/Public/docs

Y hacemos una variable en donde se guardará esa ruta
```php
$destination = public_path() . '/docs/';
```
Ahora movemos el archivo tal cual que está guardado en la variable `$file`, con la función `move` que recibe como primer parámetro el destino y como segundo el nombre.
```php
$file->move($destination, $name);
```
Por último guardaremos en la base de datos el nombre de nuestro archivo
```php
$tarea->file=$name;
```
Guardamos todos los cambios
```php
$tarea->save();
```
*Esta linea ya va fuera del if*

Y regresamos al usuario a la vista principal
```php
return redirect('admintareas');
```
Probamos creando nueva tarea

### Mostrar tareas en vista de admin

Ya que tenemos guardadas nuestras tareas en la base de datos lo que sigue sería mostrarlas en la página index de admintareas.
Esto lo haremos pasando los datos desde el controlador hacia la vista.

Dentro del controlador
>App/Http/Controllers/TareaController.php
>
en la función **index** :

Guardaremos todos los datos en una variable
```php
$tareas = Tarea::all();
```
Ya que los guardamos, los tenemos que pasar a la vista

````php
return view('admin.tareas.index',['tareas'=>$tareas]);
````

Como primer parametro es la ruta de la vista y como segundo todas las variables que vayamos a pasar, en este caso nadamas es una y esa contiene todos nuestros datos

Ya los tenemos disponibles en la vista ahora solo falta mostrarlos. Usaremos un ciclo for para reciclar el código
https://laravel.com/docs/8.x/blade#control-structures 

```php
@foreach($tareas as $tarea)
	<div class="col mb-4">
	</div>
@endforeach
```
Los datos los ponemos en donde corresponden por medio de variables que se ponen entre llaves `{{ $tarea->title }}`

```php
{{$tarea->title}}
{{$tarea->date}}
{{$tarea->time}}
{{$tarea->description}}
````

Y para el archivo:
```php
href="{{ asset("tareas/$tarea->file") }}"
{{ $tarea->file }}
```

#### Editar Tareas
	
La función **@edit** nos regresará la vista de un formulario donde podremos editar la tarea, este formulario será el mismo que en la función **@create** pero con los datos visibles.

Hacemos nuestra nueva vista `edit.blade.php`

>Resources/views/admin/tareas/edit.blade.php

Y pegamos el mismo código de `create.blade.php`

En  `index.blade.php` le ponemos ruta al botón **'Editar'** junto con el parámetro para que sepa exactamente que tarea editar
```html
href="{{ route('admintareas.edit', $tarea->id) }}"
```` 
En la función **@edit**
>App/Http/Controller/TareaController 

Primero buscamos la tarea haciendo uso del parámetro id
https://laravel.com/docs/7.x/eloquent#retrieving-single-models)

```php
$tarea=Tarea::findOrFail($id);
return view('admin.tareas.edit', ['tarea'=>$tarea]);
```
En el form de la vista ```edit.blade.php```
```php
method="POST" 
action="{{ route('admintareas.update', $tarea->id) }}"
enctype="multipart/form-data"
@method('PATCH')
@csrf
```
Y mostramos los datos en el formulario para  los podamos editar, cómo son input, podemos hacer uso del atributo "value"

```php
value="{{$tarea->titulo}}"
value="{{$tarea->date}}"
value="{{$tarea->time}}"
```
Para textarea lo pondremos como contenido dentro de las etiquetas

```php
{{$tarea->description}}
```

Y para el archivo, como es opcional, hay que poner una condicional 

Si el archivo existe, lo mostramos
```php
@if(!$tarea->file==NULL)
<label class="d-block text-left">Archivo Actual:</label>
<a target="_blank" href="{{ asset("tareas/$tarea->file") }}">{{ $tarea->file }}</a>
```

Y además damos la opción de modificarlo:

```php
<div class="form-group">
<label>Modificar Archivo:</label>
<input  type="file" name="file" enctype  >
</div>
```
Si no hay archivo, únicamente damos la opción de subirlo
```php
@else
<div class="form-group">
<label>Subir Archivo:</label>
<input type="file" name="file" enctype  >
</div>
@endif
```
Ahora definimos donde guardar los datos modificados. Es muy parecido a la funcion **@store**

En la función **'@update'**
>App/Http/Controller/TareaController 

Buscamos la tarea con la que estamos trabajando
```php      
$tarea=Tarea::findOrFail($id);
```
Y copiamos todo lo de la función **'@store'**
```php
$tarea->title=request($key='title');
$tarea->description=request($key = 'description');
$tarea->date=request($key = 'date');
$tarea->time=request($key = 'time');
$tarea->filetype=request($key = 'filetype');
if ($request->hasFile('file')) {
	$file = $request->file('file');
	$name=$file->getClientOriginalName();
	$destination = public_path() . '/tareas/';
	$file->move($destination, $name);
	$tarea->file=$name;
}
```
Guardamos los cambios hechos y redirigimos a la vista principal
```php
$tarea->update();
return redirect('admintareas');
```
#### Ver tareas por separado

Hacemos nueva vista
> /tareas/show.blade.php

Y pegamos mismo código que `tareas-show.html`, 

Le damos ruta a nuestro boton ‘Ver’ en `tareas/index`, recordemos que recibe un id también
```php
{{ route('admintareas.show', $tarea->id) }}">
```
En la función **'@show'**
>App/Http/Controller/TareaController 
```php
$tarea=Tarea::findOrFail($id);
return view('admin.tareas.show', ['tarea'=>$tarea]);
```

#### Eliminar tareas

Utilizaremos el botón que está en `tareas/index.blade.php` que es tipo submit, entonces al darle click hará la action que tenga el formulario

```php
method="POST" 
action="{{ route('admintareas.destroy', $tarea->id) }}" 
@csrf
@method('DELETE')
```
En la función **'@destroy'**
>App/Http/Controller/TareaController 

```php
$tareas=Tarea::findOrFail($id);
$tareas->delete();
return redirect('admintareas');
```

### Mostrar tareas en home

En la función **@index**
>App/Http/Controller/HomeController 

Guardamos los datos en una variable y los pasamos a la vista
 ```php
$tareas = Tarea::all();
return view('home',['tareas'=>$tareas]);
```
Importamos con ```use App\Models\Tarea;```

Y en nuestra vista `home.blade.php` hacemos un ciclo foreach
```php
@foreach($tareas as $tarea)
    <div class="col mb-4">   
    </div>
@endforeach
```
Ponemos las variables donde corresponden 
```php
{{$tarea->title}}
{{$tarea->date}}
{{$tarea->time}}
{$tarea->description}}
```
Y para el archivo
```php
href="{{ asset("tareas/$tarea->file") }}"
{{ $tarea->file }}
```
---
## Arturo
## Entregas

Hacemos modelo y migración

> php artisan make:model Entrega -m

Ahora definimos los atributos de la tabla
>Database/migrations/create_entregas_table

#### Tabla
A qué usuario pertenece y que tarea está subiendo

```php
$table->unsignedBigInteger('user_id');
$table->unsignedBigInteger('tarea_id');
```
El archivo que está entregando
```php
$table->string('file')->nullable();
```
Los comentarios
```php
$table->text('comments')->nullable();
```
La calificación
```php
$table->integer('cal')->default(0)->nullable();
```
Y definimos las que serán llaves foráneas
```php
$table->foreign('user_id')->references('id')->on('users');
$table->foreign('tarea_id')->references('id')->on('tareas');
```
Hacemos la migración
>php artisan migrate

### Relaciones

Ahora hay que definir de que manera se van a relacionar los modelos User, Tarea y Entrega por medio de funciones

>App\Models\User.php

```php
public function entregas(){
return $this->hasMany('App\Models\Entrega');
}
```
>App\Models\Entrega.php

```php
public function user(){
return $this->belongsTo('App\Models\User');
}

public function tarea(){
return $this->belongsTo('App\Models\Tarea');
}
```
>App/Tarea.php

```php
public function entregas(){
return $this->hasMany('App\Models\Entrega');
}
```
### Controlador

Hacemos controlador
>php artisan make:controller EntregaController -r

Hacemos la ruta del controlador en 
>Routes/web.php

```php
Route::resource('entrega', EntregaController::class);
```
e importamos

```php
use App\Http\Controllers\EntregaController;
```

Hacemos vista `show.blade.php` y nueva carpeta
>Resources/views/entregas/show.blade.php

Heredamos nuestra plantilla
```php
@extends('layouts.app')
@section('content')
@endsection
```
Y pegamos código de `entregas.html` dentro de la sección

En la vista `home.blade.php` damos ruta a nuestro botón de subir archivo, el cuál nos llevará a otra vista

```php
{{ route('entrega.show', $tarea->id) }}
```

En el controlador, la función **@show**
```php
$tarea=Tarea::findOrFail($id);
return view('entregas.show', ['tarea'=>$tarea]);
````

Importamos con ```use App\Models\Tarea;```

En nuestro formulario en `show.blade.php` tenemos un input que es invisble para el usuario, lo usaremos de auxiliar para pasar el id de la tarea 
```php
value="{{ $tarea->id }}"
```
Y le ponemos action a nuestro formulario
```php
action="{{ route('entrega.store') }}"
```
sin olvidarnos de estas tres lineas importantes
```php
method="POST"
enctype="multipart/form-data"
@csrf
````

En el controlador, la función **@store**
>App/Http/Controller/EntregaController 

Hacemos un nuevo objeto 'Entrega'
```php
$entrega=new Entrega();
```
Importampos con ```use App\Models\Entrega;```

Pasamos el valor del id de la tarea y del usuario
```php
$entrega->tarea_id=$request->tarea_id;
$entrega->user_id=auth()->user()->id;
```
Para el archivo es lo mismo que habíamos hecho en la sección de tareas
```php
$file = $request->file('file');
$username=auth()->user()->name;
$name=$username.$file->getClientOriginalName();
```
Importamos ```use Illuminate\Support\Facades\Auth;```

Hacemos nueva carpeta 
>Public/entregas

```php
$destination = public_path() . '/entregas/';
$file->move($destination, $name);
$entrega->file=$name;

$entrega->save();
return redirect('home');
```
## Gabriel
### Mostrar tabla de entregas para administrador

Las entregas se mostrarán en la vista particular de cada tarea por lo que modificaremos

En la función **@show**
>App/Http/Controller/TareaController 

Pasamos los datos de las entregas

```php
$tarea=Tarea::findOrFail($id);
$entregas=Entrega::all();
return view('admin.tareas.show', ['tarea'=>Tarea::findOrFail($id), 'entregas'=>$entregas]);
 ```
 
Importamos use App\Models\Entrega;

En la vista `tareas/show.blade.php` haremos un ciclo foreach para recorrer las entregas y al mismo tiempo las vamos a ir filtrando para unicamente ver las entregas relacionadas con esa tarea

```php
@foreach($entregas as $entrega)
@if($entrega->tarea_id == $tarea->id)
	<tr>
	</tr>
@endif
@endforeach
``` 
Y ponemos los valores correspondientes

```php
{{ $entrega->user->name }}
href="{{ asset("entregas/$entrega->file") }}"
{{ $entrega->file }}
{{ $entrega->comments }}
{{ $entrega->cal }}
````

#### Para subir las calificaciones
Dentro de  `tareas/show.blade` en el formulario ponemos la acción que será la funcion **@update**

```php
action="{{ route('entrega.update', $entrega->id) }}" method="POST"
@method('PATCH')
@csrf
````

En la función **'@update'**
>App/Http/Controller/EntregaController 

```php
$entrega=Entrega::findOrFail($id);
$entrega->cal=$request->cal;
$entrega->cal=$request->cal;
$entrega->update();
return back();
```` 

Para que se muestren las calificaciones ya guardadas
```php
<input type="number" min="0" max="10" name="cal" 
@if(!$entrega->cal==NULL)
value="{{ $entrega->cal }}" 
@else
value="0"
@endif
>
````

## Anallely
### Tabla de calificaciones para el usuario

Hacemos una nueva vista
>Resources/Views/cal.blade.php

Y pegamos el código de nuestro archivo `cal.html`
```php
@extends('layouts.app')
@section('content')
@endsection
```` 

Hacemos una nueva función en el controlador de Home 
>App/Http/Controller/HomeController 
```php
public function cal(){
	return view('cal');
}
````

Y hacemos la ruta en `web.php` pasandole un parámetro
```php
Route::get('cal', [App\Http\Controllers\HomeController::class, 'cal'])->name('cal');

```

En `app.blade.php` ponemos el link
```php
href="{{ route('cal') }}"
 ```

Y ahora pasamos a la vista los datos necesarios para que nos pueda mostrar las calificaciones

```php
$id=auth()->user()->id;
$entregas=Entrega::where('user_id', $id)->get();
return view('cal', ['entregas'=>$entregas, 'id'=>$id]);
````

*Importamos use App\Models\User; y use App\Models\Entrega;*

Ahora solo falta mostrar los datos en las vistas con un foreach

```php
@foreach($entregas as $entrega)
@if($entrega->user_id==$id)
@if(!$entrega->cal==NULL)
<tr>
</tr>
@endif
@endif
@endforeach
```

Y pasamos datos 
```php
{{ $entrega->tarea->title}}
{{$entrega->comments}}
{{ $entrega->cal}}
```` 

### Mostrar unicamente tareas pendientes

De todas las entregas que existen vamos a filtrar las que pertenecen al usuario autenticado

En ```HomeController.php```, en la funcion **index**

```php
$id=auth()->user()->id;
$entregas_user = DB::table('entregas')->where('user_id', $id)->get();
$bandera=0;
return view('home',['tareas'=>$tareas, 'entregas'=>$entregas, 'bandera'=>$bandera]);
````

https://laravel.com/docs/8.x/queries#retrieving-results

Para cada iteración del foreach de tareas verificamos si la tarea existe dentro de las entregas de ese usuario para decidir si se muestra la tarea o no.

```php
@foreach($entregas as $entrega)
@if($tarea->id==$entrega->tarea_id)
<p class="d-none">{{ $bandera=1 }}</p>
@endif
@endforeach
@if($bandera==0)
<div class="col mb-4">
</div>
@endif
@endforeach                  


> Written with [StackEdit](https://stackedit.io/).
