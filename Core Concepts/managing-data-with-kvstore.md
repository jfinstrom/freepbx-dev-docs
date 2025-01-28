---
title: Managing Module Configurations with KVStore (FreePBX_Helpers) in FreePBX
author: James Finstrom
date: 2025-01-28
---

# Managing Module Data with FreePBX_Helpers in FreePBX

This document explains how to use the `kvstore` methods in a class extending `FreePBX_Helpers` from the FreePBX framework. These methods simplify configuration management for modules by providing structured and efficient ways to store, retrieve, and manage data.

## Overview

To leverage the `kvstore` functionality, your class must extend `FreePBX_Helpers`. This allows access to powerful configuration management methods for handling both individual and batch operations.

## Basic Setup

Create a class that extends `FreePBX_Helpers` to access configuration management methods:

```php
class Mymodule extends \FreePBX_Helpers implements \BMO {
    // Your configuration logic here
}
```

## Core Methods

### `getConfig($var, $id = 'noid', $cache = true)`

Retrieve a stored configuration value with optional subgrouping.

**Parameters:**

-   `$var`: Configuration key (required)
-   `$id`: Sub-group identifier (default: `"noid"`)
-   `$cache`: Whether to use cached values (default: `true`)

**Example:**

```php
public function getTimezoneSetting() {
    return $this->getConfig('timezone', 'server_settings');
}
```

### `setConfig($key, $val, $id = 'noid')`

Store a configuration value with optional subgrouping.

**Parameters:**

-   `$key`: Configuration key (required)
-   `$val`: Value to store (use `false` to delete the key)
-   `$id`: Sub-group identifier (default: `"noid"`)

**Example:**

```php
public function updateUserLimit($limit) {
    return $this->setConfig('max_users', $limit, 'license_settings');
}
```

### `delConfig($key, $id = 'noid')`

Delete a configuration entry.

**Example:**

```php
public function removeOldFeatureFlag() {
    return $this->delConfig('legacy_feature', 'experimental');
}
```

## Batch Operations

### `setMultiConfig($keyval, $id = 'noid')`

Store multiple values in a transaction.

**Example:**

```php
public function initializeSettings() {
    $settings = [
        'welcome_message' => 'Hello World',
        'max_attempts' => 5,
        'retry_interval' => 300
    ];
    return $this->setMultiConfig($settings, 'auth_settings');
}
```

## Data Retrieval Methods

### `getAll($id = 'noid')`

Retrieve all configurations for a subgroup.

**Example:**

```php
public function getAllLicenseSettings() {
    return $this->getAll('license_settings');
}
```

### `getAllKeys($id = 'noid')`

List all keys for a subgroup.

**Example:**

```php
public function getActiveFeatures() {
    return $this->getAllKeys('enabled_features');
}
```

### `getAllids()`

List all existing subgroup IDs.

**Example:**

```php
public function getConfiguredSubgroups() {
    return $this->getAllids();
}
```

## Advanced Operations

### `getFirst($id = 'noid')` / `getLast($id = 'noid')`

Retrieve the first or last entry in a subgroup.

**Example:**

```php
public function getLatestErrorLog() {
    return $this->getLast('error_logs');
}
```

### `delById($id)`

Delete all entries in a subgroup.

**Example:**

```php
public function cleanupTempData() {
    return $this->delById('temp_cache');
}
```

### `deleteAll()`

Remove all configurations and drop the table.

**Example:**

```php
public function uninstall() {
    $this->deleteAll();
}
```

## Best Practices

1. **Subgroup Organization:** Use meaningful IDs for different configuration types to improve maintainability.

    ```php
    $this->setConfig('max_connections', 100, 'database_settings');
    ```

2. **Batch Updates:** Use `setMultiConfig` for efficient multiple writes.

    ```php
    $this->setMultiConfig(['color' => 'blue', 'size' => 'large'], 'ui_settings');
    ```

3. **Caching:** Leverage built-in caching to reduce database queries where appropriate.

    ```php
    // Fetch fresh data from the database, bypassing the cache
    $this->getConfig('system_version', 'noid', false);
    ```

4. **Data Types:** Complex data structures like arrays are automatically serialized for storage.

    ```php
    $this->setConfig('preferences', ['dark_mode' => true, 'notifications' => false]);
    ```

**Important Note:** The `noid` group is used for root-level configurations. Use distinct subgroup IDs to organize related settings while maintaining compatibility with the table structure.
