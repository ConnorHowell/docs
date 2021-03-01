---
title: Resources
description:
extends: _layouts.documentation
section: content
toc: |
  - [Forms](#forms)
    - [Relations](#relations)
    - [Belongs To](#relations-belongs-to)
  - [Has Many](#relations-has-many)
  - [Belongs to Many](#relations-belongs-to-many)
  - [Tables](#tables)
  - [Authorization](#authorization)
  - [Pages](#pages)
    - [Customizing Default Pages](#pages-customization)
    - [Custom Pages](#pages-custom)
---

# Resources

<p class="lg:text-2xl">Resources are static classes that describe how administrators should be able to interact with data from your app. They are associated with Eloquent models from your app.</p>

To create a resource for the `App\Models\Customer` model:

```bash
php artisan make:filament-resource Customer
```

This will create several files in the `app/Filament/Resources` directory:

```
.
+-- CustomerResource.php
+-- CustomerResource
|   +-- Pages
|   |   +-- CreateCustomer.php
|   |   +-- EditCustomer.php
|   |   +-- ListCustomers.php
```

Your new resource class lives in `CustomerResource.php`. Resource classes register [forms](#forms), [tables](#tables), [authorization settings](#authorization), and [pages](#pages) associated with that model.

The classes in the `Pages` directory are used to customize the pages in the admin panel that interact with your resource.

By default, the model associated with your resource is guessed based on the class name of the resource. You may set the static `$model` property to disable this behaviour:

```php
public static $model = Customer::class;
```

A label for this resource is generated based on the name of the resource's model. It's used the navigation menu and to display breadcrumbs. You may customise it using the static `$label` property:

```php
public static $label = 'customer';
```

## Forms {#forms}

Resource classes contain a static `form()` method that is used to customize the forms to create and update resource records.

```php
use Filament\Resources\Forms\Form;

public static function form(Form $form)
{
    return $form
        ->schema([
            // ...
        ]);
}
```

The `schema()` method is used to define the structure of your form. It is an array of components, in the order they should appear in your form.

For more information, please see the page on [Building Forms](/docs/forms).

## Relations {#relations}

### Belongs To {#relations-belongs-to}

The `Filament\Resources\Forms\Components\BelongsToSelect` field can be used in resource form schemas to create a select element with options to search and select a related record. It has the same methods available as [`Filament\Resources\Forms\Components\Select`](/docs/forms#fields-select), and others to define the relationship and column name that should be used:

```php
Components\BelongsToSelect::make('category_id')
    ->relationship('category', 'name');
```

This example assumes the following:

- A `category_id` foreign key column exists on your parent model.
- A `belongsTo()` `category` relationship on your parent model.
- A `name` column on your category model that can be used to render the list of categories to select from.

You may customize the [Query Builder](https://laravel.com/docs/queries) used to get search results by specifying a callback in the third parameter of the `relationship()` method.

```php
Components\BelongsToSelect::make('category_id')
    ->relationship('category', 'name', function ($query) {
        return $query->where('is_featured', true);
    });
```

This example will only include featured categories in search results.

### Has Many {#relations-has-many}

Relation managers are components that allow administrators to list, create, and edit related records without leaving the parent record's edit page. Resource classes contain a static `relations()` method that is used to register relation managers for your resource.

To create a relation manager, you can use:

```bash
php artisan make:filament-relation-manager CustomerResource orders
```

This will create a `app/Filament/Resources/CustomerResource/RelationManagers/OrdersRelationManager.php` file. This contains a class where you are able to define a [form](/docs/forms) and [table](/docs/tables) for your relation manager. The relation manager will interact with the `orders` relationship on your parent model.

You must register the new relation manager in your resource's `relations()` method:

```php
public static function relations()
{
    return [
        RelationManagers\OrdersRelationManager::class,
    ];
}
```

Once a table and form have been defined for the relation manager, visit the edit page of your resource to see it in action.

### Belongs to Many {#relations-belongs-to-many}

There is currently partial support for `belongsToMany()` relationships. You may use a [relation manager](#relations-has-many) to list, create, and edit related records. Soon, Filament will allow you to attach existing records to a relationship, and detach them without deleting the related record altogether. For more information, please see our [Development Roadmap](/docs/roadmap).

## Tables {#tables}

Resource classes contain a static `table()` method that is used to customize the table to list resource records.

```php
use Filament\Resources\Tables\Table;

public static function table(Table $table)
{
    return $table
        ->columns([
            // ...
        ])
        ->filters([
            // ...
        ]);
}
```

The `columns()` method is used to define the columns in your table. It is an array of column objects, in the order they should appear in your table.

Filters are predefined scopes that administrators can use to filter records in your table. The `filters()` method is used to register these.

For more information, please see the page on [Building Tables](/docs/tables).

## Authorization {#authorization}

You may create roles for users of Filament that allow them to access specific resources. You may create a `Manager` role using:

```php
art make:filament-role Manager
```

Administrators will now be able to assign this role to any Filament user using the admin panel.

To only allow users with the `Manager` role to access this resource, declare so in the static `authorization()` method:

```php
use App\Filament\Roles;

public static function authorization()
{
    return [
        Roles\Manager::allow(),
    ];
}
```

You may authorize as many roles as you wish.

> Please note: administrators will always have full access to every resource in your admin panel.

You may want to only deny users with the `Manager` role from accessing this resource. To do this, you may use the static `deny()` method instead:

```php
use App\Filament\Roles;

public static function authorization()
{
    return [
        Roles\Manager::deny(),
    ];
}
```

> Action-specific role authorization is coming soon. For more information, please see our [Development Roadmap](/docs/roadmap).

## Pages {#pages}

Pages are classes that are associated with a resource. They are essentially [Laravel Livewire](https://laravel-livewire.com) components with custom integration utilities for use with Filament.

Page class files are in the `/Pages` directory of your resource directory.

By default, resources are generated with three pages:

- List has a [table](#tables) for displaying, searching and deleting resource records. From here, you are able to access the create and edit pages. It is routed to `/`.
- Create has a [form](#forms) that is able to create a resource record. It is routed to `/create`.
- Edit has a [form](#forms) that is able to update a resource record, along with the [relation managers](#relations-has-many) registered to your resource. It is routed to `/{record}/edit`.

### Customizing Default Pages {#pages-customization}

You are able to customize text used in the default pages by overriding properties on the page class. To see the options available, check the static properties defined in the parent class of each default page.

For further customization opportunities, you can override the static `$view` property on your page to a custom view in your app:

```php
public static $view = 'customers.list-records';
```

### Custom Pages {#pages-custom}

Filament allows you to create completely custom pages for resources. To create a new page, you can use:

```bash
php artisan make:filament-page SortCustomers --resource=CustomerResource
```

This command will create two files - a page class in the `/Pages` directory of your resource directory, and a view in the `/pages` directory of the resource views directory.

You must register custom pages to a route in the static `routes()` method of your resource:

```php
public static function routes()
{
    return [
        // ...
        Pages\SortCustomers::routeTo('/sort', 'sort'),
    ];
}
```

The first parameter of the `routeTo()` method is the path of the route, and the second is its [name](https://laravel.com/docs/routing#named-routes). Any [parameters](https://laravel.com/docs/routing#route-parameters) defined in the route's path will be available to the page class, in an identical way to [Livewire](https://laravel-livewire.com/docs/rendering-components#route-params).

To generate a URL for a resource route, you may call the static `generateUrl()` method on the page class:

```php
SortCustomers::generateUrl($parameters = [], $absolute = true);
```
