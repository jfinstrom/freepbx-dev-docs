---
title: free-pbx-module-xml-structure
author: James Finstrom
date: February 19, 2025
---

# FreePBX Module XML Structure

This document outlines the base XML structure of a `module.xml` file. Note this file will contain other fields such as `<database>` and `<hooks>` which are NOT documented here.
## Overview

The `module.xml` file is a critical component of FreePBX modules, located in the module's directory (e.g., `/admin/modules/examplemodule/module.xml`). It uses a standardized XML format to define module details, which the `Modules` class processes through methods like `getXML`, `checkConflicts`, and `getInfo`. Below is the typical structure and its elements.

## XML Structure

Here’s an example of a `module.xml` file that the `Modules` class can parse:

```xml
<module>
    <rawname>examplemodule</rawname>
    <name>Example Module</name>
    <version>17.0.1</version>
    <publisher>Schmooze Com Inc.</publisher>
    <license>GPLv3</license>
  	<licenselink>http://www.gnu.org/licenses/gpl-3.0.txt</licenselink>
    <description>A sample FreePBX module for demonstration purposes.</description>
    <category>Utilities</category>
    <authentication>false</authentication>
    <depends>
        <module>core</module>
        <version>ge 17.0.1</version>
        <phpcomponent>IONCube</phpcomponent>
    </depends>
    <menuitems>
        <examplepage>Example Page</examplepage>
    </menuitems>
    <breaking>
        <module>
            <rawname>conflictingmodule</rawname>
            <type>conflict</type>
            <version>lt 15.0.1</version>
            <errormessage>RAW_NAME conflicts with this module.</errormessage>
        </module>
        <module>
            <rawname>oldmodule</rawname>
            <type>deprecated</type>
            <version>eq 15.0.1</version>
            <replace>newmodule</replace>
            <replaceerrormessage>RAW_NAME is replaced by REPLACE_NAME.</replaceerrormessage>
            <versionerrormessage>RAW_NAME version mismatch.</versionerrormessage>
        </module>
    </breaking>
</module>
```

### Element Breakdown

1. **Root Element: `<module>`**
   - The top-level container for all module metadata and configuration.

2. **Metadata Elements**
   - `<rawname>`: Machine-readable module name (e.g., `examplemodule`).
   - `<name>`: Human-readable module name (e.g., "Example Module").
   - `<version>`: Module version (e.g., `17.0.1`).
   - `<publisher>`: Entity responsible for the module (e.g., "ACME Inc.").
   - `<license>`: Licensing information (e.g., "GPLv3").
   - `<licenselink>`: Link to license text
   - `<description>`: Brief description of the module’s purpose.
   - `<category>`: Organizational category in the FreePBX UI (e.g., "Utilities").
   - `<authentication>`: Optional boolean flag indicating if authentication is required (used in `getActiveModules`).

3. **Dependencies: `<depends>`**
   - `<module>`: Name of a required module (e.g., `core`).
   - `<version>`: Version constraint (e.g., `ge 17.0.1` for "greater than or equal to 17.0.1").
   - `<phpcomponent>`: Specifies additional PHP components (e.g., `zend`), checked in `loadFunctionsInc`.

4. **Menu Items: `<menuitems>`**
   - Defines GUI navigation items (e.g., `<examplepage>`), mapping a display ID to a name. Used in `getClassName` to associate pages with modules.

5. **Conflicts: `<breaking>`**
   - Contains one or more `<module>` sub-elements for conflict or deprecation definitions (parsed in `checkConflicts`).
     - `<rawname>`: Name of the conflicting module.
     - `<type>`: Either `conflict` or `deprecated`.
     - `<version>`: Version condition (e.g., `lt 17.0` for "less than 17.0").
     - `<errormessage>`: Custom error message for the conflict.
     - `<replace>`: Replacement module for deprecated cases.
     - `<replaceerrormessage>`: Message suggesting the replacement.
     - `<versionerrormessage>`: Message for version-specific issues.

## How the Modules Class Uses This XML

The `Modules` class interacts with `module.xml` in several key ways:

- **Loading XML (`getXML`)**
  - Loads the file from the module directory into a SimpleXML object and caches it in `$this->modulexml`.

- **Conflict Checking (`checkConflicts`)**
  - Parses `<breaking>` to identify conflicts or deprecations, supporting version comparisons (e.g., `lt`, `ge`) and custom error messages.

- **Module Information (`getInfo`)**
  - Relies on `module_functions::getinfo` to extract `<rawname>`, `<version>`, and runtime status.

- **Active Modules (`getActiveModules`)**
  - Uses `<authentication>` to filter modules when the session is unauthenticated.

- **Function Loading (`loadFunctionsInc`)**
  - Checks `<depends><phpcomponent>` for requirements like Zend.

## Usage Example

To test a module’s XML parsing:

1. Create a `module.xml` file in `/admin/modules/testmodule/`.
2. Add the structure above, replacing values as needed.
3. Use the `Modules` class in PHP:

```php
$freepbx = FreePBX::create();
$modules = new FreePBX\Modules($freepbx);
$xml = $modules->getXML('testmodule');
echo $xml->name; // Outputs: "Example Module"
```
