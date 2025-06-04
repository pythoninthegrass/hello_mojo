# claude.md - Mojo Programming Language Guide

## Overview

Mojo is a high-performance programming language that bridges Python's ease of use with systems programming capabilities. It combines Python syntax and ecosystem with advanced features for AI, GPU programming, and high-performance computing.

**Key Features:**

- Python-compatible syntax and interoperability
- Systems programming capabilities with memory safety
- GPU programming support (NVIDIA)
- Compile-time metaprogramming
- Zero-cost abstractions
- MLIR-based compiler infrastructure

## Installation & Setup

### Using pip/uv

```bash
# With pip
pip install modular

# With uv (recommended for developers)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create a new project with pyproject.toml
uv init --bare --author-from git my-mojo-project
cd my-mojo-project

# Create virtual environment with system Python
uv venv  # or specify version: uv venv --python 3.11

# Activate the environment
source .venv/bin/activate

# Add Modular package
uv add modular
```

### Project Setup with Dependencies

```bash
# Add core dependencies
uv add numpy matplotlib pandas

# Add optional development dependencies
uv add --optional dev ruff ty

# Add from requirements.txt if you have one
uv add -r requirements.txt

# Install all dependencies
uv pip install -r pyproject.toml

# Install with optional extras
uv pip install -r pyproject.toml --all-extras
```

### Basic Project Structure

```text
my-mojo-project/
├── main.mojo          # Entry point
├── mymodule.mojo      # Custom module
├── pyproject.toml     # Project configuration
├── requirements.txt   # Generated dependencies
└── .venv/            # Virtual environment
```

## Language Basics

### Hello World

```mojo
def main():
    print("Hello, Mojo! 🔥")
```

### Variables and Types

```mojo
def main():
    # Implicit typing
    name = "Alice"
    age = 30
    
    # Explicit typing
    var score: Float64 = 95.5
    var count: Int = 0
    
    # Late initialization
    var value: String
    value = "initialized later"
```

### Functions

```mojo
# def function (Python-like, can raise errors)
def greet(name: String) -> String:
    return "Hello, " + name + "!"

# fn function (systems programming, explicit error handling)
fn add(a: Int, b: Int) -> Int:
    return a + b

# Function with error handling
fn divide(a: Float64, b: Float64) raises -> Float64:
    if b == 0:
        raise Error("Division by zero")
    return a / b
```

## Python Interoperability (Prioritized)

### Calling Python from Mojo

```mojo
from python import Python

def main():
    # Import Python modules
    np = Python.import_module("numpy")
    plt = Python.import_module("matplotlib.pyplot")
    
    # Use Python objects
    array = np.array([1, 2, 3, 4, 5])
    result = np.sum(array)
    print("Sum:", result)
    
    # Create Python collections
    py_list = Python.list(1, 2, 3)
    py_dict = Python.dict()
    py_dict["key"] = "value"
```

### Advanced Dependency Management

```bash
# Add Python packages with version constraints
uv add "numpy>=1.20,<2.0" "matplotlib>=3.5"

# Add from specific requirements file
uv add -r ml-requirements.txt

# Export current dependencies
uv pip freeze > requirements.txt

# Compile requirements from pyproject.toml
uv pip compile pyproject.toml -o requirements.txt

# Install from compiled requirements
uv pip install -r requirements.txt
```

### Working with Python Objects

```mojo
from python import Python, PythonObject

def process_python_data(py_obj: PythonObject):
    # Convert to Mojo types when needed
    if Python.type(py_obj) is Python.evaluate("int"):
        mojo_int = Int(py_obj)
        print("Converted to Mojo Int:", mojo_int)
    
    # Work directly with Python objects
    result = py_obj * 2
    return result
```

### Calling Mojo from Python (Preview)

```mojo
# mojo_module.mojo
from python import PythonObject
from python.bindings import PythonModuleBuilder
import math

@export
fn PyInit_mojo_module() -> PythonObject:
    try:
        var m = PythonModuleBuilder("mojo_module")
        m.def_function[factorial]("factorial", docstring="Compute n!")
        return m.finalize()
    except e:
        return abort[PythonObject](String("error creating module:", e))

fn factorial(py_obj: PythonObject) raises -> PythonObject:
    var n = Int(py_obj)
    return math.factorial(n)
```

