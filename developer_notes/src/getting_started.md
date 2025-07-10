# Getting Started with Neotest Development

This guide will help developers understand the neotest codebase and how to start contributing to it.

## Project Overview

Neotest is a framework for interacting with tests within NeoVim. It provides an extensible architecture that allows developers to create adapters for various testing frameworks.

The project has three main components:

1. **Adapters**: Language-specific modules that parse tests, build commands, and parse results
2. **Client**: Core engine that runs tests and stores the state of tests and results, emitting events during operations
3. **Consumers**: Modules that use the client to provide utilities for interacting with tests and results

## Setting Up Development Environment

### Prerequisites

- Neovim (nightly build recommended for full feature support)
- [plenary.nvim](https://github.com/nvim-lua/plenary.nvim) for utilities
- [nvim-nio](https://github.com/nvim-neotest/nvim-nio) for async functionality
- [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) for parsing

### Local Development Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/nvim-neotest/neotest.git
   ```

2. Symlink the repository to your Neovim plugins directory or use your plugin manager in development mode.

   For example, with packer.nvim:
   ```lua
   use {
     "path/to/your/local/neotest",
     requires = {
       "nvim-neotest/nvim-nio",
       "nvim-lua/plenary.nvim",
       "antoinemadec/FixCursorHold.nvim",
       "nvim-treesitter/nvim-treesitter"
     }
   }
   ```

3. Configure neotest for development:
   ```lua
   require("neotest").setup({
     -- Your development configuration
     adapters = {
       -- Add test adapters you're working with
     },
   })
   ```

## Project Structure

```
neotest/
├── lua/
│   └── neotest/
│       ├── adapters/       # Adapter interfaces
│       ├── client/         # Core client functionality
│       ├── config/         # Configuration handling
│       ├── consumers/      # User-facing features
│       ├── lib/            # Shared utilities
│       ├── types/          # Type definitions
│       └── utils/          # Helper functions
├── tests/                  # Test suite
└── doc/                    # Documentation
```

## Understanding the Core Components

### Adapters

Adapters are the bridge between neotest and testing frameworks. They handle:
- Identifying test files
- Parsing test structures
- Building commands to run tests
- Parsing test results

See [adapters.md](adapters.md) for details on developing adapters.

### Client

The client is the central piece that:
- Manages the tree of tests
- Runs tests via adapters
- Stores and provides access to results
- Emits events for consumers

### Consumers

Consumers provide user-facing functionality:
- Summary window
- Output display
- Diagnostic integration
- Run commands
- Status signs
