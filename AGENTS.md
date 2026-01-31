# expiringdict Project Guide

This file contains essential information for AI coding agents working on the expiringdict project.

## Project Overview

**expiringdict** is a Python caching library providing an ordered dictionary with auto-expiring values. It was originally developed by Mailgun Technologies Inc.

The core class `ExpiringDict` extends `OrderedDict` to provide:
- Automatic expiration of values based on time-to-live (TTL)
- Maximum capacity enforcement (oldest items evicted when max_len is exceeded)
- Thread-safe operations using RLock
- Pickle support for serialization

### Key Characteristics

- **Version**: 1.2.2
- **License**: Apache License 2.0
- **Python Support**: Python 2.7, Python 3.6+
- **Package Type**: Single-module library

## Project Structure

```
expiringdict/
├── expiringdict/
│   └── __init__.py          # Main module containing ExpiringDict class
├── tests/
│   ├── __init__.py          # Empty test package init
│   ├── expiringdict_test.py # Core functionality tests
│   └── expiringdict_extended_test.py  # Pickle and copy tests
├── setup.py                 # Package configuration
├── MANIFEST.in              # Distribution manifest
├── README.rst               # Documentation (reStructuredText)
├── CHANGELOG.rst            # Version history
├── LICENSE                  # Apache 2.0 license
└── .travis.yml              # CI configuration
```

## Technology Stack

### Core Dependencies
- **Python**: 2.7 or 3.6+
- **OrderedDict**: From `collections` (standard library), with fallback to `ordereddict` package for Python < 2.7
- **typing**: Backport for Python < 3.5 (conditional dependency)
- **threading.RLock**: For thread-safe operations

### Development Dependencies
- **nose**: Test framework
- **mock**: Mocking library (Python 2 compatibility)
- **coverage**: Code coverage measurement
- **coveralls**: Coverage reporting service
- **dill**: Extended pickle support (for serialization tests)

## Build and Test Commands

### Installation

```bash
# Install from source
pip install -e .

# Install with test dependencies
pip install -e .[tests]
```

### Testing

```bash
# Run tests with nose
nosetests

# Run tests with coverage
nosetests --with-coverage --cover-package=expiringdict
```

### CI/CD

The project uses **Travis CI** (configured in `.travis.yml`):
- Tests run on Python 2.7 and 3.6
- Coverage reported to Coveralls
- Build status badge displayed in README

## Code Style Guidelines

### Python Version Compatibility
- Code must be compatible with Python 2.7 and Python 3.6+
- Use try/except for import compatibility (e.g., `OrderedDict`, `md5`)
- Use type hints in comments for Python 2 compatibility:
  ```python
  def __init__(self, max_len, max_age_seconds, items=None):
      # type: (Union[int, None], Union[float, None], Union[None,dict,OrderedDict,ExpiringDict]) -> None
  ```

### Naming Conventions
- Class names: `CamelCase` (e.g., `ExpiringDict`)
- Method/variable names: `snake_case`
- Private methods: double underscore prefix (e.g., `__copy_expiring_dict`)

### Documentation
- Use docstrings with triple double quotes
- Include doctests in module docstring
- Keep README.rst in sync with code changes (reStructuredText format)

### Thread Safety
- All mutating operations must use `self.lock` (RLock)
- Use `with self.lock:` context managers
- Be careful with recursive lock acquisition

## Architecture Details

### Data Storage Format

Values are stored as tuples: `{key: (value, created_time)}`

Where:
- `value`: The actual cached value
- `created_time`: Unix timestamp from `time.time()`

### Key Classes and Methods

**ExpiringDict** (extends OrderedDict):

- `__init__(max_len, max_age_seconds, items=None)` - Constructor
- `__getitem__(key, with_age=False)` - Get value (expires if stale)
- `__setitem__(key, value, set_time=None)` - Set value
- `__contains__(key)` - Check key exists (expires if stale)
- `get(key, default=None, with_age=False)` - Safe get with default
- `pop(key, default=None)` - Get and remove
- `ttl(key)` - Return remaining time-to-live
- `items()` - Return non-expired items
- `items_with_timestamp()` - Return raw items with timestamps
- `values()` - Return non-expired values

### Expiration Strategy

1. **Lazy expiration**: Items are only expired on access
2. **Access triggers expiration check**: `__getitem__`, `__contains__`, iteration methods
3. **Manual cleanup not required**: No background thread

**Note**: Iteration over `keys()` does NOT remove expired values (documented limitation).

### Eviction Strategy

When `max_len` is reached:
1. If key exists: remove old entry, add new
2. If key is new: pop oldest item (FIFO from OrderedDict), then add new

## Testing Strategy

### Test Organization

1. **expiringdict_test.py**: Core functionality
   - Creation and validation
   - Basic operations (get, set, contains)
   - Expiration behavior
   - Pop operations
   - TTL functionality
   - Iteration
   - Edge cases

2. **expiringdict_extended_test.py**: Extended features
   - Pickle serialization with dill
   - Copy from regular dict
   - Copy from ExpiringDict
   - Override max_len/max_age on copy

### Testing Patterns

- Use `nose.tools` assertions: `eq_`, `ok_`, `assert_raises`
- Use `time.sleep()` for expiration testing (small delays like 0.01s)
- Use `mock.patch` for mocking time-dependent behavior
- Tests should be fast and deterministic

### Running Specific Tests

```bash
# Run specific test file
nosetests tests/expiringdict_test.py

# Run specific test
nosetests tests/expiringdict_test.py:test_create
```

## Security Considerations

### Thread Safety
- The library uses `RLock` for thread safety
- Deadlocks are prevented by RLock's reentrancy
- Be cautious when modifying locking behavior

### Resource Limits
- `max_len` enforces memory bounds
- `max_age_seconds` prevents stale data accumulation
- No automatic cleanup thread (memory-safe for long-running processes)

### Input Validation
- Assertions validate `max_len >= 1` and `max_age_seconds >= 0`
- Invalid inputs raise `AssertionError` or `ValueError`

## Release Process

1. Update version in `setup.py`
2. Update `CHANGELOG.rst` with release date and changes
3. Tag release in git
4. Build and upload to PyPI:
   ```bash
   python setup.py sdist bdist_wheel
   twine upload dist/*
   ```

## Common Tasks for Agents

### Adding New Features
1. Implement in `expiringdict/__init__.py`
2. Add tests in appropriate test file
3. Update README.rst with usage examples
4. Update CHANGELOG.rst

### Fixing Bugs
1. Add regression test first
2. Fix in main module
3. Ensure all existing tests pass
4. Update CHANGELOG.rst

### Refactoring
1. Maintain Python 2/3 compatibility
2. Preserve thread safety
3. Keep public API backward compatible
4. Run full test suite before committing
