# mpy-cross Installation and Usage

## Overview

`mpy-cross` is the official MicroPython cross-compiler that compiles Python source code (`.py` files) into bytecode (`.mpy` files) **offline**, without requiring a physical device. This makes it an essential tool for **fast syntax validation** during MicroPython development.

**Key Advantages:**
- **Fast syntax checking** - Validates code in milliseconds (vs seconds with device)
- **No device required** - Works completely offline
- **Detects MicroPython incompatibilities** - Catches type hints, f-strings, async issues
- **Generates optimized bytecode** - Creates `.mpy` files for faster imports
- **CI/CD ready** - Perfect for automated testing pipelines

## Use Cases

### 1. Quick Syntax Validation (Recommended)
```bash
# Verify syntax without creating .mpy file
mpy-cross script.py

# No error = code is syntactically valid
# Error = fix syntax issues
```

### 2. Batch File Validation
```bash
# Check all Python files in a project
mpy-cross src/*.py
```

### 3. Detecting CPython Incompatibilities
Automatically detects features not supported in MicroPython:
- Type hints (`def foo(x: int) -> str:`)
- F-strings in older MicroPython versions
- Advanced `async`/`await` patterns

### 4. Generating Optimized Bytecode
```bash
# Create .mpy file for faster imports and reduced memory
mpy-cross sensor_lib.py -o sensor_lib.mpy
```

### 5. CI/CD Integration
```yaml
# GitHub Actions example
- name: Validate MicroPython syntax
  run: |
    pip install mpy-cross
    mpy-cross src/**/*.py
```

## Installation

### Method 1: pip (Recommended)

**Install globally:**
```bash
pip install mpy-cross
```

**Verify installation:**
```bash
mpy-cross --version
# Expected output: MicroPython v1.x.x on YYYY-MM-DD; mpy-cross emitting vX
```

**Advantages:**
- Fast installation (one command)
- Suitable for most users
- Automatically added to PATH

**Limitations:**
- May not match exact MicroPython firmware version

### Method 2: From Source (For Version Matching)

Use this method if you need `mpy-cross` to match a specific MicroPython firmware version.

**Prerequisites:**
- Git
- C compiler (GCC, Clang, or MSVC)
- Make

**Installation steps:**

1. **Clone MicroPython repository:**
   ```bash
   git clone https://github.com/micropython/micropython.git
   cd micropython
   ```

2. **Checkout specific version (optional):**
   ```bash
   git checkout v1.23.0  # Match your device firmware version
   ```

3. **Build mpy-cross:**
   ```bash
   cd mpy-cross
   make
   ```

4. **Verify build:**
   ```bash
   ./mpy-cross --version
   ```

5. **Add to PATH (optional):**
   ```bash
   # Linux/macOS
   export PATH=$PATH:/path/to/micropython/mpy-cross
   
   # Windows PowerShell
   $env:PATH += ";C:\path\to\micropython\mpy-cross"
   ```

### Platform-Specific Notes

**Windows:**
- Use MSYS2 or MinGW for building from source
- Or install via pip (simpler)

**Linux:**
```bash
# Install dependencies first
sudo apt update
sudo apt install build-essential git
```

**macOS:**
```bash
# Install Xcode Command Line Tools
xcode-select --install
```

## Basic Usage

### Syntax Verification Only

**Check syntax without creating .mpy file:**
```bash
mpy-cross script.py
```

**What it checks:**
- Python syntax errors
- Indentation issues
- Import statement validity
- MicroPython compatibility (type hints, f-strings, etc.)

**What it doesn't check:**
- Module availability on device (use `mpremote` for this)
- Hardware compatibility (GPIO pins, I2C, etc.)
- Runtime errors (division by zero, etc.)

**Example:**
```bash
$ mpy-cross bad_syntax.py
bad_syntax.py:5: SyntaxError: invalid syntax
```

### Generate Bytecode Files

**Create .mpy file for deployment:**
```bash
# Simple conversion
mpy-cross sensor.py
# Creates: sensor.mpy

# Specify output file
mpy-cross sensor.py -o lib/sensor.mpy
```

**Use on device:**
```python
import sensor  # MicroPython loads sensor.mpy automatically
```

**Benefits:**
- Faster import times
- Reduced memory usage
- Protects source code (bytecode only)

### Advanced Options

```bash
# Target specific architecture (for native code)
mpy-cross script.py -march=armv7m  # For ESP32S3

# Show all options
mpy-cross --help
```

## Integration with mpremote

**Recommended two-stage verification workflow:**

### Stage 1: Offline Syntax Check (mpy-cross)
```bash
# Fast local validation
mpy-cross my_script.py
```

### Stage 2: Device Runtime Test (mpremote)
```bash
# Test on actual hardware
mpremote run my_script.py
```

**Complete workflow:**
```bash
# 1. Quick syntax verification (0.5 seconds)
mpy-cross wifi_sensor.py

# 2. Device testing (5-10 seconds)
mpremote connect auto
mpremote run wifi_sensor.py

# 3. Deploy if tests pass
mpremote fs cp wifi_sensor.py :main.py
mpremote reset
```

**When to use each tool:**

| Tool | Use When | Speed | Requires Device |
|------|----------|-------|----------------|
| `mpy-cross` | Syntax validation, batch checking | Milliseconds | No |
| `mpremote` | Hardware testing, module checks | Seconds | Yes |

## Common Errors and Solutions

### Error: `can't compile: script.py`

**Cause:** File not found or invalid path

**Solution:**
```bash
# Check file exists
ls script.py

# Use absolute path
mpy-cross /full/path/to/script.py
```

