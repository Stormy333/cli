---
section: cli-commands
title: npx
description: Run a command from a local or remote npm package
---

# npx(1)

## Run a command from a local or remote npm package

### Synopsis

```bash
npm exec -- <pkg>[@<version>] [args...]
npm exec -p <pkg>[@<version>] -- <cmd> [args...]
npm exec -c '<cmd> [args...]'
npm exec -p foo -c '<cmd> [args...]'

npx <pkg>[@<specifier>] [args...]
npx -p <pkg>[@<specifier>] <cmd> [args...]
npx -c '<cmd> [args...]'
npx -p <pkg>[@<specifier>] -c '<cmd> [args...]'

alias: npm x, npx

-p <pkg> --package=<pkg> (may be specified multiple times)
-c <cmd> --call=<cmd> (may not be mixed with positional arguments)
```

### Description

This command allows you to run an arbitrary command from an npm package
(either one installed locally, or fetched remotely), in a similar context
as running it via `npm run`.

Whatever packages are specified by the `--package` or `-p` option will be
provided in the `PATH` of the executed command, along with any locally
installed package executables.  The `--package` or `-p` option may be
specified multiple times, to execute the supplied command in an environment
where all specified packages are available.

If any requested packages are not present in the local project
dependencies, then they are installed to a folder in the npm cache, which
is added to the `PATH` environment variable in the executed process.
Package names provided without a specifier will be matched with whatever
version exists in the local project.  Package names with a specifier will
only be considered a match if they have the exact same name and version as
the local dependency.

If no `-c` or `--call` option is provided, then the positional arguments
are used to generate the command string.  If no `-p` or `--package` options
are provided, then npm will attempt to determine the executable name from
the package specifier provided as the first positional argument according
to the following heuristic:

- If the package has a single entry in its `bin` field in `package.json`,
  then that command will be used.
- If the package has multiple `bin` entries, and one of them matches the
  unscoped portion of the `name` field, then that command will be used.
- If this does not result in exactly one option (either because there are
  no bin entries, or none of them match the `name` of the package), then
  `npm exec` exits with an error.

To run a binary _other than_ the named binary, specify one or more
`--package` options, which will prevent npm from inferring the package from
the first command argument.

### `npx` vs `npm exec`

When run via the `npx` binary, all flags and options *must* be set prior to
any positional arguments.  When run via `npm exec`, a double-hyphen `--`
flag can be used to suppress npm's parsing of switches and options that
should be sent to the executed command.

For example:

```
$ npx foo@latest bar --package=@npmcli/foo
```

In this case, npm will resolve the `foo` package name, and run the
following command:

```
$ foo bar --package=@npmcli/foo
```

Since the `--package` option comes _after_ the positional arguments, it is
treated as an argument to the executed command.

In contrast, due to npm's argument parsing logic, running this command is
different:

```
$ npm exec foo@latest bar --package=@npmcli/foo
```

In this case, npm will parse the `--package` option first, resolving the
`@npmcli/foo` package.  Then, it will execute the following command in that
context:

```
$ foo@latest bar
```

The double-hyphen character is recommended to explicitly tell npm to stop
parsing command line options and switches.  The following command would
thus be equivalent to the `npx` command above:

```
$ npm exec -- foo@latest bar --package=@npmcli/foo
```

### Examples

Run the version of `tap` in the local dependencies, with the provided
arguments:

```
$ npm exec -- tap --bail test/foo.js
$ npx tap --bail test/foo.js
```

Run a command _other than_ the command whose name matches the package name
by specifying a `--package` option:

```
$ npm exec --package=foo -- bar --bar-argument
# ~ or ~
$ npx --package=foo bar --bar-argument
```

Run an arbitrary shell script, in the context of the current project:

```
$ npm x -c 'eslint && say "hooray, lint passed"'
$ npx -c 'eslint && say "hooray, lint passed"'
```

### See Also

* [npm run-script](/cli-commands/run-script)
* [npm scripts](/using-npm/scripts)
* [npm test](/cli-commands/test)
* [npm start](/cli-commands/start)
* [npm restart](/cli-commands/restart)
* [npm stop](/cli-commands/stop)
* [npm config](/cli-commands/config)