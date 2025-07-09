# Nutest Testing Framework Instructions for AI Agents

## Overview
Nutest is a testing framework for Nushell that provides test discovery, execution, and reporting capabilities. Tests run in isolated subprocesses with support for parallel execution.

## Quick Start

```nushell
# Import nutest
use /path/to/nutest

# Run all tests in current directory
nutest run-tests

# List discovered tests
nutest list-tests
```

## Writing Tests

### Basic Test Structure
```nushell
# test_example.nu
use std assert

@test
def "should add numbers correctly" [] {
    assert equal (2 + 2) 4
}

@test
def "should handle strings" [] {
    assert equal ("hello" | str upcase) "HELLO"
}
```

### Test File Naming
Tests are discovered in files matching these patterns:
- `test_*.nu`
- `test-*.nu`
- `*_test.nu`
- `*-test.nu`

### Test Annotations

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@test` | Mark a function as a test | `@test`<br>`def "my test" [] { ... }` |
| `@before-all` | Run once before all tests in suite | Setup shared resources |
| `@before-each` | Run before each test | Reset state |
| `@after-all` | Run once after all tests | Cleanup resources |
| `@after-each` | Run after each test | Reset state |
| `@ignore` | Skip this test | Temporarily disable |
| `@strategy` | Control suite concurrency | `@strategy: serial` or `@strategy: parallel` |

### Context Passing
Setup functions can pass context to tests via pipeline:

```nushell
@before-each
def setup [] {
    {
        db: (open test.db)
        temp_dir: (mktemp -d)
    }
}

@test
def "test with context" [] {
    let ctx = $in  # Context from setup
    # Use $ctx.db and $ctx.temp_dir
    assert equal ($ctx.db | query db "SELECT 1") [[1]; [1]]
}

@after-each
def cleanup [] {
    let ctx = $in
    rm -rf $ctx.temp_dir
}
```

## Running Tests

### Basic Commands
```nushell
# Run all tests
nutest run-tests

# Run tests in specific directory
nutest run-tests --path ./tests

# Run tests matching pattern
nutest run-tests --match-tests "api.*"
nutest run-tests --match-suites "integration.*"

# Run with specific number of threads
nutest run-tests --threads 4

# Fail with non-zero exit code (for CI)
nutest run-tests --fail
```

### Display Options
```nushell
# Terminal display (default, real-time)
nutest run-tests --display terminal

# Table display (summary at end)
nutest run-tests --display table

# Quiet mode (no output)
nutest run-tests --display none
```

### Return Options
```nushell
# Return nothing (default)
nutest run-tests

# Return summary stats
nutest run-tests --returns summary

# Return suite details
nutest run-tests --returns suites

# Return all test details
nutest run-tests --returns tests
```

### Reporting
```nushell
# Generate JUnit XML report
nutest run-tests --report {type: junit, path: "test-results.xml"}

# Generate badge
nutest run-tests --report {type: badge, path: "test-badge.svg"}
```

## Common Testing Patterns

### Testing Commands
```nushell
@test
def "custom command should format data" [] {
    let result = my-format-command {name: "test", value: 42}
    assert equal $result "test: 42"
}
```

### Testing External Commands
```nushell
@test
def "external command should succeed" [] {
    let result = ^echo "hello" | complete
    assert equal $result.exit_code 0
    assert equal ($result.stdout | str trim) "hello"
}
```

### Testing Error Conditions
```nushell
@test
def "should handle invalid input" [] {
    let result = try { 
        my-command "invalid" 
    } catch { |e| 
        $e.msg 
    }
    assert str contains $result "Invalid input"
}
```

### Testing File Operations
```nushell
@before-each
def setup [] {
    let temp_dir = mktemp -d
    { temp_dir: $temp_dir }
}

@test
def "should create config file" [] {
    let ctx = $in
    let config_path = $ctx.temp_dir | path join "config.nu"
    
    create-config $config_path
    
    assert equal (path exists $config_path) true
    let content = open $config_path
    assert str contains $content "version = 1"
}

@after-each
def cleanup [] {
    let ctx = $in
    rm -rf $ctx.temp_dir
}
```

## CI/CD Integration

```yaml
# GitHub Actions example
- name: Run tests
  run: |
    nu -c "use nutest; nutest run-tests --fail --report {type: junit, path: results.xml}"

# GitLab CI example
test:
  script:
    - nu -c "use nutest; nutest run-tests --fail"
  artifacts:
    reports:
      junit: results.xml
```

## Common Pitfalls and Solutions

### 1. Test Discovery Issues
```nushell
# ❌ Wrong: Test not discovered
def "my test" [] { ... }  # Missing @test annotation

# ✅ Correct: 
@test
def "my test" [] { ... }
```

### 2. Context Usage
```nushell
# ❌ Wrong: Forgetting context is piped
@test
def "test" [] {
    let ctx = setup  # Won't work
}

# ✅ Correct: Context comes from pipeline
@test
def "test" [] {
    let ctx = $in  # From @before-each
}
```

### 3. Async/Parallel Issues
```nushell
# For tests that must run serially
@strategy: serial
module test_database {
    @test
    def "test 1" [] { ... }
    
    @test
    def "test 2" [] { ... }
}
```

### 4. Output Capture
```nushell
# Test output is captured, use explicit returns
@test
def "test with output" [] {
    print "This is captured"  # Won't show during test
    assert equal 1 1
}
```

## Best Practices

1. **Organize tests by feature**: Group related tests in modules
2. **Use descriptive test names**: "should calculate tax for international orders"
3. **One assertion per test**: Makes failures clear
4. **Clean up resources**: Always use @after-each for cleanup
5. **Test edge cases**: Empty inputs, large numbers, special characters
6. **Use context for shared state**: Don't rely on global variables
7. **Keep tests independent**: Each test should be runnable alone

## Example Test Suite

```nushell
# test_user_service.nu
use std assert

@strategy: parallel
module test_user_service {
    @before-all
    def setup_database [] {
        print "Creating test database..."
        # Setup code here
        { db_path: "/tmp/test.db" }
    }
    
    @before-each
    def setup_user [] {
        let suite_ctx = $in
        {
            ...$suite_ctx
            user: {id: 1, name: "Test User"}
        }
    }
    
    @test
    def "should create new user" [] {
        let ctx = $in
        let user = create_user "John Doe" "john@example.com"
        
        assert equal $user.name "John Doe"
        assert equal $user.email "john@example.com"
        assert equal ($user.id | describe) "int"
    }
    
    @test
    def "should validate email" [] {
        let result = try {
            create_user "Jane" "invalid-email"
        } catch { |e| $e.msg }
        
        assert str contains $result "Invalid email"
    }
    
    @test
    @ignore  # TODO: Fix this test
    def "should handle concurrent creates" [] {
        # Test implementation pending
    }
    
    @after-all
    def cleanup_database [] {
        let ctx = $in
        rm -f $ctx.db_path
        print "Cleaned up test database"
    }
}
```

## Tips for AI Agents

1. **Always use @test annotation** - Without it, tests won't be discovered
2. **Check test file naming** - Must match `*test*.nu` patterns
3. **Run with --fail for CI** - Ensures non-zero exit on failure
4. **Use structured returns** - `--returns tests` for processing results
5. **Capture external command output** - Use `complete` for exit codes
6. **Isolate test data** - Use temp directories and cleanup
7. **Test both success and failure** - Cover error paths explicitly