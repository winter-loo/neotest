# Developing Adapters for Neotest

This guide explains how to develop custom adapters for the Neotest framework.

## Adapter Interface

All Neotest adapters must implement the `neotest.Adapter` interface defined in `lua/neotest/adapters/interface.lua`. The interface requires the following methods:

### Required Methods

1. **root(dir)** - Find the project root directory given a current directory
   ```lua
   ---@async
   ---@param dir string Directory to treat as cwd
   ---@return string|nil Absolute root dir of test suite
   ```

2. **is_test_file(file_path)** - Determine if a file is a test file
   ```lua
   ---@async
   ---@param file_path string
   ---@return boolean
   ```

3. **discover_positions(file_path)** - Parse all tests within a file
   ```lua
   ---@async
   ---@param file_path string Absolute file path
   ---@return neotest.Tree|nil
   ```

4. **build_spec(args)** - Build command specifications to run tests
   ```lua
   ---@param args neotest.RunArgs
   ---@return nil|neotest.RunSpec|neotest.RunSpec[]
   ```

5. **results(spec, result, tree)** - Parse test results
   ```lua
   ---@async
   ---@param spec neotest.RunSpec
   ---@param result neotest.StrategyResult
   ---@param tree neotest.Tree
   ---@return table<string, neotest.Result>
   ```

### Optional Methods

1. **filter_dir(name, rel_path, root)** - Filter directories when searching for test files
   ```lua
   ---@async
   ---@param name string Name of directory
   ---@param rel_path string Path to directory, relative to root
   ---@param root string Root directory of project
   ---@return boolean
   ```

## Creating an Adapter

### Step 1: Define the Adapter Module

Start by creating a new Lua module for your adapter:

```lua
local async = require("neotest.async")
local lib = require("neotest.lib")

---@type neotest.Adapter
local MyAdapter = { name = "my_adapter" }

-- Implement required methods
function MyAdapter.root(dir)
  -- Find project root
  -- Example:
  return lib.files.match_root_pattern("some_config_file.json")(dir)
end

-- ... Implement other methods

return MyAdapter
```

### Step 2: Identifying Test Files

Implement the `is_test_file` method to identify which files contain tests:

```lua
function MyAdapter.is_test_file(file_path)
  -- Example: Check if file matches a pattern
  return file_path:match("_test%.") or file_path:match("test_") or file_path:match("%.test%.")
end
```

### Step 3: Parsing Test Structures

For languages supported by Treesitter, use the neotest treesitter library to parse tests:

```lua
function MyAdapter.discover_positions(file_path)
  -- Define Treesitter query to find tests
  local query = [[
    ;; Define your query to match test structures
    ((function_call
        name: (identifier) @func_name (#match? @func_name "^test_")
        arguments: (arguments (_) @test.name)
    )) @test.definition
  ]]
  
  -- Parse the file
  return lib.treesitter.parse_positions(file_path, query, {})
end
```

For languages without Treesitter support, you can use regular expressions or other parsing methods.

### Step 4: Building Run Specifications

The `build_spec` method creates commands to run tests:

```lua
function MyAdapter.build_spec(args)
  local position = args.tree:data()
  local command = {"test_runner"}
  
  if position.type == "test" then
    table.insert(command, "--test")
    table.insert(command, position.name)
  elseif position.type == "namespace" then
    table.insert(command, "--namespace")
    table.insert(command, position.name)
  end
  
  return {
    command = command,
    context = {
      file = position.path,
      position_id = position.id,
    }
  }
end
```

### Step 5: Parsing Results

Parse the results returned from the test runner:

```lua
function MyAdapter.results(spec, result, tree)
  -- Parse output from the test runner
  local results = {}
  
  -- Example: Parse JSON output
  local parsed_results = vim.json.decode(result.output)
  
  for _, test_result in ipairs(parsed_results) do
    local position_id = -- find matching position ID
    results[position_id] = {
      status = test_result.passed and "passed" or "failed",
      output = test_result.output,
      short = test_result.message,
    }
    
    if test_result.error_line then
      results[position_id].errors = {
        {
          message = test_result.message,
          line = test_result.error_line
        }
      }
    end
  end
  
  return results
end
```

## Best Practices

1. **Use async operations** when appropriate to avoid blocking the UI
2. **Handle errors gracefully** in each method
3. **Maintain compatibility** with other adapters
4. **Use common patterns** from existing adapters when possible
5. **Document your adapter** thoroughly

## Testing Your Adapter

1. Create sample test files for your language/framework
2. Set up your adapter in a neotest config
3. Test each functionality:
   - Test discovery
   - Running tests
   - Viewing results
   - Debugging tests (if supported)

## Examples

For concrete examples, look at existing adapters like:
- [neotest-python](https://github.com/nvim-neotest/neotest-python)
- [neotest-plenary](https://github.com/nvim-neotest/neotest-plenary)
- [neotest-vim-test](https://github.com/nvim-neotest/neotest-vim-test)

## Publishing Your Adapter

1. Create a GitHub repository with the naming convention `neotest-[language/framework]`
2. Include a comprehensive README with installation and configuration instructions
3. Add your adapter to the list in the main Neotest README.md
