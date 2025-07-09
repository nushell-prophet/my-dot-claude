# Nushell Development Instructions for AI Agents

## CRITICAL: This is NOT Python!

## Understanding NUON (Nushell Object Notation)

NUON is Nushell's data notation format - a superset of JSON:
- **Lists**: `[1 2 3]` - spaces OR commas separate items
- **Records**: `{name: "value", age: 30}` - colons after keys, commas optional
- **Comments**: Allowed in NUON
- **Bare strings**: `name` instead of `"name"` for simple identifiers

## Core Language Patterns

### Variable Declaration
```nu
let x = 5                # Immutable by default
mut x = 5                # Explicitly mutable
let my_list = [1 2 3]    # Lists use spaces, not commas
let result = (func $x)   # Commands may need parentheses
```

### Function Definition
```nu
def greet [name: string] {       # Parameters in square brackets, no $
    $"Hello, ($name)!"           # Implicit return - last expression
}

# Parameter types:
def cmd [
    required: string             # Required parameter
    optional?: int               # Optional parameter with ?
    --flag (-f)                  # Flag
    --option: string             # Option with value
    ...rest                      # Rest parameters
] { }
```

### String Patterns
```nu
$"Hello ($name)"                 # String interpolation
$'Hello ($name)'                 # Single quotes: no escape sequences
"Hello \n World"                 # Double quotes: C-style escapes
'Hello \n World'                 # Single quotes: literal \n
r#'Hello "World"'#               # Raw strings
`/path/with spaces/`             # Backticks for paths
```

### Control Flow
```nu
# If is an expression
if $x > 0 { "positive" } else { "negative" }

# Prefer functional style
$items | each {|item| process $item }     # Fast, streaming
# NOT: for $item in $items { ... }        # Slow, returns null
```

### Data Access
```nu
$data.key                        # Cell path access
$data | get key                  # Pipeline access
$data.key? | default "value"     # Optional access with ?
$list.0                          # List index (0-based)
$list | last                     # Last element (no negative indexing)
```

### Lists and Iteration
```nu
$numbers | each {|x| $x * 2 }               # Transform lists
$list | append $item                        # Returns new list
mut list = []; $list ++= [$item]           # Mutable append
[...$list $item ...$more]                   # Spread operator
```

### Imports and Modules
```nu
use module.nu                    # Import module as command
use module.nu *                  # Import all exports
use module.nu [func1 func2]      # Import specific functions - LIST syntax!
use module.nu func               # Import single function
# NOT: use module.nu [func as f] # No aliasing in import lists!
```

### Error Handling
```nu
try {
    risky_operation
} catch {|e|
    print $"Error: ($e)"
}

# Create errors:
error make {msg: "Something went wrong"}
```

## Key Conceptual Differences

1. **Immutability First**: Variables are immutable by default
2. **Everything is Data**: Commands return structured data, not text
3. **Static Parsing**: No `eval()` - all parsing happens before execution
4. **Pipeline Centric**: Use `|` to transform data
5. **Scoped Changes**: Environment changes are scoped to blocks
6. **Type Safety**: Types are enforced at parse time when specified

## Common Python → Nushell Mistakes

| Python | Nushell |
|--------|---------|
| `x = 5` | `let x = 5` |
| `[1, 2, 3]` | `[1 2 3]` |
| `data['key']` | `$data.key` |
| `list[-1]` | `$list \| last` |
| `len(list)` | `$list \| length` |
| `print(x)` | Just evaluate `$x` |
| `True/False` | `true/false` |
| `None` | `null` |
| `elif` | `else if` |
| `for x in list:` | `$list \| each {\|x\| }` |
| `dict()` | `{}` |

## Working with External Commands

```nu
^ls                              # Use ^ prefix
^cat file.txt | lines            # Convert output to Nu data

# Capture all outputs
cat unknown.txt | complete
# Returns: {stdout: "", stderr: "error message", exit_code: 1}

# Redirections
command o> output.txt            # Redirect stdout
command e> error.txt             # Redirect stderr  
command o+e> combined.txt        # Redirect both
```

## Performance Patterns

```nu
# Use parallel processing
ls | where type == dir | par-each {|it| ls $it.name | length }

# Efficient imports
use std/assert equal             # Only what you need
# NOT: use std *                 # Loads everything

# Streaming vs collecting
open large.json | get items | where price > 100    # Good
# NOT: let data = open large.json; $data.items... # Collects all
```