```python
# main.py
import max._mojo.mojo_importer
import mojo_module

print(mojo_module.factorial(5))  # 120
```

## Core Language Features

### Structs

```mojo
@value  # Automatically generates lifecycle methods
struct Point:
    var x: Float64
    var y: Float64
    
    def distance_from_origin(self) -> Float64:
        return (self.x**2 + self.y**2)**0.5
    
    def __str__(self) -> String:
        return String("Point(", self.x, ", ", self.y, ")")

def main():
    point = Point(3.0, 4.0)
    print(point)  # Point(3.0, 4.0)
    print("Distance:", point.distance_from_origin())  # 5.0
```

### Traits

```mojo
trait Drawable:
    fn draw(self): ...

@value
struct Circle(Drawable):
    var radius: Float64
    
    fn draw(self):
        print("Drawing circle with radius", self.radius)

fn render[T: Drawable](shape: T):
    shape.draw()

def main():
    circle = Circle(5.0)
    render(circle)
```

### Parameters (Compile-time)

```mojo
fn print_n_times[count: Int](message: String):
    @parameter
    for i in range(count):
        print(message)

def main():
    print_n_times[3]("Hello!")  # Unrolled at compile time
```

### Collections

```mojo
from collections import List, Dict, Set

def main():
    # Lists
    numbers = [1, 2, 3, 4, 5]
    numbers.append(6)
    
    # Dictionaries
    scores = Dict[String, Int]()
    scores["Alice"] = 95
    scores["Bob"] = 87
    
    # Sets
    unique_values = {1, 2, 3, 2, 1}  # {1, 2, 3}
    
    # Iteration
    for num in numbers:
        print(num)
```

## GPU Programming

```mojo
from gpu import DeviceContext
from gpu.host import DeviceBuffer

def simple_gpu_kernel():
    print("Running on GPU thread")

def main():
    ctx = DeviceContext()
    
    # Launch kernel
    ctx.enqueue_function[simple_gpu_kernel](grid_dim=1, block_dim=4)
    ctx.synchronize()
    
    # Work with GPU memory
    buffer = ctx.enqueue_create_buffer[DType.float32](1024)
    # Process data...
```

## Performance Patterns

### SIMD Operations

```mojo
def simd_example():
    # Vectorized operations
    vec1 = SIMD[DType.float32, 4](1.0, 2.0, 3.0, 4.0)
    vec2 = SIMD[DType.float32, 4](5.0, 6.0, 7.0, 8.0)
    result = vec1 + vec2  # [6.0, 8.0, 10.0, 12.0]
    print(result)
```

### Memory Management

```mojo
from memory import UnsafePointer

fn efficient_processing[type: AnyType](data: UnsafePointer[type], size: Int):
    # Low-level memory operations for performance
    for i in range(size):
        # Process data[i]
        pass
```

## Error Handling

```mojo
def safe_division() -> String:
    try:
        result = divide(10.0, 0.0)
        return String("Result: ", result)
    except e:
        return String("Error: ", e)
    finally:
        print("Division attempt completed")

def main():
    print(safe_division())
```

## Development Workflow with uv

### Complete Project Setup

```bash
# Create new project with git author info
uv init --bare --author-from git my-mojo-app
cd my-mojo-app

# Create virtual environment
uv venv --python 3.11

# Activate environment
source .venv/bin/activate

# Add core dependencies
uv add modular numpy matplotlib

# Add development tools
uv add --optional dev ruff ty

# Create main.mojo file
echo 'def main(): print("Hello from Mojo!")' > main.mojo

# Install all dependencies including dev extras
uv pip install -r pyproject.toml --all-extras

# Run with activated environment
mojo main.mojo
```

### Code Quality with Ruff and Ty

```bash
# Add development tools
uv add --optional dev ruff ty

# Install dev dependencies
uv pip install -r pyproject.toml --all-extras

# Format and lint Python code
ruff format .          # Format code
ruff check .           # Lint and auto-fix
ruff check --fix .     # Apply automatic fixes

# Type checking with ty (faster alternative to mypy)
ty .                   # Type check all files
ty main.py            # Type check specific file
ty --watch .          # Watch mode for continuous checking
```

### Dependency Management Workflows

