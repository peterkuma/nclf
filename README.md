# NCLF

New Command Line Format (NCLF) is a command line format specification aiming
to be a more universal replacement for the conventional unix command line
format of passing arguments. It has a direct and unambiguous mapping
to JSON and aims to be more compatible with standard function calling interfaces
in programming languages. JSON-formatted argument content is
supported.

The idea behind NCLF is to help remove the distinction between running
command line programs and calling functions in programming languages.

**Note:** NCLF does not depend on wide adoption. Use it in your own programs
if you like it and ignore otherwise. The intention is to make unix even better.

## Examples

### Two positional arguments

```sh
program hello world
```

```json
[["hello", "world"], {}]
```

### One named argument and two positional arguments

```sh
program x=1 a b
```

```json
[["a", "b"], { "x": 1 }]
```

### JSON array as a positional argument

```sh
program [1,2,3]
```

```json
[[1, 2, 3], {}]
```

## JSON object as named argument

```sh
program x='{"y": "z"}'
```

```json
[[], {"x": {"y": "z"}}]
```

### String encoding

```sh
program '"x=1"'
```

```json
["x=1", {}]
```

## Description

Command line arguments can be positional or named. The resulting JSON
is a array of two elements:

1. array of positional arguments
2. object containing named arguments

Named arguments are in the format name=value, where name is a string matching
`[a-zA-Z_][a-zA-Z0-9_]*`.

Attempt is made to convert every positional argument and named argument value
to JSON. If the attempt fails, the value is assumed to be a string.

Positional arguments which would otherwise be recognised as named arguments
can be encoded as JSON string, e.g. '"x=1"' in Bourne shell will ensure
"x=1" is passed to the process, interpreted as a JSON string instead of
{"x": 1}. To ensure file names or user-supplied arguments are not interpreted
as named arguments, we suggest using the tool
[as_s](https://github.com/peterkuma/as_s), e.g.:

```sh
program $(as_s *)
```

as_s will encode all of its arguments as JSON strings and output a list
of arguments compatible with Bourne shell.

## Implementations

### Python

Currently NCLF is implemented by the
[nclf-python](https://github.com/peterkuma/nclf-python) package:

```python
import sys
from nclf import nclf

args = nclf(sys.argv)
print(args)
```

will decode NCLF-formatted command line arguments and print the result.

## License

The NCLF specification is public domain.