## Type System

```nu
# Type annotations
let name: string = "value"
let items: list<string> = [foo bar]
let data: record<name: string, age: int> = {name: "Alice", age: 30}

# Runtime type checking
$value | describe                # Returns type as string

# Multi-type signatures
def process []: [string -> string, list<any> -> table] {
    # Accepts string OR list, returns different types
}
```

## Structured Data Navigation

```nu
# Cell paths
$data.users.0.name               # Navigate nested data
$data.users.name                 # Broadcasts over lists

# get vs select
$table | get name                # Returns VALUES (list)
$table | select name             # Returns STRUCTURE (table)

# Optional access
$record.field?                   # Returns null if missing
$config.timeout? | default 30    # With default
```

## Advanced Patterns

### Parse-Time vs Runtime
```nu
# ❌ WRONG - Runtime variable in parse-time keyword
let module = "utils"
use $module                      # ERROR

# ✅ RIGHT - Use constants
const MODULE = "utils"
use $MODULE                      # OK
```

### Hooks for Automation
```nu
$env.config.hooks = {
    pre_prompt: [{ 
        # Before each prompt
    }]
    pre_execution: [{ 
        # Before command execution
    }]
    env_change: {
        PWD: [{|before, after| 
            # When directory changes
        }]
    }
}
```

### Common Gotchas

1. **Environment Scoping**
   ```nu
   do { cd /tmp }; pwd           # Still in original dir
   def --env go-temp [] { cd /tmp }; go-temp  # Changes persist
   ```

2. **Single Return Value**
   ```nu
   def bad [] { "a"; "b" }       # Only returns "b"
   def good [] { ["a" "b"] }     # Returns both
   ```

3. **Mutable Variables in Closures**
   ```nu
   mut x = 0
   [1 2 3] | each { $x += 1 }   # ERROR
   # Use: [1 2 3] | length
   ```

## Script Best Practices

```nu
#!/usr/bin/env nu

def main [
    file?: path                  # Optional positional
    --stdin (-s)                 # Flag for pipe support
    --format: string = "json"    # Option with default
] {
    let data = if $stdin {
        $in
    } else if $file != null {
        open $file
    } else {
        error make {msg: "Provide file or use --stdin"}
    }
    # Process data...
}
```

## Quick Reference

- `$nu`: System info and paths
- `$env`: Environment variables (mutable)
- `$in`: Pipeline input
- `$it`: Only in `where` conditions
- `describe`: Check types
- `inspect`: Debug pipelines
- `par-each`: Parallel processing
- `?`: Optional access operator
- `...`: Spread operator

Remember: Nushell is a **functional shell**, not an imperative scripting language. Think pipelines, not procedures!

## Windows Compatibility

### CRITICAL: Cross-Platform Path Handling
- **NEVER** hardcode paths with forward slashes in module imports
- **Wrong**: `use numd/utils/file_ops.nu`
- **Correct**: 
  ```nu
  const file_ops_path = ([numd utils file_ops.nu] | path join)
  use $file_ops_path
  ```
- **Always** use `path join` for cross-platform compatibility
- This ensures code works on both Unix-like systems and Windows

### Example Patterns
```nu
# Wrong - breaks on Windows
use module/submodule/file.nu

# Correct - works everywhere
const module_path = ([module submodule file.nu] | path join)
use $module_path

# For dynamic paths
let config_path = [$env.HOME .config myapp config.toml] | path join
```

## Command Naming Conventions

When creating Nushell command modules:

1. **Command names use hyphens**: `clear-outputs`, `list-code-options`
2. **Command files use underscore suffix**: `run_cmd.nu`, `clear_outputs_cmd.nu` (avoids naming conflicts)
3. **Export named functions, not main**: 
   ```nu
   # In run_cmd.nu
   export def run [...] { }
   
   # In clear_outputs_cmd.nu
   export def clear-outputs [...] { }
   ```
4. **Subcommands use quoted strings**:
   ```nu
   export def "capture start" [...] { }
   export def "capture stop" [...] { }
   ```
5. **Import syntax**:
   ```nu
   export use commands/run_cmd.nu run
   export use commands/capture_cmd.nu ["capture start", "capture stop"]
   ```

### Why This Pattern?
- Nushell doesn't allow exported commands to have the same name as their module file
- Using `_cmd.nu` suffix prevents naming conflicts
- Allows commands to keep their natural hyphenated names
- Help output shows proper command names instead of script filenames
