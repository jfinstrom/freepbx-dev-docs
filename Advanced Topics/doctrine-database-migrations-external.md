---
title: FreePBX Doctrine Database Management
author: James Finstrom
date: 2025-07-23
---


# Using Doctrine in FreePBX for Database Table Management

This document explains how to use Doctrine functionality within FreePBX to manage database tables. It covers connecting to a database, defining table structures with property types, and performing migrations. Note that `/etc/freepbx.conf` is used for the FreePBX autoloader in standalone scripts but is unnecessary within FreePBX functions or classes.

## Overview

FreePBX uses Doctrine, an Object-Relational Mapping (ORM) tool, to manage database tables. The `FreePBX\Database` class simplifies database connections and schema migrations.

## Connecting to the Database

Initialize the `FreePBX\Database` class with a DSNSW, username, and password.

```php
$host = 'localhost';
$port = 3306;
$database = 'asterisk';
$user = 'freepbxuser';
$password = 'your_secure_password';
$dsn = sprintf('mysql:host=%s;port=%s;dbname=%s', $host, $port, $database);
$MyDatabase = new FreePBX\Database($dsn, $user, $password);
```

- **DSN Format**: `mysql:host=<host>;port=<port>;dbname=<database>`.
- **Credentials**: Use secure `$user` and `$password`.
- **Autoloader**: Include `/etc/freepbx.conf` for standalone scripts to load the FreePBX autoloader; omit within FreePBX functions or classes.

## Defining Table Structures

Tables are defined as an associative array with `columns` and optional `indexes`. Each column specifies properties and their types.

### Table Definition Examples

Below are two example tables: one with various data types (`example_table`) and a real-world use case (`call_logs`) for storing call detail records in a FreePBX system.

#### Example Table: `example_table`

```php
$tables = [
  'example_table' => [
    'columns' => [
      'id' => [
        'type' => 'integer',          // Integer
        'autoincrement' => true,      // Boolean
        'unsigned' => true,           // Boolean
        'notnull' => true,            // Boolean
        'primarykey' => true,         // Boolean
      ],
      'string_col' => [
        'type' => 'string',           // String
        'length' => 100,              // Integer
        'notnull' => false,           // Boolean
        'default' => null,            // Mixed (null, string, number)
      ],
      'integer_col' => [
        'type' => 'integer',          // Integer
        'notnull' => false,           // Boolean
      ],
      'bigint_col' => [
        'type' => 'bigint',           // Integer
        'notnull' => false,           // Boolean
      ],
      'smallint_col' => [
        'type' => 'smallint',         // Integer
        'notnull' => false,           // Boolean
      ],
      'boolean_col' => [
        'type' => 'boolean',          // Boolean
        'notnull' => false,           // Boolean
        'default' => 0,               // Integer (0 or 1)
      ],
      'date_col' => [
        'type' => 'date',             // String (YYYY-MM-DD)
        'notnull' => false,           // Boolean
      ],
      'datetime_col' => [
        'type' => 'datetime',         // String (YYYY-MM-DD HH:MM:SS)
        'notnull' => false,           // Boolean
      ],
      'float_col' => [
        'type' => 'float',            // Float
        'precision' => 10,            // Integer
        'scale' => 2,                 // Integer
        'notnull' => false,           // Boolean
      ],
      'decimal_col' => [
        'type' => 'decimal',          // String
        'precision' => 12,            // Integer
        'scale' => 4,                 // Integer
        'notnull' => false,           // Boolean
      ],
      'text_col' => [
        'type' => 'text',             // String
        'notnull' => false,           // Boolean
      ],
      'blob_col' => [
        'type' => 'blob',             // Binary
        'notnull' => false,           // Boolean
      ],
    ],
    'indexes' => [
      'string_col_idx' => [
        'type' => 'index',            // String
        'cols' => ['string_col'],     // Array of strings
      ],
      'unique_text_col' => [
        'type' => 'unique',           // String
        'cols' => ['text_col'],       // Array of strings
      ],
    ],
  ],
  'call_logs' => [
    'columns' => [
      'call_id' => [
        'type' => 'integer',          // Integer
        'autoincrement' => true,      // Boolean
        'unsigned' => true,           // Boolean
        'notnull' => true,            // Boolean
        'primarykey' => true,         // Boolean
      ],
      'caller_id' => [
        'type' => 'string',           // String
        'length' => 20,               // Integer
        'notnull' => true,            // Boolean
        'default' => '',              // String
      ],
      'destination' => [
        'type' => 'string',           // String
        'length' => 20,               // Integer
        'notnull' => true,            // Boolean
        'default' => '',              // String
      ],
      'call_start' => [
        'type' => 'datetime',         // String (YYYY-MM-DD HH:MM:SS)
        'notnull' => true,            // Boolean
      ],
      'duration' => [
        'type' => 'integer',          // Integer (seconds)
        'unsigned' => true,           // Boolean
        'notnull' => true,            // Boolean
        'default' => 0,               // Integer
      ],
      'call_status' => [
        'type' => 'string',           // String (e.g., "ANSWERED", "NOANSWER")
        'length' => 20,               // Integer
        'notnull' => true,            // Boolean
        'default' => 'NOANSWER',      // String
      ],
      'recording' => [
        'type' => 'blob',             // Binary (call recording data)
        'notnull' => false,           // Boolean
      ],
    ],
    'indexes' => [
      'caller_id_idx' => [
        'type' => 'index',            // String
        'cols' => ['caller_id'],      // Array of strings
      ],
      'call_start_idx' => [
        'type' => 'index',            // String
        'cols' => ['call_start'],     // Array of strings
      ],
    ],
  ],
];
```

