# Debugging Neotest

This guide provides techniques and tools for debugging issues in the Neotest framework and adapters.

## Debugging with Print Statements

The simplest way to debug Lua code is using print statements:

```lua
vim.print("Debug:", variable)
```

However, these messages can get lost in the Neovim message history. For more structured debugging, use Neotest's logging system.

## Using Neotest's Logging System

Neotest provides a built-in logging system in `lua/neotest/logging.lua`:

```lua
local logger = require("neotest.logging")

-- Different log levels
logger.debug("Debug message")  -- Only shown when debug mode is enabled
logger.info("Info message")    -- General information
logger.warn("Warning message") -- Potential issues
logger.error("Error message")  -- Errors that shouldn't happen
```

### Enabling Debug Logging

To see debug logs, set the log level to debug in your Neotest configuration:

```lua
require("neotest").setup({
  log_level = "debug",  -- trace, debug, info, warn, error
})
```

### Log Files

Neotest logs are written to:
- Unix: `~/.local/state/nvim/neotest.log`

You can monitor logs in real-time using:

```bash
tail -f ~/.local/state/nvim/neotest.log
```

## Debugging Adapters

### Common Issues in Adapters

1. **Test Discovery Problems**
   - Check your Treesitter queries or regex patterns
   - Verify file paths and directory structures
   - Log the discovery process with detailed messages

2. **Command Building Issues**
   - Log the constructed command
   - Verify environment variables and paths
   - Test the command manually in a terminal

3. **Result Parsing Problems**
   - Log the raw output from the test runner
   - Check for format changes in the test runner output
   - Verify the parsing logic step by step

### Using the Debug Strategy

Neotest provides a debug strategy for running tests through a DAP adapter:

```lua
require("neotest").run.run({ strategy = "dap" })
```

To debug your own adapter with DAP, you need to implement the debug configuration
in your adapter's `build_spec` method.

## Tracing Function Calls

To understand the flow of execution, you can add tracing to functions:

```lua
-- At the beginning of a function
logger.debug("Entering function_name with args:", vim.inspect(args))

-- Before returning
logger.debug("Exiting function_name with result:", vim.inspect(result))
```

## Analyzing Test Trees

The test position tree is a core data structure in Neotest. To debug issues with it:

```lua
-- Log the entire tree structure
logger.debug("Tree structure:", vim.inspect(tree:to_list()))

-- Log a specific position
logger.debug("Position data:", vim.inspect(tree:data()))

-- Log children
logger.debug("Children:", vim.inspect(tree:children()))
```

## Debugging Async Code

Async code can be particularly challenging to debug. Some techniques:

1. Add log statements before and after async operations
2. Use `async.util.scheduler()` to run code on the main Neovim thread
3. Break complex async operations into smaller functions with logging

Example:
```lua
local async = require("neotest.async")

local function my_async_function()
  logger.debug("Starting async operation")
  
  local result = async.fn.system({"command"})
  logger.debug("Command result:", result)
  
  async.util.scheduler()  -- Switch back to main thread
  logger.debug("Back on main thread")
  
  return result
end
```

## Inspecting State

To inspect the current state of tests and results:

```lua
-- Get all test results
local results = require("neotest").state.results()
logger.debug("All results:", vim.inspect(results))

-- Get specific test result
local test_result = require("neotest").state.result(position_id)
logger.debug("Test result:", vim.inspect(test_result))
```

## Debugging UI Elements

For issues with UI components like the summary window:

1. Check if the UI is being updated correctly
2. Verify that events are being emitted and handled
3. Inspect window and buffer state

```lua
-- Log buffer content
local lines = vim.api.nvim_buf_get_lines(bufnr, 0, -1, false)
logger.debug("Buffer content:", vim.inspect(lines))

-- Log window dimensions
local win_config = vim.api.nvim_win_get_config(winnr)
logger.debug("Window config:", vim.inspect(win_config))
```

