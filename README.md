# investigate cue issue #2567

[Cue fmt destroys working code #2567](https://github.com/cue-lang/cue/issues/2567)

## related issues

- [#1629 improve strategy for comment output](https://github.com/cue-lang/cue/issues/1629) - overview of 6 issues

### misplaced comments

- [#720 # Cue fmt drags comments into import block](https://github.com/cue-lang/cue/issues/720) - can reproduce
- [#1447 # cmd/fmt: single import and comment combined](https://github.com/cue-lang/cue/issues/1447) - can reproduce
- [#1478 # cmd/fmt: comma appended to comment instead of list element](https://github.com/cue-lang/cue/issues/1478) - cannot reproduce - closed
- [#1553 # Every time format, a comma is added.](https://github.com/cue-lang/cue/issues/1553) - fixed - closed
- [#2274 # cmd/fmt: fmt emits invalid CUE when commenting struct references in lists](https://github.com/cue-lang/cue/issues/2274) - can reproduce
- [#2423 # cmd/fmt: invalid CUE emitted when all fields are commented inside struct](https://github.com/cue-lang/cue/issues/2423) - can reproduce
- [#2567 # Cue fmt destroys working code](https://github.com/cue-lang/cue/issues/2567) - can reproduce

### tabs and indent

- [#722 # cue fmt indents top-level comments ](https://github.com/cue-lang/cue/issues/722)
- [#1006 # cue fix tries to align comments inside braces with those outside, causing weird field indentation](https://github.com/cue-lang/cue/issues/1006)
- [#1040 # cmd/cue: fmt converts tabs to spaces in comments ](https://github.com/cue-lang/cue/issues/1040) - cueckoo closed this as completed in dac4917 on May 30, 2022
- [#1629 # improve strategy for comment output](https://github.com/cue-lang/cue/issues/1629) - Most of these are to do with comment formatting, but there's a deeper issue here: what should happen to comments when we apply unification to CUE values?

### commented error message

- [#2646 # cmd/cue: def sometimes inserts CUE comments for errors](https://github.com/cue-lang/cue/issues/2646)

## test cases

### 1447 - single import and comment combined

#### 1447-1.cue ok

`cue fmt`inserts a newline before the comment.

```
cue % cat testdata/1447-1.cue                                            [issues-fmt-comments L|…1]
import "strings"
// comment
s: string
u: strings.ToUpper(s)
```

```
cue % cat testdata/1447.cue | go run . fmt -                             [issues-fmt-comments L|…1]
file: -
DebugStr: import "strings", <[d0// comment] s: string>, u: strings.ToUpper(s)
import "strings"

// comment
s: string
u: strings.ToUpper(s)
```

#### 1447-2.cue bad (fmt moves comment into the import block) but the result passes eval

```
cue % cat testdata/1447-2.cue                                            [issues-fmt-comments L|…1]
import "strings"
// comment

s: string
u: strings.ToUpper(s)
```

```
cue % cat testdata/1447-2.cue | go run . fmt -                           [issues-fmt-comments L|…1]
file: -
DebugStr: import <[2// comment] "strings">, s: string, u: strings.ToUpper(s)
import ( "strings"
    // comment
)

s: string
u: strings.ToUpper(s)
```

#### 1447-3.cue bad (fmt moves comments into the import block) but the result passes eval

```
cue % cat testdata/1447-3.cue | go.r fmt -                                                              [issues-fmt-comments L|…1]
file: -
DebugStr: import <[2// comment a] [2// comment b] "strings">, s: string, u: strings.ToUpper(s)
import ( "strings"
    // comment a

    // comment b
)

s: string
u: strings.ToUpper(s)
```

```
cue % cat testdata/1447-3.cue | go.r fmt -                                                              [issues-fmt-comments L|…1]
file: -
DebugStr: import <[2// comment a] [2// comment b] "strings">, s: string, u: strings.ToUpper(s)
import ( "strings"
    // comment a

    // comment b
)

s: string
u: strings.ToUpper(s)
```

### 1478 # cmd/fmt: comma appended to comment instead of list element - cannot reproduce - closed

Probabbly fixed by the #1553 fix, should check.

### 1553 # Every time format, a comma is added. - fixed and closed

Fixed in [commit](https://github.com/cue-lang/cue/commit/d16c7998541098bb304910add474bfb76308db96)

@satotake authored and @mpvl committed on Jul 22, 2022

```
cue/format: fix `StructLit` trailing comma bug

`cue fmt` unexpectedly adds trailing comma not before but after comment
if `StructLit` satisfies the following conditions:

* is an element of list
* has no element
* is before `Comment`

This CL:

* fix this issue
* add a new test data
* fix an existing test data

Close #1553
Signed-off-by: satotake <doublequotation@gmail.com>
Change-Id: Icd181b6c8bee651ff98cf17ca706d453f960b7b4
Reviewed-on: https://review.gerrithub.io/c/cue-lang/cue/+/539255
Reviewed-by: Marcel van Lohuizen <mpvl@gmail.com>
Unity-Result: CUEcueckoo <cueckoo@cuelang.org>
TryBot-Result: CUEcueckoo <cueckoo@cuelang.org>

```

This probably also fixed #1478, not reproducible in v0.6.0.

### 2274 # cmd/fmt: fmt emits invalid CUE when commenting struct references in lists

input:

```
#Number: two: 2

aList: [
  1,
  #Number.two,
  // comment

  3,
]
```

mangled by cue fmt:

```

#Number: two: 2

aList: [
1,
#Number.two
// comment,

    3,

]
```

### 2423 # cmd/fmt: invalid CUE emitted when all fields are commented inside struct

### 2567 # Cue fmt destroys working code

#### 2567-3.cue

```

testdata % cat 2567-3.cue
[
if true // inline comment
{}
]

testdata % ccf 2567-3.cue
file: -, msg: decoded, DebugStr:[<[l2// inline comment] if true {}>]
[
if true {} // inline comment,
]

```

Above is marked `l2` which makes sense wrt the input file, but it seems that `{}` was moved to the same line afterwards.

### compare 2274 case variations

What is the meaninng of `l2` and `l3`?

```

cue-issues-fmt-comments % cat testdata/2274-01n.cue | cue fmt -
file: -, msg: before fix, DebugStr:#N: {two: 2}, l: [1, <[l2// comment] 2>, 3]
file: -, msg: decoded, DebugStr:#N: {two: 2}, l: [1, <[l2// comment] 2>, 3]
#N: two: 2

l: [
1,
2, // comment
3,
]

cue-issues-fmt-comments % cat testdata/2274-01s.cue | cue fmt -
file: -, msg: before fix, DebugStr:#N: {two: 2}, l: [1, <[l3// comment] #N.two>, 3]
file: -, msg: decoded, DebugStr:#N: {two: 2}, l: [1, <[l3// comment] #N.two>, 3]
#N: two: 2

l: [
1,
#N.two // comment,
3,
]

```

Should look into DebugStr to see what `l2` and `l3` mean, in terms of the AST.

Should look into the NewDecoder to see how the AST is created.

```

./internal/astinternal/debugstr.go:195: str += "d"
./internal/astinternal/debugstr.go:198: str += "l"

```

```

    case *ast.CommentGroup:
        str := "["
        if v.Doc {
            str += "d"
        }
        if v.Line {
            str += "l"
        }
        str += strconv.Itoa(int(v.Position))
        var a = []string{}
        for _, c := range v.List {
            a = append(a, c.Text)
        }
        return str + strings.Join(a, " ") + "] "

```

```

// A CommentGroup represents a sequence of comments
// with no other tokens and no empty lines between.
type CommentGroup struct {
// TODO: remove and use the token position of the first comment.
Doc bool
Line bool // true if it is on the same line as the node's end pos.

    // Position indicates where a comment should be attached if a node has
    // multiple tokens. 0 means before the first token, 1 means before the
    // second, etc. For instance, for a field, the positions are:
    //    <0> Label <1> ":" <2> Expr <3> "," <4>
    Position int8
    List     []*Comment // len(List) > 0

    decl

}

```

```

[1, <[l2// comment] 2>, 3]
// looks like
// <0> #N <1> . <2> two <3> // comment <4> , <5>
// but it should be (I guess)
// #N.two, // comment
// <0> #N <1> . <2> two <3> , <4> // comment <5>
// DebugStr:#N: {two: 2}, l: [1, <[l4// comment] #N.two>, 3]

```

```

l: [
1,
#N.two // comment,
3,
]

```

Should look into the parser

```

cue % grep.go '\.Line =' [issues-fmt-comments L|✚2…1]
./cue/token/position_test.go:217: want.Line = line - l1 + 100
./cue/token/position_test.go:221: want.Line = line - l2 + 3
./cue/parser/parser.go:367: comment.Line = true
...

```

## testing and debugging notes

### meaning of position codes in DebugStr

```

DebugStr: import "strings", <[d0// comment] s: string>, u: strings.ToUpper(s)
DebugStr: import <[2// comment] "strings">, s: string, u: strings.ToUpper(s)
DebugStr:#N: {two: 2}, l: [1, <[l3// comment] #N.two>, 3]

```

`l` means Line, `d` means Doc, `0` means Position 0, `2` means Position 2, `3` means Position 3.

What does the absence of `d` and `l` mean?

It looks like:
`l` means a trailing comment on a code line
`d` means a doc comment, i.e. a full comment line (a line starting with `//`) and immediately followed by a code line
no letter means a full comment line followed by a blank line

```

## use ast tree walk functions for diagnostics?

```

cue % grep.go 'func walk\(' [issues-fmt-comments L|…1]
./cue/ast/astutil/walk.go:52:func walk(v visitor, node ast.Node) { // d96ad3d Marcel van Lohuizen 2018-12-11 - cue/parser: add package
./cue/ast/walk.go:58:func walk(v visitor, node Node) { // da38611 Marcel van Lohuizen - cue/ast: add package

```

Used in ./internal/mod/modresolve/resolve.go and ./internal/mod/modresolve/sanitize.go

```

// walk traverses an AST in depth-first order: It starts by calling
// v.Visit(node); node must not be nil. If the visitor w returned by
// v.Visit(node) is not nil, walk is invoked recursively with visitor
// w for each of the non-nil children of node, followed by a call of
// w.Visit(nil).
func walk(v visitor, node ast.Node)

```

Unused

```

// walk traverses an AST in depth-first order: It starts by calling
// v.Visit(node); node must not be nil. If the visitor w returned by
// v.Visit(node) is not nil, walk is invoked recursively with visitor
// w for each of the non-nil children of node, followed by a call of
// w.Visit(nil).
func walk(v visitor, node Node)

```

Both funcs are very similar, probably cloned and modified slightly.

```

cue % grep.go 'func Walk' [issues-fmt-comments L|…1]
./cue/ast/walk.go:27:func Walk(node Node, before func(Node) bool, after func(Node)) {

```

```

// Walk traverses an AST in depth-first order: It starts by calling f(node);
// node must not be nil. If before returns true, Walk invokes f recursively for
// each of the non-nil children of node, followed by a call of after. Both
// functions may be nil. If before is nil, it is assumed to always return true.
func Walk(node Node, before func(Node) bool, after func(Node)) {
walk(&inspector{before: before, after: after}, node)
}

```

## notes and observations

###

[#722](https://github.com/cue-lang/cue/issues/722)
```

@rogpepe Unrelated but seen here: documenting the `cue export` <ins>handling of comments</ins>

Your testscript of Feb 25, 2022, shows that, given the option `--out cue`, `cue export` reproduces the comments found in the input (badly indented, in this case) and thus behaves just like `cue fmt`.

I observed that `cue def c.cue` also has the same `fmt`-like behavior.

This does not appear to be documented anywhere. I think it should be documented in `cue export -h` and `cue def -h`, as a minimum.

### patch to add debug prints

```
diff --git a/cmd/cue/cmd/fmt.go b/cmd/cue/cmd/fmt.go
index 85333afb..8572be22 100644
--- a/cmd/cue/cmd/fmt.go
+++ b/cmd/cue/cmd/fmt.go
@@ -15,6 +15,8 @@
 package cmd

 import (
+       "os"
+
        "github.com/spf13/cobra"

        "cuelang.org/go/cue/ast"
@@ -23,6 +25,7 @@ import (
        "cuelang.org/go/cue/format"
        "cuelang.org/go/cue/load"
        "cuelang.org/go/cue/token"
+       "cuelang.org/go/internal/astinternal"
        "cuelang.org/go/internal/encoding"
        "cuelang.org/go/tools/fix"
 )
@@ -73,10 +76,14 @@ func newFmtCmd(c *Command) *cobra.Command {
                                                f := d.File()

                                                if file.Encoding == build.CUE {
+                                                       // astinternal.DebugStrToStdErr("before fix", f)
                                                        f = fix.File(f)
                                                }

                                                files = append(files, f)
+                                               if os.Getenv("CUEDO_FMT_DEBUGSTR") != "" {
+                                                       astinternal.DebugStrToStdErr("decoded ast", f)
+                                               }
                                        }
                                        // Do not defer this Close call, as we are looping over builds,
                                        // and otherwise we would hold all of their files open at once.
diff --git a/internal/astinternal/debugstr.go b/internal/astinternal/debugstr.go
index 81cba966..4f5f63be 100644
--- a/internal/astinternal/debugstr.go
+++ b/internal/astinternal/debugstr.go
@@ -16,6 +16,8 @@ package astinternal

 import (
        "fmt"
+       "os"
+       "path/filepath"
        "strconv"
        "strings"

@@ -283,3 +285,11 @@ func DebugStr(x interface{}) (out string) {
 }

 const sep = ", "
+
+// TEMPORARY
+// DebugStrToStdErr prints DebugStr for `f`
+func DebugStrToStdErr(msg string, f *ast.File) {
+       fmt.Fprintf(os.Stderr, "file: %s %s DebugStr:%s\n", filepath.Base(f.Filename), msg, DebugStr(f))
+}
+
+// END TEMPORARY
diff --git a/internal/encoding/encoding.go b/internal/encoding/encoding.go
index 53020fac..b015adc5 100644
--- a/internal/encoding/encoding.go
+++ b/internal/encoding/encoding.go
@@ -232,7 +232,11 @@ func NewDecoder(f *build.File, cfg *Config) *Decoder {
        switch f.Encoding {
        case build.CUE:
                if cfg.ParseFile == nil {
-                       i.file, i.err = parser.ParseFile(path, r, parser.ParseComments)
+                       if os.Getenv("CUEDO_PARSER_TRACE") != "" {
+                               i.file, i.err = parser.ParseFile(path, r, parser.ParseComments, parser.Trace) // TEMPORARY
+                       } else {
+                               i.file, i.err = parser.ParseFile(path, r, parser.ParseComments)
+                       }
                } else {
                        i.file, i.err = cfg.ParseFile(path, r)
                }
cue %                                                                                                 [issues-fmt-comments L|…1]
```

#### To create the patch in the root of the cue repo clone:

```
cue % pwd
/Users/rudifarkas/GitHub/golang/src/cue
cue % git lg 2

- 64a14f90(HEAD -> issues-fmt-comments)(2023-11-18 18:14:43 +0100)(Rudi Farkas)cmd/cue/cmd/fmt.go - instrument for debugging \_
- 7a4ea866 (tag: base)(2023-11-10 17:26:18 +0000)(Daniel Martí)CONTRIBUTING: fix repo reference rendering

```

... then run commands:
`

```

git diff base HEAD > cuedo.patch

```

or

```

git diff 7a4ea866 64a14f90 > cuedo.patch

```

#### To apply the patch on a different branch or repo clone, run in the root of the repo (`$GOPATH/src`):

```

git apply cuedo.patch

```

... and build and install the modified cue, in the `$GOPATH/src/cue/cmd/cue`:

```

cue % go install .

```

This will install the modified cue as `$GOPATH/bin/cue`.

Make sure that this is found in your `$PATH` before the system `cue` binary.

```

```

```

```

```

```

```

```

```

```

```

```