### Error: `SyntaxError: invalid syntax`

**Cause:** Python syntax error or MicroPython incompatibility

**Solution:**
```python
# Type hints (not supported in MicroPython)
def foo(x: int) -> str:
    return str(x)

# Remove type hints
def foo(x):
    return str(x)
```

### Error: MicroPython incompatibility detected

**Common incompatibilities:**

| Feature | CPython | MicroPython | Solution |
|---------|---------|-------------|----------|
| Type hints | Yes | No | Remove `: type` annotations |
| F-strings | Yes | v1.12+ | Use `.format()` for older versions |
| `async`/`await` | Yes | Limited | Check firmware support |
| Some stdlib | Yes | No | Use MicroPython equivalents (`urequests`, `ujson`) |

### Error: `mpy-cross: command not found`

**Solution:**
```bash
# Check if pip installed it
pip show mpy-cross

# If not found, install
pip install mpy-cross

# Verify PATH
which mpy-cross  # Linux/macOS
where mpy-cross  # Windows
```

### Warning: Version mismatch with firmware

**Check versions:**
```bash
# Device firmware version
mpremote eval "import sys; print(sys.implementation)"

# mpy-cross version
mpy-cross --version
```

**Solution if mismatched:**
- Build `mpy-cross` from source matching firmware version
- Or update device firmware to match `mpy-cross`

## CI/CD Integration

### GitHub Actions Example

```yaml
name: MicroPython Syntax Check

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      
      - name: Install mpy-cross
        run: pip install mpy-cross
      
      - name: Validate syntax
        run: |
          # Check all .py files
          find . -name "*.py" -exec mpy-cross {} \;
      
      - name: Report results
        if: failure()
        run: echo "MicroPython syntax validation failed"
```

### Pre-commit Hook

```bash
# .git/hooks/pre-commit
#!/bin/bash
echo "Validating MicroPython syntax..."

for file in $(git diff --cached --name-only --diff-filter=ACM | grep '\.py$'); do
    mpy-cross "$file" || {
        echo "Syntax error in $file"
        exit 1
    }
done

echo "All files validated"
```

### Makefile Integration

```makefile
.PHONY: validate
validate:
	@echo "Validating MicroPython syntax..."
	@find src -name "*.py" -exec mpy-cross {} \;
	@echo "Validation complete"

.PHONY: build-mpy
build-mpy:
	@mkdir -p dist
	@find src -name "*.py" -exec sh -c \
		'mpy-cross "$$1" -o "dist/$$(basename $$1 .py).mpy"' _ {} \;
	@echo "Built .mpy files in dist/"
```

## Best Practices

### 1. Always Validate Before Device Testing
```bash
# Good workflow
mpy-cross script.py && mpremote run script.py

# Bad workflow (wastes time if syntax error)
mpremote run script.py
```

### 2. Validate All Files in Batch
```bash
# Check entire project
mpy-cross src/**/*.py lib/**/*.py
```

### 3. Use in Development Loop
```bash
# Watch for changes and auto-validate
while inotifywait -e modify src/*.py; do
    mpy-cross src/*.py
done
```

### 4. Match Firmware Versions
```bash
# Check device firmware
mpremote eval "import sys; print(sys.version)"

# Build matching mpy-cross
git clone https://github.com/micropython/micropython.git
cd micropython && git checkout v1.23.0
make -C mpy-cross
```

### 5. Don't Commit .mpy Files
```bash
# .gitignore
*.mpy
```

Bytecode is architecture-specific and should be regenerated per deployment.

## Troubleshooting

### mpy-cross hangs or crashes

**Cause:** Very large file or memory issue

**Solution:**
- Split large files into modules
- Increase system memory
- Check for infinite loops in global scope

### Compiled .mpy won't load on device

**Cause:** Version mismatch between `mpy-cross` and device firmware

**Solution:**
```bash
# Check compatibility
mpremote eval "import sys; print(sys.implementation._mpy)"
mpy-cross --version

# Rebuild with matching version
```

### Permission denied error

**Solution:**
```bash
# Linux/macOS
chmod +x /path/to/mpy-cross

# Or use pip-installed version
pip install --user mpy-cross
```

## Comparison: mpy-cross vs mpremote

| Feature | mpy-cross | mpremote |
|---------|-----------|----------|
| **Purpose** | Syntax validation, bytecode generation | Device interaction, runtime testing |
| **Speed** | Milliseconds | Seconds |
| **Requires device** | No |  Yes |
| **Detects** | Syntax errors, incompatibilities | Runtime errors, missing modules |
| **Use for** | Quick validation, CI/CD | Hardware testing, deployment |
| **Best time** | During development | Before deployment |

**Use both for complete verification:**
1. `mpy-cross` - Fast syntax check (every save)
2. `mpremote` - Device test (before commit)

## Links

- **Official Documentation**: [MicroPython mpy-cross](https://docs.micropython.org/en/latest/reference/mpyfiles.html)
- **Source Code**: [GitHub - micropython/mpy-cross](https://github.com/micropython/micropython/tree/master/mpy-cross)
- **PyPI Package**: [mpy-cross on PyPI](https://pypi.org/project/mpy-cross/)
- **MicroPython Forums**: [forum.micropython.org](https://forum.micropython.org/)

## Next Steps

After installing `mpy-cross`:

1. [Validate your first script](#basic-usage): `mpy-cross blink.py`
2. [Integrate with workflow](#ci-cd-integration): Add to pre-commit hooks or CI/CD
3. [Test on device](#integration-with-mpremote): Use `mpremote` for hardware verification

For complete XIAO MicroPython development workflow, see the main [SKILL.md](../../SKILL.md).