```bash
# Export current environment to requirements.txt
uv pip freeze > requirements.txt

# Compile requirements from pyproject.toml with version pinning
uv pip compile pyproject.toml -o requirements.txt

# Install from requirements.txt using pip interface
uv pip install -r requirements.txt

# Add dependencies from existing requirements
uv add -r requirements.txt

# Update all dependencies
uv pip install -r pyproject.toml --upgrade
```

## Best Practices

### 1. Choose the Right Function Type

- Use `def` for Python-like functions that may raise errors
- Use `fn` for systems programming with explicit error handling
- Use `@staticmethod` for utility functions on structs

### 2. Leverage Type System

```mojo
# Explicit typing for clarity
fn process_data(items: List[String]) -> Dict[String, Int]:
    var result = Dict[String, Int]()
    for item in items:
        result[item] = len(item)
    return result
```

### 3. Use Traits for Generic Programming

```mojo
trait Numeric:
    fn __add__(self, other: Self) -> Self: ...
    fn __mul__(self, other: Self) -> Self: ...

fn compute[T: Numeric](a: T, b: T) -> T:
    return a + b * a
```

### 4. Memory Safety

```mojo
# Use ownership annotations
fn process_owned(owned data: List[Int]) -> List[Int]:
    # data is consumed, no copying
    data.append(42)
    return data^  # Transfer ownership

fn process_borrowed(borrowed data: List[Int]) -> Int:
    # data is borrowed, read-only access
    return len(data)
```

## Current Limitations & Sharp Edges

### Missing Python Features

- No list/dict comprehensions yet: `[x for x in range(10)]`
- No `lambda` functions
- No `yield`/generators
- No classes (structs only)
- Limited exception hierarchy (`Error` instead of `Exception`)

### Workarounds

```mojo
# Instead of list comprehension
def create_squares(n: Int) -> List[Int]:
    var result = List[Int]()
    for i in range(n):
        result.append(i * i)
    return result

# Instead of lambda, use nested functions
def sort_with_key():
    fn key_func(x: Int) -> Int:
        return -x  # Reverse order
    # Use key_func as needed
```

### Type System Gotchas

```mojo
# StringLiteral vs String behaves differently
def example():
    var s1: String = "hello"      # String type
    var s2 = String("hello")      # String type  
    var s3 = "hello"              # StringLiteral type
    
    # Prefer explicit String conversion for consistency
    print(s1 or String("world"))  # Predictable behavior
```

## Module Organization

### Creating Modules

```mojo
# mathutils.mojo
fn add(a: Int, b: Int) -> Int:
    return a + b

fn multiply(a: Int, b: Int) -> Int:
    return a * b

@value
struct Vector2D:
    var x: Float64
    var y: Float64
    
    fn magnitude(self) -> Float64:
        return (self.x**2 + self.y**2)**0.5
```

### Using Modules

```mojo
# main.mojo
from mathutils import add, Vector2D
import mathutils

def main():
    result = add(5, 3)
    vec = Vector2D(3.0, 4.0)
    print("Magnitude:", vec.magnitude())
    
    # Alternative import style
    product = mathutils.multiply(4, 7)
```

## Testing

```mojo
from testing import assert_equal, assert_true

def test_basic_math():
    assert_equal(add(2, 3), 5)
    assert_true(Vector2D(3, 4).magnitude() == 5.0)

def main():
    test_basic_math()
    print("All tests passed!")
```

### Running Tests and Quality Checks

```bash
# Run Mojo tests
mojo test_*.mojo

# Code quality pipeline
ruff format .                    # Format code
ruff check --fix .              # Lint and auto-fix
ty .                            # Type check

# Watch mode for development
ty --watch .                    # Continuous type checking
ruff check --watch .            # Continuous linting
```

## Common Patterns

### Builder Pattern

```mojo
@value
struct Config:
    var debug: Bool
    var threads: Int
    var name: String
    
    @staticmethod
    fn builder() -> ConfigBuilder:
        return ConfigBuilder()

struct ConfigBuilder:
    var config: Config
    
    fn __init__(out self):
        self.config = Config(False, 1, "default")
    
    fn with_debug(mut self) -> Self:
        self.config.debug = True
        return self
    
    fn with_threads(mut self, count: Int) -> Self:
        self.config.threads = count
        return self
    
    fn build(owned self) -> Config:
        return self.config

def main():
    config = Config.builder().with_debug().with_threads(4).build()
```

