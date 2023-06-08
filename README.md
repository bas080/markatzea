# markatzea

Markatzea enhances your markdown by running your code-blocks and printing out
the output of those commands.

It should/does not break your existing markdown. It adds a little syntax for
defining the interpreter for that code block. This is defined after the
code-block's language name. The code-block is then passed as stdin into the
interpreter configured for that code-block.

## Usage

```bash
pod2text markatzea
```
```
NAME
    markatzea - evaluate your markdown code blocks

SYNOPSIS
    markatzea <file>

DESCRIPTION
    markatzea is a tool which takes markdown, evaluates code blocks with
    interpreters and prints the output of those processes to a different
    codeblock.

```

## Examples

### Normal code-block

When no interpreter is defined, markatzea will print the markdown as is.

    ```bash
    echo 'Does not evaluate this bash code-block'
    ```

```bash
echo 'Does not evaluate this bash code-block'
```

### Evaluated code-block

    ```bash bash
    echo 'Does evaluate this bash code-block with bash'
    ```

```bash
echo 'Does evaluate this bash code-block with bash'
```
```
Does evaluate this bash code-block with bash
```

### Output Language

You can define the language to use for the output code block.

    ```bash|javascript bash
    echo 'const value = 42;'
    ```

```bash
echo 'const value = 42;'
```
```javascript
const value = 42;
```

### Literate Programming

You can achieve a form of a literate programming using a template language that
offers a command line interface. See [memplate][2] for more information.

An example:

```bash
set -eo pipefail
```

Now we use the aliased template in another template.

```bash
<sane-bash-defaults

ls not-a-file |
  cat - ||
  echo 'Good! The ls process caused the pipe to stop.'
```
```
Good! The ls process caused the pipe to stop.
```

## Projects that use markatzea

> A list of projects that use markatzea.

- https://github.com/bas080/markatzea
- https://github.com/bas080/patroon
- https://github.com/bas080/furver
- https://github.com/bas080/package.sh

Notice that it makes it easier to maintain and test documentation by making
usage examples runnable and thereby testable.

## License

[GPL-3.0][1]

[1]:./LICENSE
[2]:https://github.com/bas080/memplate
