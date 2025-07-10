learn lua syntax and basics in https://learnxinyminutes.com/lua/

search in language reference https://www.lua.org/manual/5.4/

---

Here are some common code constructs found in [neotest](https://github.com/nvim-neotest/neotest) repo:

## 1. Metatable Patterns

### Object-Oriented Programming with Metatables
```lua
-- Class definition with __index metamethod
local TestRunner = {}
function TestRunner:new()
  local obj = {}
  setmetatable(obj, self)
  self.__index = self
  return obj
end

-- Method definition using colon syntax
function TestRunner:run_tree(tree, args)
  -- self is automatically passed
end
```

### Proxy Objects with __newindex
```lua
-- Dynamic property assignment interception
setmetatable(consumer_listeners, {
  __newindex = function(_, key, value)
    if not client.listeners[key] then
      error("Invalid event name: " .. key)
    end
    client.listeners[key][name] = value
  end,
})
```

### Callable Objects with __call
```lua
-- Making tables callable like functions
neotest.status = setmetatable(neotest.status, {
  __call = function(_, ...)
    return init(...)
  end,
})
```

## 2. Error Handling Patterns

### Protected Calls (xpcall/pcall)
```lua
-- Catching errors with custom error handler
local success, result = xpcall(function()
  return dangerous_operation()
end, function(err)
  error_message = debug.traceback(err, 1)
end)

if not success then
  handle_error(error_message)
end
```

### Assertion-based Validation
```lua
assert(type(val) == "number", "concurrent must be a boolean or a number")
```

## 3. Functional Programming Patterns

### Higher-Order Functions and Closures
```lua
-- Creating closures for deferred execution
local async_runners = {}
for _, spec in ipairs(specs) do
  table.insert(async_runners, function()
    self:_run_spec(spec, tree, args, adapter_id, adapter, results_callback)
  end)
end
```

### Callback Pattern
```lua
-- Functions accepting callback functions
function run_tree(tree, args, adapter_id, adapter, results_callback)
  -- Process results
  results_callback(results)
end
```

## 4. Table Manipulation Patterns

### Table Extension and Merging
```lua
-- Deep merging tables
user_config = vim.tbl_deep_extend("force", default_config, config)

-- Shallow merging with precedence
args = vim.tbl_extend("keep", args or {}, { strategy = default_strategy })
```

### Table as Set/Dictionary
```lua
-- Using tables as sets
local files = {}
for _, node in tree:iter_nodes() do
  files[node:data().path] = true
end

-- Converting back to array
local file_list = vim.tbl_keys(files)
```

## 5. Module System Patterns

### Module Definition and Export
```lua
-- Standard module pattern
local M = {}

function M.some_function()
  -- implementation
end

return M
```

### Factory Functions
```lua
-- Returning constructor functions
return function(processes)
  return TestRunner:new(processes)
end
```

## 6. Async/Concurrency Patterns

### NIO (Neovim IO) Async Operations
```lua
local nio = require("nio")

-- Creating async functions
neotest.run.stop = nio.create(neotest.run.stop, 1)

-- Concurrent execution
nio.gather(async_runners)

-- Async context
nio.run(function()
  -- async operations
end)
```

### Semaphore for Resource Management
```lua
local semaphore = nio.control.semaphore(concurrent_limit)
semaphore.with(function()
  -- Protected resource access
end)
```

## 7. Event System Patterns

### Observer Pattern with Event Listeners
```lua
-- Event registration
client.listeners.results = function(adapter_id, results)
  handle_results(results)
end

-- Event emission
for name, listener in pairs(self.listeners[event] or {}) do
  listener(unpack(args))
end
```

## 8. Configuration Patterns

### Default Configuration with Override
```lua
local default_config = {
  log_level = vim.log.levels.WARN,
  -- other defaults
}

-- User config overrides defaults
local config = vim.tbl_deep_extend("force", default_config, user_config)
```

### Lazy Initialization
```lua
-- Compute expensive values only when needed
local function convert_concurrent(val)
  if val == 0 or val == true then
    local cpu_info = vim.loop.cpu_info() or {}
    return #cpu_info + 4
  end
  return val
end
```

## 9. Tree/Data Structure Patterns

### Tree Traversal
```lua
-- Iterating over tree nodes
for _, node in tree:iter_nodes() do
  local pos = node:data()
  process_position(pos)
end

-- Parent traversal
for parent in node:iter_parents() do
  process_parent(parent)
end
```

## 10. Validation and Type Checking

### Nil-safe Operations
```lua
-- Safe table access with fallback
for name, listener in pairs(self.listeners[event] or {}) do
  -- process listener
end

-- Nil coalescing
local cpu_info = vim.loop.cpu_info() or {}
```

## 11. String and Path Manipulation

### String Formatting
```lua
-- String interpolation
local message = ("invalid strategy value: %s"):format(spec.strategy)

-- Path manipulation
local path = vim.fn.fnamemodify(project_root, ":p")
path = path:sub(1, #path - 1) -- Remove trailing slash
```

## 12. Debugging and Logging Patterns

### Structured Logging
```lua
logger.info("Emitting", event, "event")
logger.debug("Calling listener", name, "for event", event)
logger.warn("Position already running:", tree:data().id)
```

### Debug Information
```lua
-- Stack traces for errors
errmsg = debug.traceback(err, 1)
```