### Resource Management with Context Managers

```mojo
@value
struct FileManager:
    var filename: String
    
    fn __enter__(self) -> Self:
        print("Opening file:", self.filename)
        return self
    
    fn __exit__(self):
        print("Closing file:", self.filename)

def main():
    with FileManager("data.txt") as f:
        print("Working with file")
    # File automatically closed
```

## Performance Tips

1. **Use `fn` for hot paths** - Better optimization
2. **Leverage SIMD** - Vectorize operations when possible
3. **Minimize Python calls in loops** - Cache Python objects
4. **Use `@parameter` decorators** - Enable compile-time optimizations
5. **Choose appropriate collection types** - `List` vs `VariadicList` vs raw pointers

## uv Configuration

### pyproject.toml Example

```toml
[project]
name = "my-mojo-project"
version = "0.1.0"
description = "A Mojo project with Python dependencies"
dependencies = [
    "modular",
    "numpy>=1.20.0,<2.0",
    "matplotlib>=3.5.0",
    "pandas>=1.5.0",
]

[project.optional-dependencies]
dev = [
    "ruff>=0.1.0",
    "ty>=0.1.0",
    "pytest>=7.0.0",
]

[tool.ruff]
# Configure ruff linting and formatting
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "W", "C90", "I", "N", "UP", "YTT", "S", "BLE", "FBT", "B", "A", "C4", "DTZ", "T10", "EM", "EXE", "ISC", "ICN", "G", "INP", "PIE", "T20", "PYI", "PT", "Q", "RSE", "RET", "SLF", "SIM", "TID", "TCH", "ARG", "PTH", "FIX", "ERA", "PD", "PGH", "PL", "TRY", "NPY", "RUF"]

[tool.ty]
# Configure ty type checker
strict = true
python-version = "3.11"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

### uv Commands Reference

```bash
# Project creation and setup
uv init --bare --author-from git my-project    # Create project with git author
uv venv --python 3.11                         # Create venv with specific Python
source .venv/bin/activate                     # Activate environment

# Dependency management
uv add package                                # Add runtime dependency
uv add --optional dev ruff ty                # Add dev dependencies
uv add -r requirements.txt                   # Add from requirements file
uv pip install -r pyproject.toml             # Install from pyproject.toml
uv pip install -r pyproject.toml --all-extras # Install with all extras

# Export and compilation
uv pip freeze > requirements.txt             # Export current env
uv pip compile pyproject.toml -o requirements.txt # Compile from pyproject

# Running and environment
uv run mojo main.mojo                        # Run with dependencies available
uv pip install --upgrade                     # Update all packages
```

## Code Quality Pipeline

### Pre-commit Setup

```bash
# Install development tools
uv add --optional dev ruff ty pre-commit

# Install pre-commit hooks
pre-commit install

# Run manually
ruff format .                    # Format all files
ruff check --fix .              # Lint and fix issues
ty .                            # Type check all files
```

### CI/CD Integration

```yaml
# .github/workflows/quality.yml
name: Code Quality
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v1
        with:
          version: "latest"
      
      - name: Install dependencies
        run: |
          uv venv
          source .venv/bin/activate
          uv pip install -r pyproject.toml --all-extras
      
      - name: Format check
        run: |
          source .venv/bin/activate
          ruff format --check .
      
      - name: Lint
        run: |
          source .venv/bin/activate
          ruff check .
      
      - name: Type check
        run: |
          source .venv/bin/activate
          ty .
```

## Getting Help

- **Documentation**: <https://docs.modular.com/mojo/>
- **Examples**: <https://github.com/modular/modular/tree/main/examples/mojo>
- **Recipes**: <https://builds.modular.com/?category=recipes>
- **Community**: Discord and Forum at <https://www.modular.com/community>
- **Issues**: <https://github.com/modular/modular/issues>
- **uv Documentation**: <https://docs.astral.sh/uv/>
- **Ruff Documentation**: <https://astral.sh/ruff>
- **Ty Documentation**: <https://github.com/astral-sh/ty>

Remember: Mojo is rapidly evolving. Always check the latest changelog for breaking changes and new features.
