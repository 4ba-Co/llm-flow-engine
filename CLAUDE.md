# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LLM Flow Engine is a DSL-based workflow engine for orchestrating multiple Large Language Model (LLM) interactions in Python. It enables complex AI workflows through YAML configuration files and supports collaborative work between multiple LLM models using directed acyclic graph (DAG) execution.

## Key Technologies

- **Python 3.8+** with asyncio/aiohttp for async execution
- **PyYAML** for DSL parsing
- **loguru** for structured logging
- Core dependencies: `aiohttp>=3.8.0`, `pyyaml>=6.0`, `loguru>=0.7.0`

## Development Commands

### Setup and Dependencies
```bash
# Install development environment
make setup-dev
# Or manually:
pip install -e ".[dev]"

# Install just project dependencies
make install
```

### Testing and Validation
```bash
# Run project validation tests
make test
# Or directly:
python validate_project.py

# For Windows users
python validate_project_win.py
```

### Code Quality
```bash
# Format code
black .

# Type checking
mypy llm_flow_engine/

# Linting
flake8 llm_flow_engine/
```

### Build and Publishing
```bash
# Clean build artifacts
make clean

# Build package
make build

# Check package integrity
make check

# Full pre-publish validation
make pre-publish

# Publish to TestPyPI
make test-publish

# Publish to PyPI
make publish
```

### Running Examples
```bash
# Basic usage demo
python examples/demo_example.py

# Model configuration demo
python examples/model_config_demo.py

# Package usage demo
python examples/package_demo.py
```

## Architecture Overview

### Core Components

- **`FlowEngine`** (`flow_engine.py`) - Main orchestration engine and entry point
- **`WorkFlow`** (`workflow.py`) - Unified workflow management supporting both simple and DAG execution
- **`Executor`** (`executor.py`) - Individual task execution with timeout/retry logic
- **`ModelConfigProvider`** (`model_config.py`) - Multi-platform LLM configuration management
- **`builtin_functions.py`** - Built-in function library (LLM calls, data transforms, HTTP requests)
- **`dsl_loader.py`** - YAML DSL parser and workflow loading

### DSL Workflow Structure

Workflows are defined in YAML with this structure:
```yaml
metadata:
  version: "1.0"
  description: "Description"

input:
  type: "start"
  name: "workflow_input"
  data:
    key: "value"

executors:
  - name: task_name
    type: task
    func: function_name
    custom_vars:
      param: "${input.key}"  # Placeholder resolution
    depends_on: ["other_task"]  # DAG dependencies
    timeout: 30
    retry: 2

output:
  type: "end"
  name: "workflow_output"
  data:
    result: "${task_name.output}"
```

### Key Patterns

1. **Placeholder Resolution**: Use `${node.output}` syntax for inter-node data passing
2. **DAG Execution**: Tasks can specify `depends_on` for dependency management
3. **Parallel Execution**: Independent tasks run concurrently
4. **Model Abstraction**: Support for Ollama, OpenAI, Anthropic, and custom APIs
5. **Async/Await**: All execution is async with proper error handling and retries

### Built-in Functions

- **`llm_simple_call`** - Basic LLM model calls with user_input and model parameters
- **`llm_api_call`** - Advanced API calls with custom prompts and parameters
- **`text_process`** - Text preprocessing and formatting
- **`data_merge`** - Combine multiple data sources
- **`combine_outputs`** - Merge results from multiple executors
- **`calculate`** - Mathematical operations

### Model Configuration

Support for multiple LLM platforms:
- **Ollama** - Local models via auto-discovery
- **OpenAI** - GPT series models
- **Anthropic** - Claude models
- **Custom APIs** - OpenAI-compatible endpoints

Auto-discovery example:
```python
provider = await ModelConfigProvider.from_host_async(
    api_host="http://127.0.0.1:11434",
    platform="ollama"
)
```

## Common Development Patterns

### Testing New Features
1. Add tests to `validate_project.py`
2. Run `make test` to validate
3. Test examples in `examples/` directory

### Adding Built-in Functions
1. Add function to `builtin_functions.py`
2. Register in `BUILTIN_FUNCTIONS` dict
3. Update validation tests
4. Test with example workflows

### Model Provider Integration
1. Extend `ModelConfigProvider` in `model_config.py`
2. Add platform-specific configuration handling
3. Test auto-discovery if applicable

### Cross-Platform Compatibility
- Windows encoding handled via UTF-8 configuration
- Platform-specific validation scripts
- GitHub Actions CI tests Ubuntu, Windows, macOS
- Use `safe_print()` function for Unicode output

## File Organization

- `/llm_flow_engine/` - Core engine source code
- `/examples/` - Usage examples and demo workflows
- `/scripts/` - Build, publish, and utility scripts
- `/docs/` - Additional documentation
- `validate_project.py` - Main validation script (Unix/Linux/macOS)
- `validate_project_win.py` - Windows-specific validation