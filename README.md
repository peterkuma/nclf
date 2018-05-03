# NCLF

**Development status:** proposal

New Command Line Format (NCLF) is a command line format specification aiming
to be a more universal replacement for the conventional unix command line
format of passing arguments. It has a direct and unambiguous mapping
to JSON and aims to be more compatible with standard function calling interfaces
in programming languages. JSON-formatted argument content is
supported.

The idea behind NCLF is to help remove the distinction between running
command line programs and calling functions in programming languages.

NCLF supports safe passing of arbitrary binary arguments via encoding as JSON
strings.

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

## Boolean arguments

```sh
program +xyz ++abc
```

```json
["", {"x": true, "y": true, "z": true, "abc": true}]
```

### String encoding

```sh
program '"x=1"'
```

```json
["x=1", {}]
```

## Literal positional arguments after =

```sh
program = x=1
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
to JSON. If the attempt fails, attempt is made to decode the value as a JSON
string. Failing that, the value is assumed to be a string as is.

Positional arguments which would otherwise be recognised as named arguments
can be encoded as JSON string, e.g. '"x=1"' in Bourne shell will ensure
"x=1" is passed to the process, interpreted as a JSON string instead of
{"x": 1}. To ensure file names or user-supplied arguments are not interpreted
as named arguments, we suggest using the tool
[as_s](https://github.com/peterkuma/nclf-python#as_s), e.g.:

```sh
program $(as_s *)
```

as_s will encode all of its arguments as JSON strings and output a list
of arguments compatible with Bourne shell, handling binary arguments
properly.

**Note:** Any untrusted string input should be passed either inside a JSON
value or via as_s. as_s supports arbitrary binary values.

**Note:** UTF-8 encoded unicode strings can be passed as values, and if
properly encoded they are decoded as JSON strings. If not properly encoded,
they are treated as binary strings. Literal strings following '=' are always
binary, as are strings passed via `as_s`.

Alternatively, positional arguments can be passed after `=`, in which case
they are treated as string literals without any interpretation.

Arguments starting with `+` followed by one or more `[a-zA-Z0-9]` characters
are short boolean named arguments with characters corresponding to names.

Arguments starting with `++` followed by a valid name are long boolean named
arguments.

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

### Command line

[nclf-python](https://github.com/peterkuma/nclf-python) contains a command
line program `nclf` which decodes NCLF-formatted commmand line arguments
and outputs the resulting JSON to the standard output.

## More examples

What some common commands would look like if they were using NCLF:

```sh
grep --help
grep ++help
```

```sh
ls -l -- *
ls +l = *
# Note that ls -l * could fail with arbitrarily named files.
```

```sh
tar xzf archive.tar.gz
tar +xz f=archive.tar.gz
```

## Conventional argument passing issues

Conventional argument passing has numerous issues, which NCLF tries to address:

- Commands such as `program -x argument` have ambiguous interpretation,
`argument` can be either positional or the value of `-x`. This limits
interoperability with other interfaces.

- `--` to separate literal arguments is not widely implemented and used.

- Conventional argument passing is not as powerful as function calling,
limiting the use of command line programs.

- The burden of decoding argument values is left on the program, and is often
not clearly defined.

## License

The NCLF specification is public domain.
