<p align="center">
Cheat sheet for xonsh shell with copy-pastable examples.
</p>

<p align="center">
If you like the cheatsheet click ⭐ on the repo and stay tuned.
</p>

<p align="center">
Work in progress. PRs are welcome.
</p>

# Operators

### `$()`

Captures stdout and returns output with [universal new lines](https://www.python.org/dev/peps/pep-0278/):
```python
aliases['args'] = lambda args: print(args)

args $(echo -e '1\n2\r3 4\r\n5')       # Subproc mode
#['1\n2\n3 4\n5\n']

output = $(echo -e '1\n2\r3 4\r\n5')   # Python mode 
output
#'1\n2\n3 4\n5\n'
```

### `!()`

Captures stdout and returns [CommandPipeline](http://xon.sh/api/proc.html#xonsh.proc.CommandPipeline). Truthy if successful (returncode == 0), compares to, iterates over lines of stdout:
  
```python
ret = !(echo 123)
ret
#CommandPipeline(
#  pid=404136,                                                                                                     
#  returncode=0,                                                                                                   
#  args=['echo', '123'],                                                                                           
#  alias=None,                                                                                                     
#  timestamps=[1604742882.1826484, 1604742885.1393967],                                                            
#  executed_cmd=['echo', '123'],                                                                                   
#  input='',                                                                                                       
#  output='123\n',                                                                                                 
#  errors=None
#)   

if ret:
      print('Success')     
#Success

for l in ret:
      print(l)     
#123
#

```

### `$[]` 

Passes stdout to the screen and returns `None`:

```python
ret = $[echo 123]
#123
repr(ret)
'None'
```

### `![]`

Passes stdout to the screen and returns [HiddenCommandPipeline](https://xon.sh/api/proc.html#xonsh.proc.HiddenCommandPipeline):

```python
ret = ![echo -e '1\n2\r3 4\r\n5']
#1
#3 4
#5
ret               # No representation, no return value because it's hidden CommandPipeline object
ret.out           # But it has the properties from CommandPipeline
'1\n2\r3 4\n5\n'
```

### `@()`

Evaluates Python and pass the arguments:

```python
aliases['args'] = lambda args: print(args)

args 'Supported:' @('string') @(['list','of','strings']) 
#['Supported:', 'string', 'list', 'of', 'strings']

echo -n '!' | @(lambda args, stdin: 'Callable in the same form as callable aliases'+ stdin.read())
#Callable in the same form as callable aliases!!!
```

### `@$()`

Split output of the command by whitespaces:

```python
aliases['args'] = lambda args: print(args)

args @$(echo -e '1\n2\r3 4\r\n5')
#['1', '2\r3', '4', '5']
```

# Environment Variables

```python
aliases['args'] = lambda args: print(args)

$VAR = 'value'    # Set environment variable

'VAR' in ${...}   # Check environment variable exists
#True

${'V' + 'AR'}     # Get environment variable value by name from expression
#'value'

print($VAR)
with ${...}.swap(VAR='another value'):   # Change value for commands block
    print($VAR)
print($VAR)
#value
#another value
#value

$VAR='new value' xonsh -c r'echo $VAR'   # Change value for subproc command
#new value

```

See also the list of [xonsh default environment variables](http://xon.sh/envvars.html).

## Shell syntax
* **`|`**: Shell-style pipe
* **`and`**, **`or`**: Logically joined commands, lazy
* **`&&`**, **`||`**: Same
* **`COMMAND &`**: Background into job (May use `jobs`, `fg`, `bg`)
* **`>`**: Write (stdout) to
* **`>>`**: Append (stdout) to
* **`</spam/eggs`**: Use file for stdin
* **`out`**, **`o`**
* **`err`**, **`e`**
* **`all`**, **`a`** (left-hand side only)

```
>>> COMMAND1 e>o < input.txt | COMMAND2 > output.txt e>> errors.txt
```

# [Advanced String Literals](https://xon.sh/tutorial.html#advanced-string-literals)

* **`"foo"`**: Regular string: backslash escapes
* **`f"foo"`**: Formatted string: brace substitutions, backslash escapes
* **`r"foo"`**: Raw string: unmodified
* **`p"foo"`**: Path string: backslash escapes, envvar substitutions, returns `Path`
* **`pr"foo"`**: Raw Path string: envvar substitutions, returns `Path`
* **`pf"foo"`**: Formatted Path string: backslash escapes, brace substitutions, envvar substitutions, returns `Path`
* **`fr"foo"`**: Raw Formatted string: brace substitutions

# [Globbing](https://xon.sh/tutorial.html#normal-globbing)
[Normal globbing](https://xon.sh/tutorial.html#normal-globbing):
```python
ls *.*
# or
ls g`*.*`
# or return path instances:
for f in gp`.*`:
      print(f.exists())
```
[Regular Expression Globbing](https://xon.sh/tutorial.html#regular-expression-globbing):
```python
ls `.*`
# or
ls r`.*`
# or return path instances:
for f in rp`.*`:
      print(f.exists())
```
[Custom function globbing](https://xon.sh/tutorial.html#custom-path-searches):
```python
def foo(s):
    return [i for i in os.listdir('.') if i.startswith(s)]
cd /
@foo`bi`
#['bin']
```

# [Aliases](https://xon.sh/tutorial.html#aliases)
* `aliases['g'] = 'git status -sb'`
* `aliases['gp'] = ['git', 'pull']`

# [Callable aliases](https://xon.sh/tutorial.html#callable-aliases)

```python
def _myargs1(args):
#def _myargs2(args, stdin=None):
#def _myargs3(args, stdin=None, stdout=None):
#def _myargs4(args, stdin=None, stdout=None, stderr=None):
#def _myargs5(args, stdin=None, stdout=None, stderr=None, spec=None):
#def _myargs6(args, stdin=None, stdout=None, stderr=None, spec=None, stack=None):
    print(args)
    
aliases['args'] = _myargs1
del _myargs1

args 1 2 3
#['1', '2', '3']
```
or:
```python
aliases['args'] = lambda args: print(args)

args 1 2 3
#['1', '2', '3']
```

# Macros

## [Simple macros](https://xon.sh/tutorial_macros.html#function-macros)

```python
def identity(x : str):
    return x

# No macro call

identity('me')
# 'me'

identity(42)
# 42

identity(identity)
# <function __main__.identity>

# Macro call

identity!('me')
# "'me'"

identity!(42)
# '42'

identity!(identity)
# 'identity'

identity!(42)
# '42'

identity!(  42 )
# '42'

identity!(import os)
# 'import os'

identity!(if True:
    pass)
# 'if True:\n    pass'
```

## [Subprocess Macros](https://xon.sh/tutorial_macros.html#subprocess-macros)

```python
echo! "Hello!"
# "Hello!"

bash -c! echo "Hello!"
# Hello!

docker run -it --rm xonsh/xonsh:slim xonsh -c! 2+2
# 4
```
Inside of a macro, all additional munging is turned off:
```

echo $USER
# lou

echo! $USER
# $USER

```

## [Macro block](https://xon.sh/tutorial_macros.html#context-manager-macros)

```python
import json

class JsonBlock:
    __xonsh_block__ = str

    def __enter__(self):
        return json.loads(self.macro_block)

    def __exit__(self, *exc):
        del self.macro_block, self.macro_globals, self.macro_locals


with! JsonBlock() as j:
    {
        "Hello": "world!"
    }
    
j['Hello']
# world!
```

# [Tab-Completion](https://xon.sh/tutorial_completers.html)

```python
def dummy_completer(prefix, line, begidx, endidx, ctx):
    '''
    Completes everything with options "lou" and "carcolh",
    regardless of the value of prefix.
    '''
    return {"lou", "carcolh"}
    
'''
Add completer: completer add NAME FUNC
'''
completer add dummy dummy_completer
```

# Xonsh Script (xsh)

* **`$ARGS`**: List of all command line parameter arguments.
* **`$ARG0`, `$ARG1`, ... `$ARG9`**: Script command line argument at index n.

# Xontrib

Xontrib list: [github topic](https://github.com/topics/xontrib), [github repos](https://github.com/search?q=xontrib-&type=repositories), [official list](https://xon.sh/xontribs.html).

Create xontrib [using cookiecutter template](https://github.com/xonsh/xontrib-cookiecutter):
```
pip install cookiecutter
cookiecutter gh:xonsh/xontrib-cookiecutter
```

# [Help](https://xon.sh/tutorial.html#help-superhelp-with)
Add `?` (regular help) or `??` (super help) to the command:
```python
ls?
# man page for ls

import json
json?
# json module help
json??
# json module super help
```

# Credits
* [Xonsh Tutorial](https://xon.sh/tutorial.html)
* Most copy-pastable examples prepared by [xontrib-hist-format](https://github.com/anki-code/xontrib-hist-format)
* The first short version of the cheat sheet was made by @AstraLuma. Thanks!
