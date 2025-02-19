---
title: module-xml-database-section
author: James Finstrom
date: February 19, 2025
---

# Database Table XML Structure

This document describes the XML structure added to the module.xml to manage databases

## Overview

The XML represents a single database table, including its columns, indexes, and keys. It is rooted in a `<database>` element and formatted for readability using `DOMDocument`. The XML declaration is stripped from the final output.

## XML Elements

### `<database>`

- **Description**: Root element of the XML document.
- **Attributes**:
  - `name` (optional): The database name, included if `$this->database` is not empty.
- **Children**: Contains one `<table>` element.

### `<table>`

- **Description**: Represents a database table.
- **Parent**: `<database>`
- **Attributes**:
  - `name`: The table name, sourced from `$args[0]`.
- **Children**: Contains `<field>` and `<key>` elements.

### `<field>`

- **Description**: Defines a column in the table.
- **Parent**: `<table>`
- **Attributes**:
  - `name`: Column name.
  - `type`: Data type (see supported types below).
  - `length` (optional): Column length (e.g., for `string`).
  - `default` (optional): Default value.
  - `unsigned` (optional): `true` or `false`.
  - `notnull` (optional): `true` or `false` (inverted from `getNotnull()`).
  - `primarykey` (optional): `true` or `false`.
  - `autoincrement` (optional): `true` or `false`.
- **Supported Types**:
  - `string`: Includes `length` attribute.
  - `blob`: No additional attributes.
  - `integer`: No additional attributes.
  - `bigint`: No additional attributes.
  - `smallint`: No additional attributes.
  - `date`: No additional attributes.
  - `datetime`: No additional attributes.
  - `text`: No additional attributes.
  - `boolean`: No additional attributes.
  - `float`: Includes `precision` and `scale` attributes.
  - `decimal`: Includes `precision` and `scale` attributes.
- **Notes**: 
  - Unsupported types throw an exception with "Unknown type" message.
  - `precision` and `scale` are specific to `float` and `decimal`.

### `<key>`

- **Description**: Defines an index or key.
- **Parent**: `<table>`
- **Attributes**:
  - `name`: Index name.
  - `type`: `unique` or `index`.
- **Children**: Contains `<column>` elements.
- **Notes**: Primary keys are marked on `<field>` elements, not as `<key>`.

### `<column>` (within `<key>`)

- **Description**: Specifies a column in an index or key.
- **Parent**: `<key>`
- **Attributes**:
  - `name`: Column name.

## Example

Here’s an example of the XML:

```xml
<database name="mydb">
    <table name="users">
        <field name="id" type="integer" primarykey="true" autoincrement="true" notnull="true"/>
        <field name="username" type="string" length="50" notnull="true"/>
        <field name="balance" type="decimal" precision="10" scale="2" default="0.00"/>
        <field name="is_active" type="boolean" default="true"/>
        <key name="idx_username" type="unique">
            <column name="username"/>
        </key>
    </table>
</database>
```
