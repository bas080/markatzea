# markatzea

TODO
- Replace placeholder with output of code block
  - https://unix.stackexchange.com/questions/49377/substitute-pattern-within-a-file-with-the-content-of-other-file


Markatzea enhances your markdown by running your code-blocks and printing out
the output of those commands.

It should/does not break your existing markdown. It adds a little syntax for
defining the interpreter for that code block. This is defined after the
code-block's language name. The code-block is then passed as stdin into the
interpreter configured for that code-block.

## Features

- Weaving code-blocks together.
- Evaluating (weaved) code blocks with your desired interpreter.
- Writing code blocks or the output of evaluation to files.
- Use what you know about bash to enable all kinds of features.

## Examples

### Hello World

```bash bash
echo 'hello bash'
```
```
hello bash
```

```python python
print('hello python')
```
```
hello python
```

```js node -p
'hello javascript'
```
```
hello javascript
```
> You can use `node -p` to print the value of the last js expression. No need to
> write `console.log(...)`.

```perl perl
print("hello perl\n")
```
```
hello perl
```

```lua luajit
print("hello lua")
```
```
hello lua
```

> Write to file without writing to stdout.

```c cat - > ./hello-world.c
#include <stdio.h>

int main() {
   printf("Hello World!");
   return 0;
}
```
```
```

```bash bash
gcc -o ./hello-world.out ./hello-world.c
chmod +x ./hello-world.out
./hello-world.out
rm -v ./hello-world.*
```
```
Hello World!removed './hello-world.c'
removed './hello-world.out'
```

You get the hello world idea. Now for some other examples.

### Write code-blocks to a file

We'll use `tee` for this example.

```bash tee ./example_tmp_file
temporary file contents
```
```
temporary file contents
```

Check if the temporary file has the desired content.

```bash bash
cat ./example_tmp_file
```
```
temporary file contents
```

Remove the temporary file.

```bash bash
rm -v ./example_tmp_file
```
```
removed './example_tmp_file'
```

### Oneliners

Besides defining a single interpreter, you can create all kinds of oneliners.

A pipeline:

```bash bash | bash
echo 'echo pipeline'
```
```
pipeline
```

```bash bash || true
exit 1
```
```
```

> Process exits with zero because of `|| true`.

```bash cat - && echo world
hello
```
```
hello
world
```

Markatzea allows you to write long oneliners which are used as the
"interpreter". I would however advice you to keep the interpreter
configurations as short as possible.for the sake of easy of reasoining. Ideally
a single process. When you needs are more complex, make your own scripts and
use those as the interpreter.

### Non zero exitcode

```bash bash 2>&1 || true
print_hello_and_error() {
  echo hello
  some_command_that_does_not_exist
  return 42
}

print_hello_and_error
```
```
hello
main: line 3: some_command_that_does_not_exist: command not found
```

> Notice that we redirect stderr to stdout. This is usefull whenever we want to
> demo not just the stdout but also the stderr output.

It will print stdout and stderr and shows the exit code by acting as if it ran
`echo $?`. Let's see that again.

```node node 2>&1 || true
throw new Error('Something unexpected happened.')
```
```
[stdin]:1
throw new Error('Something unexpected happened.')
^

Error: Something unexpected happened.
    at [stdin]:1:7
    at Script.runInThisContext (node:vm:131:12)
    at Object.runInThisContext (node:vm:308:38)
    at node:internal/process/execution:81:19
    at [stdin]-wrapper:6:22
    at evalScript (node:internal/process/execution:80:60)
    at node:internal/main/eval_stdin:29:5
    at Socket.<anonymous> (node:internal/process/execution:209:5)
    at Socket.emit (node:events:377:35)
    at endReadableNT (node:internal/streams/readable:1312:12)
```

### No output

```node node
process.exit(0)
```
```
```

### Creating a script file and running it

A more practical use-case could be generating a script file and running/testing
it.

```bash tee ./hello-world.sh
#!/usr/bin/env bash

echo hello world
```
```
#!/usr/bin/env bash

echo hello world
```

```bash bash
chmod +x ./hello-world.sh
```
```
```

Test if the script works as expected.

```bash bash
test "$(./hello-world.sh)" = "hello world" &&
  echo "ok" &&
  rm -v ./hello-world.sh
```
```
ok
removed './hello-world.sh'
```

### Code block aliases

```node cat # always
const always = x => () => x
```
```
const always = x => () => x
```

```node node -p
# always

const 42 = always(42)

42()
```
```
