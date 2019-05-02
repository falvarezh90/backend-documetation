
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

Luego de crear la ruta del nuevo endpoint, se debe crear una prueba de unidad que testee los casos exitosos y de fracaso de la nueva funcionalidad. Un ejemplo de test se puede revisar la clase ```Modules\General\Tests\Unit\GUser\CrudUserTest``` donde se prueba un Crud de usuario (```GUser```). 

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

Por último, en el código anterior se puede se observar que la transacción finaliza con ```return CustomResponse::created($resp);```, que se refiere a la llamada de un método de la clase ```App\Http\Response\CustomResponse``` (creada por don José) que retorna un array. el cual es devuelto al usuario como respuesta a la solicitud. Los métodos de esta clase que se utilizan normalmente son ```ok``` para solicitudes exitosas, ```created``` para solicitudes que crean entidades en la base de datos, ```error``` y ```normalError``` para solicitudes con errores.

Dos puntos adicionales a tener en cuenta sobre los servicios, el primero se refiere a que los servicios deben estar alojados en una carpeta que tenga el nombre de la entidad o modelo al que se refiere el servicio (como con los test y request). El otro punto hace referencia a que si el servicio modifica en la base de datos muchos modelos, el servicio primcipal debería llamar a otros servicios dedicados a modificar estos otros modelos. Esto tiene como finalidad hacer a la aplicación más modular y legible.

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

Por ejemplo, hay 2 archivos de semillas para los usuarios: ```Modules\General\Database\Seeders\Development\GUserTableSeeder``` y  ```Modules\General\Database\Seeders\Production\GUserTableSeeder```, correspondiente a semillas de desarrollo y producción respectivamente.



## Librería para implementar la especificación JSON API

La librería para utilizar la especificación JSON API se llama ```laravel-query-builder``` y la documentación de dicha librería se puede encontrar en este [link](https://github.com/spatie/laravel-query-builder).

Esta librería se utiliza en la mayoría de los controladores del backend, específicamente en los métodos ```index``` y ```show```. Al revisar la clase ```Modules\Warehouse\Http\Controllers\Api\WhProductController``` se puede encontrar la implementación de los métodos de dicha librería

```
/**
* Display a listing of the resource.
*
* @param  \Illuminate\Http\Request  $request
* @return \Illuminate\Http\Response
*/
public function index(Request $request)
{
    $query =  QueryBuilder::for(WhProduct::where('wh_product.flag_delete', 0))
                            ->allowedIncludes('wh_item', 'wh_pack', 'wh_packing', 'wh_promo', 'wh_subfamily', 'audit_historic_prices',
                                                'pch_detail_purchase_debit_notes', 'pch_detail_purchase_invoices', 'pch_detail_purchase_credit_notes',
                                                'pch_detail_purchase_quotations', 'pos_detail_employee_sales', 'pos_detail_internal_consumptions',
                                                'pos_detail_sales', 'pos_manual_discounts', 'sl_change_sale_prices', 'sl_detail_list_prices.sl_list_price',
                                                'sl_detail_sale_credit_notes', 'sl_detail_sale_debit_notes', 'sl_detail_sale_invoices', 'sl_detail_sale_quotations',
                                                'sl_detail_sale_tickets', 'sl_offers', 'sl_wholesale_discounts', 'wh_detail_dispatch_guides', 'wh_detail_orderers',
                                                'wh_detail_product_receptions', 'wh_detail_sale_notes', 'wh_detail_transfer_between_warehouses', 'wh_packs',
                                                'wh_items', 'wh_packings', 'wh_promos', 'wh_stock_movements', 'wh_tags', 'wh_warehouses_primary',
                                                'wh_product_upc_code', 'wh_product_global_stocks')
                            ->allowedAppends('sale_prices', 'composition', 'stocks', 'discounts', 'virtual_offer_price', 'sale_price_by_branch_office',
                                                'branch_offices', 'critical_stock_by_warehouse', 'repacking_children', 'repacking_parents', 'purchase_orders')
                            ->allowedFilters(
                                Filter::custom('search', FiltersWhProductSearch::class),
                                Filter::custom('of_warehouse', FiltersWhProduct_OfWarehouse::class),
                                Filter::custom('critical_stock_by_warehouse', FiltersWhProductCriticalStock_ByWarehouse::class),
                                Filter::custom('wh_family_id', FiltersWhProductFamily::class),
                                Filter::custom('wh_families_id', FiltersWhProductFamilies::class),
                                Filter::custom('wh_subfamilies_id', FiltersWhProductSubFamilies::class),
                                Filter::custom('tags_id', FiltersWhProductTag::class),
                                Filter::custom('upc_code', FiltersWhProductUpcCode::class),
                                'internal_code',
                                'name',
                                Filter::exact('is_salable'),
                                Filter::exact('is_repackaged'),
                                Filter::exact('wh_subfamily_id'),
                                Filter::custom('type', FiltersWhProductType::class),
                                Filter::custom('types', FiltersWhProductTypes::class), // Ej: filter[types]=pack,promo  <-- return pack and promos, not item or packing
                                Filter::custom('is_consumable', FiltersWhProductIsConsumable::class), // Ej: filter[types]=pack,promo  <-- return pack and promos, not item or packing
                                Filter::custom('pch_provider_id', FiltersWhProductPchProvider::class) // Ej: filter[types]=pack,promo  <-- return pack and promos, not item or packing
                            )
                            ->defaultSort('updated_at')
                            ->allowedSorts(
                                'id',
                                'name',
                                'alias_name',
                                'internal_code',
                                'created_at',
                                'updated_at'
                                // Sort::custom('sale_price', WhProduct_SalePriceSort::class)
                            );
    ...

    if($request->get_all_results)
    {
        $response['data'] = $query->get();
        return $response;
    }
    else {
        $data = $query->paginate($request->limit);
        return $data;
    }
}
```

En el segmento de código anterior, se puede observar el uso de ```includes```, ```appends```, ```filters``` (algunos personalizados) y ```sorts```. En la documentación se puede obenter más orientación de cómo funcionan cada uno de estos métodos. 

A grandes rasgos, lo que hace esta librería es extender la clase ```QueryBuilder``` de Laravel para recibir y responder una request respetando la especificación JSON API.



## Documentación de la Base de Datos (Pendiente)

Este item quedará pendiente porque son muchas tablas, pero espero tenerlos antes del Lunes de la proxima semana (cualquier duda sobre cualquier cosa me la dejas en el Slack o le preguntas al José).



