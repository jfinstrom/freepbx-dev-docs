---
title:  General Module Hooks — handling returned hook data (BMO/Hooks())
author: James Finstrom
date: 2025-08-21
---


# General Module Hooks — handling returned hook data (BMO/Hooks())

This short companion explains how to handle the data the framework's hook processor returns to a provider. The hook processor (e.g., processHooks()) returns an associative array keyed by the calling module name (usually the registering module's class/name) where each value is whatever the hooking module returned (array, string, scalar, null). This document shows safe, practical patterns to validate, merge, and act on those results.

Example return shape
- processHooks() => associative array:
  - keys: module names (e.g., "ModuleHandlerA")
  - values: whatever the handler returned (array|string|scalar|null)

Example:
```php
// Example of what processHooks() might return:
$responses = [
    "ModuleHandlerA" => ["status" => "ok", "notes" => ["one"]],
    "ModuleHandlerB" => ["status" => "error", "error" => "invalid config"],
    "ModuleHandlerC" => null,
    "ModuleHandlerD" => "simple-string"
];
```

General guidance
- Treat results as untrusted input. Always validate types and required keys before using results.
- Prefer structured, serializable return values (associative arrays with known keys).
- Decide and document merge semantics for multiple handlers (e.g., last-writer-wins, first-writer-wins, accumulator).
- Keep handler processing deterministic and fast. If a handler needs heavy work, require it to perform background processing.
- Log unexpected data and handler errors including module names for easier debugging.

Common processing patterns (pseudo-PHP)

1) Simple aggregation (collect successes and errors)
```php
function processHookResponses_Aggregate(array $responses) {
    $agg = ['successes' => [], 'errors' => [], 'raw' => $responses];

    foreach ($responses as $module => $result) {
        if ($result === null) {
            $agg['errors'][$module] = 'no response';
            continue;
        }
        if (is_array($result) && isset($result['error'])) {
            $agg['errors'][$module] = $result['error'];
            continue;
        }
        if (!is_array($result) && !is_string($result) && !is_numeric($result)) {
            $agg['errors'][$module] = 'unexpected return type: ' . gettype($result);
            continue;
        }
        $agg['successes'][$module] = $result;
    }

    return $agg;
}
```

2) Merge associative results (last-writer-wins)
- Use this when handlers contribute fields that should override earlier values. Ensure handlers know the merge semantics.
```php
function processHookResponses_LastWriterWins(array $responses, array $base) {
    $final = $base;
    foreach ($responses as $module => $result) {
        if (!is_array($result)) continue;
        foreach ($result as $k => $v) {
            $final[$k] = $v; // later modules overwrite earlier ones
        }
    }
    return $final;
}
```

3) Merge associative results (first-writer-wins)
```php
function processHookResponses_FirstWriterWins(array $responses, array $base) {
    $final = $base;
    foreach ($responses as $module => $result) {
        if (!is_array($result)) continue;
        foreach ($result as $k => $v) {
            if (!array_key_exists($k, $final)) {
                $final[$k] = $v;
            }
        }
    }
    return $final;
}
```

4) Accumulator pattern (handlers append to lists)
- Use this when handlers return lists to be concatenated.
```php
function processHookResponses_Accumulate(array $responses) {
    $items = [];
    foreach ($responses as $module => $result) {
        if (!is_array($result)) continue;
        if (isset($result['items']) && is_array($result['items'])) {
            $items = array_merge($items, $result['items']);
        }
    }
    return $items;
}
```

5) Short-circuit / veto behavior
- Only use if the hook contract explicitly allows a handler to abort the operation. Document the behavior and enforce it clearly.
```php
function processHookResponses_CheckVeto(array $responses) {
    foreach ($responses as $module => $result) {
        if (is_array($result) && !empty($result['veto'])) {
            return [
                'status' => 'aborted',
                'by' => $module,
                'reason' => $result['veto'],
            ];
        }
    }
    return ['status' => 'ok'];
}
```

6) Per-module lookup (processHooksByModule equivalent)
- When you call the "by module" variant or otherwise want a single module's result:
```php
$single = \FreePBX::Hooks()->processHooksByModule('ModuleHandlerA', $payload);
// Validate the single result
if ($single === null) {
    // no response or not implemented
}
if (is_array($single) && isset($single['error'])) {
    // handle error
}
```

Provider-side usage example (putting it together)
```php
// In provider class
$payload = ['context' => 'doSomething', 'input' => $args];
$responses = \FreePBX::Hooks()->processHooks($payload);

// 1) Aggregate & log
$agg = processHookResponses_Aggregate($responses);
if (!empty($agg['errors'])) {
    foreach ($agg['errors'] as $mod => $err) {
        logger("Hook error from $mod: $err");
    }
}

// 2) Merge changes into final result using a documented strategy
$final = processHookResponses_LastWriterWins($responses, ['result' => 'base']);

// 3) Honor vetos (if contract allows)
$veto = processHookResponses_CheckVeto($responses);
if ($veto['status'] === 'aborted') {
    return $veto; // stop and return reason
}

// 4) Return combined response to caller
return [
    'result' => $final,
    'extensions' => $agg['successes'],
    'hook_summary' => [
        'errors' => $agg['errors'],
        'raw' => $responses,
    ],
];
```

Error handling and logging
- Wrap calls to the hook processor in try/catch if your environment may throw exceptions for missing modules or other conditions.
- Log the module name when reporting problems; that makes debugging much easier.
- If the hook processor includes stack traces or exceptions from handlers, decide whether to expose them to callers or only record them in logs.

Testing tips
- Create a simple test consumer that returns a predictable payload so you can confirm ordering and merge behavior.
- Change priorities in module declarations to validate ordering.
- Test null, string, numeric, and malformed returns to verify your validation logic.

Contract and documentation
- The most important single thing: document the hook contract (payload shape, allowed return types, merge semantics, error and veto behavior). Consumers rely on precise contracts to behave correctly.
- In module.xml and provider docs, include example payloads and an example expected return value.

Summary checklist for providers
- Call processHooks($payload) and expect an associative array keyed by module.
- Validate every entry before using it.
- Decide and apply a merge strategy; document it.
- Log errors with module names.
- If changes are made to module.xml, refresh the system cache so the registry is updated (retrieve_conf / fwconsole reload).

Appendix: sample returned data (for reference)
```php
[
  "ModuleHandlerA" => ["status" => "ok", "config" => ["feature" => true]],
  "ModuleHandlerB" => ["status" => "ok", "config" => ["feature" => false]],
  "ModuleHandlerC" => ["error" => "missing credentials"],
  "ModuleHandlerD" => null,
  "ModuleHandlerE" => ["veto" => "not allowed by policy"]
]
```
Use the patterns above to validate, log, merge, or abort based on this structure.
