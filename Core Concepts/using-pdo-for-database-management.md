---
title: Using PDO for Database Management
author: James Finstrom
date: 2025-01-28
---

# Managing Module Data Using PDO in FreePBX

This document demonstrates how to use PDO for database interactions within a FreePBX module. The `Mymodule` class uses PDO for database access, allowing efficient and secure operations.

## Basic Setup

The `Mymodule` class is constructed with the `$freepbx` object, which provides access to the BMO framework and a PDO instance for database operations. In the constructor, set `$this->FreePBX` to the BMO object and `$this->Database` to the PDO object:

```php
class Mymodule {
    protected $FreePBX;
    protected $Database;

    public function __construct($freepbx) {
        $this->FreePBX = $freepbx;
        $this->Database = $freepbx->Database;
    }
}
```

## Core Operations

### Insert Data

**Example:** Add a new user to a table `users` with columns `name` and `role`.

```php
public function addUser($name, $role) {
    $sql = "INSERT INTO users (name, role) VALUES (:name, :role)";
    $stmt = $this->Database->prepare($sql);
    $stmt->execute([':name' => $name, ':role' => $role]);
    return $this->Database->lastInsertId();
}
```

### Update Data

**Example:** Update a user's role in the `users` table by their ID.

```php
public function updateUserRole($userId, $newRole) {
    $sql = "UPDATE users SET role = :role WHERE id = :id";
    $stmt = $this->Database->prepare($sql);
    $stmt->execute([':role' => $newRole, ':id' => $userId]);
    return $stmt->rowCount();
}
```

### Delete Data

**Example:** Remove a user from the `users` table by their ID.

```php
public function deleteUser($userId) {
    $sql = "DELETE FROM users WHERE id = :id";
    $stmt = $this->Database->prepare($sql);
    $stmt->execute([':id' => $userId]);
    return $stmt->rowCount();
}
```

### Retrieve Data

#### Fetch a Single Record

**Example:** Get a user's details by their ID.

```php
public function getUserById($userId) {
    $sql = "SELECT * FROM users WHERE id = :id";
    $stmt = $this->Database->prepare($sql);
    $stmt->execute([':id' => $userId]);
    return $stmt->fetch(PDO::FETCH_ASSOC);
}
```

#### Fetch Multiple Records

**Example:** Get all users with the role `admin`.

```php
public function getAdminUsers() {
    $sql = "SELECT * FROM users WHERE role = :role";
    $stmt = $this->Database->prepare($sql);
    $stmt->execute([':role' => 'admin']);
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```

#### Count Records

**Example:** Count the number of users in the `users` table.

```php
public function getUserCount() {
    $sql = "SELECT COUNT(*) FROM users";
    $stmt = $this->Database->query($sql);
    return $stmt->fetchColumn();
}
```

## Best Practices

1. **Use Prepared Statements:** Always use prepared statements to prevent SQL injection.

    ```php
    $sql = "SELECT * FROM movies WHERE genre = :genre";
    $stmt = $this->Database->prepare($sql);
    $stmt->execute([':genre' => 'sci-fi']);
    ```

2. **Error Handling:** Use try-catch blocks to handle exceptions gracefully.

    ```php
    try {
        $sql = "INSERT INTO movies (title, director) VALUES (:title, :director)";
        $stmt = $this->Database->prepare($sql);
        $stmt->execute([':title' => 'Interstellar', ':director' => 'Christopher Nolan']);
    } catch (PDOException $e) {
        error_log('Database Error: ' . $e->getMessage());
    }
    ```

3. **Fetch Modes:** Use appropriate fetch modes (`PDO::FETCH_ASSOC`, `PDO::FETCH_OBJ`, etc.) to structure returned data.

    ```php
    $stmt->fetchAll(PDO::FETCH_OBJ);
    ```

4. **Use Consistent Naming:** Follow consistent naming conventions for variables and tables to improve code readability.

    ```php
    // Example:
    $sql = "SELECT * FROM heroes WHERE universe = :universe";
    ```
