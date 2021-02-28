---
title: Building Tables
description:
extends: _layouts.documentation
section: content
---

# Building Tables

- [Columns](#columns)
  - [Text](#fields-text)
- [Filters](#filters)
- [Context Customization](#context-customization)

Filament includes a table builder which can be used to create interactive tables in the admin panel.

Tables have [columns](#columns) and [filters](#filters), which are defined in two methods on the table object.

Here is an example table configuration for a `CustomerResource`:
```php
use Filament\Resources\Tables\Columns;
use Filament\Resources\Tables\Filter;
use Filament\Resources\Tables\Table;

public static function table(Table $table)
{
    return $table
        ->columns([
            Columns\Text::make('name')->primary(),
            Columns\Text::make('email')->url(fn ($customer) => "mailto:{$customer->email}"),
            Columns\Text::make('type')
                ->options([
                    'individual' => 'Individual',
                    'organization' => 'Organization',
                ]),
            Columns\Text::make('birthday')->date(),
        ])
        ->filters([
            Filter::make('individuals', fn ($query) => $query->where('type', 'individual')),
            Filter::make('organizations', fn ($query) => $query->where('type', 'organization')),
        ]);
}
```

## Columns {#columns}

Resource column classes are located in the `Filament\Resources\Tables\Columns` namespace.

All columns have access to the following customization methods:

```php
Column::make($name)
    ->getValueUsing($callback = fn ($record) => $record->getAttribute('{column name}')) // Set the callback used to retrieve the value of the column from a given record.
    ->label($label) // Set custom label text for with the column header, which is otherwise automatically generated based on its name. It supports localization strings.
    ->primary() // Sets the column as primary, which emphasises it and links to access a record.
    ->searchable() // Allows the values in this column to be searched.
    ->sortable(); // Allows the values in this column to be sorted.
```

### Text {#columns-text}

```php
Text::make($name)
    ->action($action) // Set Livewire action that should be called when this column is clicked. The current record key will be passed in as a parameter.
    ->currency($symbol = '$', $decimalSeparator = '.', $thousandsSeparator = ',') // Format values in this column as if they were currency.
    ->date($format = 'F j, Y') // Format values in this column as dates, using PHP date formatting tokens.
    ->dateTime($format = 'F j, Y H:i:s') // Format values in this column as date-times, using PHP date formatting tokens.
    ->default() // Set the default value for when this field does not exist.
    ->formatUsing($callback = fn ($value) => $value) // Set the callback used to format the value of the column.
    ->options($options = []) // Set the key-value array of available values that this column could hold.
    ->url($url); // Set URL callback that should be used to generate a URL to send the user to when this column is clicked.
```

> Support for other column types is coming soon. For more information, please see our [Development Roadmap](/docs/roadmap).

## Filters {#filters}

Filters are used to scope results in the table. Here is an example of a filter at allows only customers with a `type` of `individual` to be shown in the table:

```php
Filter::make('individuals', fn ($query) => $query->where('type', 'individual')),
```

They have access to the following customization options:

```php
Filter::make($name, $callback = fn ($query) => $query)
    ->label($label); // Set custom label text for with the filter, which is otherwise automatically generated based on its name. It supports localization strings.
```

> Currently, filters are static and only one may be applied at a time. Parameter-based filters and support for applying multiple filters at once is coming soon. For more information, please see our [Development Roadmap](/docs/roadmap).

## Context Customization {#context-customization} {#layout}

You may customize tables based on the page they are used. To do this, you can chain the `only()` or `except()` methods onto any column or filter.
```php
use App\Filament\Resources\CustomerResource\Pages;
use Filament\Resources\Tables\Filter;
use Filament\Resources\Tables\Table;

public static function table(Table $table)
{
    return $table
        ->filters([
            Filter::make('individuals', fn ($customer) => $customer->type === 'individual')
                ->only(Pages\ListCustomers::class),
        ]);
}
```
In this example, the `individuals` filter will `only()` be available on the `ListCustomers` page.

```php
use App\Filament\Resources\CustomerResource\Pages;
use Filament\Resources\Tables\Columns;
use Filament\Resources\Tables\Table;

public static function table(Table $table)
{
    return $table
        ->columns([
            Columns\Text::make('name')
                ->except(Pages\ListCustomers::class, fn ($column) => $column->primary()),
        ]);
}
```
In this example, the `name` column will be primary, `except()` on the `ListCustomers` page.

This is an incredibly powerful pattern, and allows you to completely customize a table contextually by chaining as many methods as you wish to the callback.