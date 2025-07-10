# Testing Neotest Code

This guide explains how to test your changes to the Neotest framework and adapters.

## Testing Framework

Neotest uses Plenary's testing framework for its test suite. Tests are located in the `tests/` directory and organized by component type.

## Running Tests

### Running the Full Test Suite

To run all tests, use the following command from the root of the Neotest repository:

```bash
nvim --headless -c "PlenaryBustedDirectory tests/unit/ {minimal_init = 'tests/minimal_init.lua'}"
```

### Running Specific Tests

To run specific tests, you can specify the test file pattern:

```bash
nvim --headless -c "PlenaryBustedDirectory tests/unit/adapters {minimal_init = 'tests/minimal_init.lua'}"
```

## Test Structure

Tests in Neotest follow the Plenary test structure with `describe` blocks for grouping related tests and `it` blocks for individual test cases:

```lua
local async = require("plenary.async")

describe("Component name", function()
  before_each(function()
    -- Setup code
  end)

  after_each(function()
    -- Teardown code
  end)

  it("should do something", function()
    -- Test code
    assert.equals(expected, actual)
  end)

  it("should handle async operations", async.it(function()
    -- Async test code using async.run and async.wait
  end))
end)
```

## Testing Adapters

### Unit Tests for Adapters

When testing adapters, focus on testing each component separately:

1. **Test File Detection**:
   ```lua
   describe("is_test_file", function()
     it("identifies test files correctly", function()
       local adapter = require("my_adapter")
       assert.is_true(adapter.is_test_file("test_file.py"))
       assert.is_false(adapter.is_test_file("regular_file.py"))
     end)
   end)
   ```

2. **Test Discovery**:
   ```lua
   describe("discover_positions", function()
     it("discovers tests in a file", function()
       local adapter = require("my_adapter")
       local tree = adapter.discover_positions("path/to/test_file.py")
       assert.is_not_nil(tree)
       -- Verify the tree structure
     end)
   end)
   ```

3. **Command Building**:
   ```lua
   describe("build_spec", function()
     it("builds correct command for test", function()
       local adapter = require("my_adapter")
       local spec = adapter.build_spec({
         -- Mock arguments
       })
       assert.equals("expected_command", spec.command[1])
     end)
   end)
   ```

4. **Results Parsing**:
   ```lua
   describe("results", function()
     it("parses results correctly", function()
       local adapter = require("my_adapter")
       local results = adapter.results(
         -- Mock spec
         -- Mock result
         -- Mock tree
       )
       assert.equals("passed", results["test_id"].status)
     end)
   end)
   ```

### Integration Testing

For integration testing, create example test files that your adapter should handle and verify that:

1. The test files are correctly identified
2. Tests within the files are correctly discovered
3. Commands are correctly built for running tests
4. Results are correctly parsed when tests are run

## Mocking

Use Plenary's mocking capabilities for unit tests:

```lua
local mock = require("luassert.mock")

describe("Component with dependencies", function()
  local dependency_mock
  
  before_each(function()
    dependency_mock = mock(require("dependency"), true)
  end)

  after_each(function()
    mock.revert(dependency_mock)
  end)

  it("should use dependency", function()
    -- Setup mock behavior
    dependency_mock.function_name.returns("expected_value")
    
    -- Test code that uses the dependency
    
    -- Verify mock was called
    assert.stub(dependency_mock.function_name).was_called()
  end)
end)
```

## Testing Async Code

For testing asynchronous code, use Plenary's async utilities:

```lua
local async = require("plenary.async")

it("should handle async operations", async.it(function()
  local result = async.wait(async.run(function()
    -- Async operation
    return "result"
  end))
  
  assert.equals("result", result)
end))
```

## Continuous Integration

Neotest uses GitHub Actions for continuous integration. When submitting a pull request, ensure all tests pass in the CI pipeline.

The CI configuration is defined in the `.github/workflows` directory.

## Best Practices

1. **Write tests for new features**: Any new feature should come with tests
2. **Test edge cases**: Consider unusual inputs and error conditions
3. **Keep tests isolated**: Each test should run independently
4. **Use descriptive test names**: Name tests based on what they verify
5. **Clean up after tests**: Ensure tests don't leave side effects

## Troubleshooting Tests

If your tests are failing:

1. Run the specific failing test to see detailed error output
2. Use `print()` statements for debugging (they will appear in the test output)
3. Check that mocks and fixtures are set up correctly
4. Verify that async operations are properly awaited
5. Look for side effects from other tests that might affect your test
