# Go Meta Linter
[![Build Status](https://travis-ci.org/alecthomas/gometalinter.png)](https://travis-ci.org/alecthomas/gometalinter) [![Gitter chat](https://badges.gitter.im/alecthomas.png)](https://gitter.im/alecthomas/Lobby)

The number of tools for statically checking Go source for errors and warnings
is impressive.

This is a tool that concurrently runs a whole bunch of those linters and
normalises their output to a standard format:

    <file>:<line>:[<column>]: <message> (<linter>)

eg.

    stutter.go:9::warning: unused global variable unusedGlobal (varcheck)
    stutter.go:12:6:warning: exported type MyStruct should have comment or be unexported (golint)

It is intended for use with editor/IDE integration.

## Editor integration

- [SublimeLinter plugin](https://github.com/alecthomas/SublimeLinter-contrib-gometalinter).
- [vim-go](https://github.com/fatih/vim-go) with the `:GoMetaLinter` command.
- [syntastic (vim)](https://github.com/scrooloose/syntastic/wiki/Go:---gometalinter) `let g:syntastic_go_checkers = ['gometalinter']`
- [Atom go-plus package](https://atom.io/packages/go-plus).
- [Emacs Flycheck checker](https://github.com/favadi/flycheck-gometalinter).
- [Go for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=lukehoban.Go).

## Supported linters

- [go vet](https://golang.org/cmd/vet/) - Reports potential errors that otherwise compile.
- [go vet --shadow](https://golang.org/cmd/vet/#hdr-Shadowed_variables) - Reports variables that may have been unintentionally shadowed.
- [gotype](https://golang.org/x/tools/cmd/gotype) - Syntactic and semantic analysis similar to the Go compiler.
- [deadcode](https://github.com/tsenart/deadcode) - Finds unused code.
- [gocyclo](https://github.com/alecthomas/gocyclo) - Computes the cyclomatic complexity of functions.
- [golint](https://github.com/golang/lint) - Google's (mostly stylistic) linter.
- [varcheck](https://github.com/opennota/check) - Find unused global variables and constants.
- [structcheck](https://github.com/opennota/check) - Find unused struct fields.
- [aligncheck](https://github.com/opennota/check) - Warn about un-optimally aligned structures.
- [errcheck](https://github.com/kisielk/errcheck) - Check that error return values are used.
- [dupl](https://github.com/mibk/dupl) - Reports potentially duplicated code.
- [ineffassign](https://github.com/gordonklaus/ineffassign/blob/master/list) - Detect when assignments to *existing* variables are not used.
- [interfacer](https://github.com/mvdan/interfacer) - Suggest narrower interfaces that can be used.
- [unconvert](https://github.com/mdempsky/unconvert) - Detect redundant type conversions.
- [goconst](https://github.com/jgautheron/goconst) - Finds repeated strings that could be replaced by a constant.
- [gosimple](https://github.com/dominikh/go-simple) - Report simplifications in code.
- [staticcheck](https://github.com/dominikh/go-staticcheck) - Statically detect bugs, both obvious and subtle ones.
- [gas](https://github.com/GoASTScanner/gas) - Inspects source code for security problems by scanning the Go AST.

Disabled by default (enable with `--enable=<linter>`):

- [testify](https://github.com/stretchr/testify) - Show location of failed testify assertions.
- [test](http://golang.org/pkg/testing/) - Show location of test failures from the stdlib testing module.
- [gofmt -s](https://golang.org/cmd/gofmt/) - Checks if the code is properly formatted and could not be further simplified.
- [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - Checks missing or unreferenced package imports.
- [lll](https://github.com/walle/lll) - Report long lines (see `--line-length=N`).
- [misspell](https://github.com/client9/misspell) - Finds commonly misspelled English words.
- [unused](https://github.com/dominikh/go-unused) - Find unused variables.

Additional linters can be added through the command line with `--linter=NAME:COMMAND:PATTERN` (see [below](#details)).

### Configuration file

gometalinter now supports a JSON configuration file which can be loaded via
`--config=<file>`. The format of this file is determined by the Config struct
in `config.go`.

The configuration file mostly corresponds to command-line flags, with the following exceptions:

- Linters defined in the configuration file will overlay existing definitions, not replace them.
- "Enable" defines the exact set of linters that will be enabled.

Here is an example configuration file:

```json
{
  "DisableAll": true,
  "Enable": ["deadcode", "unconvert"]
}
```

## Installing

There are two options for installing gometalinter.

1. Install a stable version, eg. `go get -u gopkg.in/alecthomas/gometalinter.v1`.
   I will generally only tag a new stable version when it has passed the Travis
  regression tests. The downside is that the binary will be called `gometalinter.v1`.
2. Install from HEAD with: `go get -u github.com/alecthomas/gometalinter`.
   This has the downside that changes to gometalinter may break.

## Quickstart

Install gometalinter (see above).

Install all known linters:

```
$ gometalinter --install
Installing:
  structcheck
  aligncheck
  deadcode
  gocyclo
  ineffassign
  dupl
  golint
  gotype
  goimports
  errcheck
  varcheck
  interfacer
  goconst
  gosimple
  staticcheck
  unused
  misspell
  lll
  gas
```

Run it:

```
$ cd example
$ gometalinter ./...
stutter.go:13::warning: unused struct field MyStruct.Unused (structcheck)
stutter.go:9::warning: unused global variable unusedGlobal (varcheck)
stutter.go:12:6:warning: exported type MyStruct should have comment or be unexported (golint)
stutter.go:16:6:warning: exported type PublicUndocumented should have comment or be unexported (golint)
stutter.go:8:1:warning: unusedGlobal is unused (deadcode)
stutter.go:12:1:warning: MyStruct is unused (deadcode)
stutter.go:16:1:warning: PublicUndocumented is unused (deadcode)
stutter.go:20:1:warning: duplicateDefer is unused (deadcode)
stutter.go:21:15:warning: error return value not checked (defer a.Close()) (errcheck)
stutter.go:22:15:warning: error return value not checked (defer a.Close()) (errcheck)
stutter.go:27:6:warning: error return value not checked (doit()           // test for errcheck) (errcheck)
stutter.go:29::error: unreachable code (vet)
stutter.go:26::error: missing argument for Printf("%d"): format reads arg 1, have only 0 args (vet)
```


Gometalinter also supports the commonly seen `<path>/...` recursive path
format. Note that this can be *very* slow, and you may need to increase the linter `--deadline` to allow linters to complete.

## FAQ

### Exit status

gometalinter sets two bits of the exit status to indicate different issues:

| Bit | Meaning
|-----|----------
| 0   | A linter generated an issue.
| 1   | An underlying error occurred; eg. a linter failed to execute. In this situation a warning will also be displayed.

eg. linter only = 1, underlying only = 2, linter + underlying = 3

### What's the best way to use `gometalinter` in CI?

There are two main problems running in a CI:

1. <s>Linters break, causing `gometalinter --install --update` to error</s> (this is no longer an issue as all linters are vendored).
2. `gometalinter` adds a new linter.

I have solved 1 by vendoring the linters.

For 2, the best option is to disable all linters, then explicitly enable the
ones you want:

    gometalinter --disable-all --enable=errcheck --enable=vet --enable=vetshadow ...

### How do I make `gometalinter` work with Go 1.5 vendoring?

`gometalinter` has a `--vendor` flag that just sets `GO15VENDOREXPERIMENT=1`, however the
underlying tools must support it. Ensure that all of the linters are up to date and built with Go 1.5
(`gometalinter --install --force`) then run `gometalinter --vendor .`. That should be it.

### Why does `gometalinter --install` install a fork of gocyclo?

I forked `gocyclo` because the upstream behaviour is to recursively check all
subdirectories even when just a single directory is specified. This made it
unusably slow when vendoring. The recursive behaviour can be achieved with
gometalinter by explicitly specifying `<path>/...`. There is a
[pull request](https://github.com/fzipp/gocyclo/pull/1) open.

### Gometalinter is not working

That's more of a statement than a question, but okay.

Sometimes gometalinter will not report issues that you think it should. There
are three things to try in that case:

#### 1. Update to the latest build of gometalinter and all linters

    go get -u github.com/alecthomas/gometalinter
    gometalinter --install

If you're lucky, this will fix the problem.

#### 2. Analyse the debug output

If that doesn't help, the problem may be elsewhere (in no particular order):

1. Upstream linter has changed its output or semantics.
2. gometalinter is not invoking the tool correctly.
3. gometalinter regular expression matches are not correct for a linter.
4. Linter is exceeding the deadline.

To find out what's going on run in debug mode:

    gometalinter --debug

This will show all output from the linters and should indicate why it is
failing.

#### 3. Report an issue.

Failing all else, if the problem looks like a bug please file an issue and
include the output of `gometalinter --debug`.

## Details

Additional linters can be configured via the command line:

```
$ gometalinter --linter='vet:go tool vet -printfuncs=Infof,Debugf,Warningf,Errorf {path}:PATH:LINE:MESSAGE' .
stutter.go:21:15:warning: error return value not checked (defer a.Close()) (errcheck)
stutter.go:22:15:warning: error return value not checked (defer a.Close()) (errcheck)
stutter.go:27:6:warning: error return value not checked (doit()           // test for errcheck) (errcheck)
stutter.go:9::warning: unused global variable unusedGlobal (varcheck)
stutter.go:13::warning: unused struct field MyStruct.Unused (structcheck)
stutter.go:12:6:warning: exported type MyStruct should have comment or be unexported (golint)
stutter.go:16:6:warning: exported type PublicUndocumented should have comment or be unexported (deadcode)
```

## Checkstyle XML format

`gometalinter` supports [checkstyle](http://checkstyle.sourceforge.net/)
compatible XML output format. It is triggered with `--checkstyle` flag:

	gometalinter --checkstyle

Checkstyle format can be used to integrate gometalinter with Jenkins CI with the
help of [Checkstyle Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin).
