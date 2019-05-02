
## Documentación Backend

El backend está estructurado a través de módulos, los cuales fueron implementados con la librería ```laravel-modules```. Los módulos son los siguientes:

- Accounting (Contabilidad)
- Auth (Autenticación)
- General (Modules Generales utilizados a través de toda la aplicación, como por ejemplo, GUser, GMenu, etc.)
- HR (Recursos Humanos)
- Pos (Punto de venta)
- Purchase (Adquisiciones)
- Sale (Ventas)
- Warehouse (Bodega)

Para crear Archivos de migraciones, semillas, notificación, comandos, etc. Se debe utilizar comandos artisan que vienen en la librería  ```laravel-modules``` especificando el módulo en el que quieres crear el archivo (es decir, que no se deben utilizar los comandos por defecto de laravel).

En este [enlace](https://nwidart.com/laravel-modules/v4/advanced-tools/artisan-commands) se listan los comandos de la librería ```laravel-modules```. 

## Implementación de endpoints para la API

### 1.- Agregar Ruta

Para agregar un nuevo endpoint, lo primero que se debe hacer es agregar una ruta en el archivo ```Modules\name_module\Routes\api.php```, donde ```name_module``` corresponde al módulo al que pertenece el nuevo endpoint (Accounting, Auth, General, Pos, Purchase, Sale, Warehouse).


### 2.- Crear Test

Luego de crear la ruta del nuevo endpoint, se debe crear una prueba de unidad que testee los casos exitosos y de fracaso de la nueva funcionalidad. Un ejemplo de test se puede revisar la clase ```Modules\General\Tests\Unit\GUser\CrudUserTest``` dónde se prueba un Crud de usuario (```GUser```). 

Tener en cuenta que los test se crean en el directorio correspondiente al módulo con el que se está trabajando, por ejemplo, en el caso anterior se está trabajando en el módulo ```General```. Otro punto a tener en cuenta, es que el test se debe alojar en un directorio con el nombre de la entidad con la que se está trabajando, por ejemplo, en el caso anterior el test está alojado en el directorio ```GUser``` por tratarse de un crud de esa entidad o modelo.

### 3.- Crear Request (Opcional)

Si el endpoint tiene un formato especial para los parámetros de entrada, se debe crear una clase de tipo ```Request``` para restringir las solicitudes con formato incorrecto. Un ejemplo de clase Request podría ser ```Modules\General\Http\Requests\GUser\CreateGUserRequest```. Al igual que con los test, la clase ```Request``` debe estar alojado en un directorio con el nombre de la entidad con la que se está trabajando, que en el caso anterior, sería una carpeta con el nombre ```GUser```.


### 4.- Crear Servicio

Luego de pasar la restricción que impone la clase ```Request``` a las solicitudes entrantes, el código debe llamar a la una clase que implemente la lógica que se desea desarrollar, a esta clase se la denomina ```servicio```. Por ejemplo, para el crud del modelo ```GUser``` se creó la clase ```Modules\General\Services\GUser\CrudUserService```. 

Si la lógica a desarrollar realiza actualizaciones en la Base de Datos, las funciones que realizan estas actualizaciones deberían estar encerradas en una transacción. El código de la transacción debe ser agregado en el controlador (no en el servicio como se hacía anteriormente). Por ejemplo, en el controlador ```Modules\General\Http\Controllers\Api\GUserController``` en el método ```store``` se puede observar que la función para almacenar un usuario está envuelta en una ```transacción```:

```
/**
 * Store a newly created resource in storage.
 *
 * @param  CreateGUserRequest  $request
 * @return \Illuminate\Http\Response
 */
public function store(CreateGUserRequest $request)
{
    $response = DB::transaction(function() use (&$request) {
        $resp = $this->crudUserService->store($request);
        return CustomResponse::created($resp);
    });
    return response()->json($response, $response['http_status_code']);
}
```

Por último, en el código anterior se puede se observa que la transacción finaliza con ```return CustomResponse::created($resp);```, que se refiere a la llamada de un método de la clase ```App\Http\Response\CustomResponse``` (creada por don José) que retorna un array. el cual es devuelta al usuario (frontend) como respuesta a la solicitud. Los métodos de esta clase que se utilizan normalmente son ```ok``` para solicitudes exitosas, ```created``` para solicitudes que crean entidades en la base de datos, ```error``` y ```normalError``` para solicitudes con errores.

### 5.- Pasar test de Prueba unitaria implementada en (2)

Luego de crear las clases necesarias para implementar la lógica del endpoint, se debe ejecutar las purebas unitarias implementadas y ajustar el código hasta pasar todas las pruebas desarrolladas. Para ejecutar un solo test (y no todos los test del backend), se utiliza el siguiente comando:

```vendor\bin\phpunit --filter MyTest```

Reemplazando ```MyTest``` por el nombre del test correspondiente.

### Resumen

Por lo tanto, la creación de un endpoint en el backend se podría resumir en los siguientes pasos:

1. Agregar ruta en el archivo ```api.php``` (del módulo en el cual se está trabajando)
2. Crear Clase ```Request``` (opcional)
3. Crear ```Servicio``` (encerrar la lógica en una transacción en caso de ser necesario)
4. Ajustar la lógica desarrollada hasta pasar todos los test implementados en ```(2)```

### Información adicional sobre las migraciones

Actualmente el backend tiene 2 carpetas con semillas en cada módulo, una con semillas de ```desarrollo``` y otra con semillas de ```producción```. Esto implica que si tienes que crear una archivo de semillas, tienes que crear una en cada carpeta, lo mismo ocurre si tienes que modificar una semilla existente, ya que tienes que modificar la semilla en cada directorio.

Por ejemplo, hay 2 archivos de semillas para los usuarios: ```Modules\General\Database\Seeders\Development\GUserTableSeeder``` y  ```Modules\General\Database\Seeders\Production\GUserTableSeeder``,` correspondiente semillas de desarrollo y producción.

## Documentación de la Base de Datos (Pendiente)

Este item quedará pendiente porque son muchas tablas, pero espero tenerlos antes del Lunes de la proxima semana (cualquier duda sobre cualquier cosa me la dejas en el Slack o le preguntas al José).