### Column Properties and Types

- **type**: Data type (`integer`, `string`, `bigint`, `smallint`, `boolean`, `date`, `datetime`, `float`, `decimal`, `text`, `blob`) – String.
- **autoincrement**: Enable auto-increment for primary keys – Boolean.
- **unsigned**: Enforce non-negative values for numeric types – Boolean.
- **notnull**: Require non-null values – Boolean.
- **primarykey**: Designate as primary key – Boolean.
- **length**: Max length for `string` types – Integer.
- **precision**: Total digits for `float` or `decimal` – Integer.
- **scale**: Decimal places for `float` or `decimal` – Integer.
- **default**: Default value (e.g., `null`, `0`, string, number) – Mixed.

### Index Properties and Types

- **type**: Index type (`index`, `unique`) – String.
- **cols**: Columns in the index – Array of strings.

## Performing Migrations

Use the `migrate` method to manage schema changes via `modifyMultiple`.

```php
$dryrun = false;
$migrate = $MyDatabase->migrate('nonemptystring');
$migrate->modifyMultiple($tables, $dryrun);
```

- **migrate('nonemptystring')**: Initializes migration with a string identifier – String.
- **modifyMultiple($tables, $dryrun)**: Applies table definitions.
  - `$tables`: Table definitions – Array.
  - `$dryrun`: Simulate changes if `true` – Boolean.
- **Source**: See `Migration.class.php` in FreePBX framework.

## Supported Data Types

- `integer`: Whole numbers – Integer.
- `string`: Variable-length strings – String.
- `bigint`: Large integers – Integer.
- `smallint`: Small integers – Integer.
- `boolean`: True/false (0/1) – Boolean.
- `date`: YYYY-MM-DD – String.
- `datetime`: YYYY-MM-DD HH:MM:SS – String.
- `float`: Floating-point numbers – Float.
- `decimal`: Fixed-point numbers – String.
- `text`: Large text – String.
- `blob`: Binary data – Binary.

## Usage Example

To create or update `call_logs`:

```php
$MyDatabase = new FreePBX\Database($dsn, $user, $password);
$dryrun = false;
$migrate = $MyDatabase->migrate('call_log_migration');
$migrate->modifyMultiple($tables, $dryrun);
```

- Use `$dryrun = true` to preview changes.
- Omit `/etc/freepbx.conf` in FreePBX functions or classes.

## Best Practices

- **Secure Credentials**: Avoid hardcoded passwords.
- **Dry Run**: Test with `$dryrun = true`.
- **Validate Definitions**: Match column types to database requirements.
- **Error Handling**: Use try-catch for connection/migration errors.
- **Documentation**: Refer to `Migration.class.php` for advanced options.

