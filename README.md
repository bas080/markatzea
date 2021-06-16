# markatzea

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

## Requirements

Markatzea requires bash and some common utilities to run.

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

Here we have the hello world c program:

```c cat - > ./hello-world.c
#include <stdio.h>

int main() {
   printf("Hello World!\n");
   return 0;
}
```

Before we can run this program we must first compile it and then we clean it
up.

```bash bash 2>&1
gcc -o ./hello-world.out ./hello-world.c
chmod -v +x ./hello-world.out
./hello-world.out
rm -v ./hello-world.*
```
```
mode of './hello-world.out' retained as 0775 (rwxrwxr-x)
Hello World!
removed './hello-world.c'
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


```js node 2>&1 || true
throw new Error('Something unexpected happened.')
```
```
[stdin]:1
throw new Error('Something unexpected happened.')
^

Error: Something unexpected happened.
    at [stdin]:1:7
    at Script.runInThisContext (node:vm:129:12)
    at Object.runInThisContext (node:vm:305:38)
    at node:internal/process/execution:81:19
    at [stdin]-wrapper:6:22
    at evalScript (node:internal/process/execution:80:60)
    at node:internal/main/eval_stdin:29:5
    at Socket.<anonymous> (node:internal/process/execution:212:5)
    at Socket.emit (node:events:402:35)
    at endReadableNT (node:internal/streams/readable:1343:12)
```

### No output

```js node
process.exit(0)
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
chmod -v +x ./hello-world.sh
```
```
mode of './hello-world.sh' changed from 0664 (rw-rw-r--) to 0775 (rwxrwxr-x)
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

Markatzea does not solve templating itself. We enable templating by using
a cli templating tool named `memplate`. You can find it here: TODO

We'll write a little node program to showcase using markatzea with `memplate`.

First the modules the program depends on:

```js memplate assert-import
const assert = require('assert')
```

Next is a helper function:

```js memplate always-fn
const always = x => () => x
```

And finally a test.

```js memplate always-fn-test

const fourtyTwo = always(42)

assert.equal(fourtyTwo(), 42)
assert.equal(fourtyTwo(), 43)
```

Let's see the result

```js memplate result
<assert-import
<always-fn
<always-fn-test
```

```js memplate
<result
```
```
const assert = require('assert')
const always = x => () => x

const fourtyTwo = always(42)

assert.equal(fourtyTwo(), 42)
assert.equal(fourtyTwo(), 43)
```

We can run the program by piping the output of memplate to node.

```bash memplate | node 2>&1 || true
<result
```
```
node:assert:123
  throw new AssertionError(obj);
  ^

AssertionError [ERR_ASSERTION]: 42 == 43
    at [stdin]:7:8
    at Script.runInThisContext (node:vm:129:12)
    at Object.runInThisContext (node:vm:305:38)
    at node:internal/process/execution:81:19
    at [stdin]-wrapper:6:22
    at evalScript (node:internal/process/execution:80:60)
    at node:internal/main/eval_stdin:29:5
    at Socket.<anonymous> (node:internal/process/execution:212:5)
    at Socket.emit (node:events:402:35)
    at endReadableNT (node:internal/streams/readable:1343:12) {
  generatedMessage: true,
  code: 'ERR_ASSERTION',
  actual: 42,
  expected: 43,
  operator: '=='
}
```

By combining memplate and markatzea we can now split our code into readable
blocks and glue them together at a later moment in time.
