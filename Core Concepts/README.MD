# Core Concepts

This directory contains documentation on fundamental concepts related to FreePBX module development.

## Architecture and Bootstrap

- [Bootstrap Chain and Global State](bootstrap-and-global-state.md)
- [BMO Architecture — Big Module Object](bmo-architecture.md)
- [Legacy functions.inc.php Coexistence with BMO](legacy-functions-inc.md)
- [Module Lifecycle — Install, Upgrade, Uninstall](module-lifecycle.md)

## Module Structure and Metadata

- [module.xml Manifest Structure](module-xml-manifest.md)
- [module.xml Database Section](module-xml-database-section.md)
- [FreePBX Module XML Structure](module-xml-file.MD) *(legacy overview)*
- [BMO Class Interface — Modulename.class.php](bmo-class-interface.md)
- [Naming Conventions](naming-conventions.md)
- [Code Style Conventions](code-style-conventions.md)

## Data Storage

- [KVStore Configuration Storage](kvstore-configuration.md)
- [Managing Module Data with KVStore](managing-data-with-kvstore.md) 
- [Database PDO Patterns](database-pdo-patterns.md)
- [Using PDO for Database Management](using-pdo-for-database-management.md) 

## Request Handling and GUI

- [Request_Helper — $_REQUEST Processing](request-helper.md)
- [GUI Page Routing, showPage, and Views](gui-pages-and-views.md)
- [Action Bar and Right Navigation](gui-action-bar.md)
- [Bootstrap Table Grid Patterns](bootstrap-table-grids.md)
- [AJAX Routing](ajax-routing.md)
- [Asset Pipeline — JS, CSS, LESS](asset-pipeline.md)

## Hooks and Dialplan

- [Hooks System](hooks-system.md)
- [Hook Response Processing Patterns](hook-response-patterns.md)
- [General Module Hooks](general-module-hooks.md) 
- [Dialplan Injection](dialplan-injection.md)
- [Asterisk Config File Generation](asterisk-config-generation.md)
- [Feature Codes](feature-codes.md)
- [Destinations Framework Integration](destinations-framework.md)