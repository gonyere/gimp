# GIMP Coding Style

This document describes the preferred coding style for the GIMP source code.
It was originally inspired by [GNU's coding
style](https://www.gnu.org/prep/standards/standards.html#Writing-C) and
developed from there across the years.

Coding style is completely arbitrary as it is a a matter of consistency,
readability and maintenance, not taste. Therefore it doesn't matter what
you prefer (we all have some part of the rules we would like different,
and we can apply these to our personal projects), just follow these
rules so that we can work together efficiently.

This document will use examples at the very least to provide
authoritative and consistent answers to common questions regarding the
coding style and will also try to identify the allowed exceptions.

## Table of contents

[TOC]

## Dealing with the old code

__The new GIMP code should adhere to the style explained below.__ The existing
GIMP code largely follows these conventions, but there are some differences like
extra or missing whitespace, tabs instead of space, wrong formatting, etc.

__Our policy is to update the style of a code block or function if and only if
you happen to touch that code.__

Please don't waste your time and reviewers' time with merge requests or patches
with _only_ code style fixes unless you have push rights to the GIMP's
repository.

## Git usage
### Commit messages

Commit messages should follow the following rules:

- Always provide informative titles. No one-word commits saying nothing
  like "bug fix", so that we can at least search through the git history
  if needed. It can still be short messages for very simple fixes: for
  instance "Fix typo" or "Fix small memory leak" are informative of the
  type of fix.
- Prefix the title with the codebase section (i.e. the root folder
  usually) which was changed. For instance: `libgimpbase: fix memory
  leak` immediately tells us it was a memory leak fix in the
  `libgimpbase` library. If several sections are touched, list them with
  comma-separation.
- Alternatively, when the change is a response to a bug report, you may
  prefix with `Issue #123:` (where `#123` is the report ID) instead.
- If the changed code itself is not self-explanatory enough, you can add
  longer change description (2 lines after the title) to explain more.
  It is not mandatory, but it is never unwelcome because old code
  exploration to understand why things were done (possibly years before
  by people long gone) is a real thing we do all the time. So if it's
  not obvious, explain us.
- Explanations can also be made in the shape of links to a bug report
  (ours, or a third-party project tracker, or some manual), even though
  additional text explanations may still be useful (unfortunately URLs
  may change or disappear). If the link is to our own bug tracker,
  usually giving the ID is enough.
- Same [as for code](#line-width), wrap your commit message to
  reasonable line widths, such as 72 or 80 maximum so that other
  contributors don't have to scroll horizontally on narrow
  vizualisation. There may be exceptions, for instance when pasting some
  error messages which may end up confusing when wrapped. But other than
  this, wrap your text (most `git` client would have a feature to do it
  for you).
- If the title is too long because of the max-width settings, a common
  format is to break it with '…' and to continue the title 2 lines below.
  Then the description goes again 2 lines below.

Here is an example of a well formatted fix in the `plug-ins/` section:

```
plug-ins: fix file-gih.

We currently cannot call gimp_pdb_run_procedure() for procedures
containing arrays because this C-type doesn't contain the size
information (which is in a second parameter but since the rule is not
hard-coded, our API can't know this).

See issue #7369.
```

Here is another as a response to a bug report and a long title:

```
Issue #6695: Wrong tab after JPG export because of "Show preview"…

… feature.

Using the new gimp_display_present() function in file-jpeg to make sure
the original display is back to the top.
```

If you want to see more good examples, this `git` command will list
commits whose messages are generally well formatted:
`git log --author="Jehan\|mitch\|Jacob Boerema"`

### Linear git log

Nearly all our repositories (`gimp-web` being the exception) have a
fully linear history. The "merge commit" workflow is definitely good and
useful in some projects' workflow, especially with bigger projects with
many contributors and subdivided maintenance roles (where the main tree
is mostly about merging public trees of several submaintainers' public
trees, who themselves applied contributed commits by individuals).

This doesn't work well with GIMP's current workflow and our number of
contributors. This is even worse with the completely useless merge
commits created by hosting tools which completely misused (or even
misunderstood) the merge concept.

This is why our Gitlab projects are configured to only push commits
linearly. This means that when a contributed tree is behind, you must
first rebase it through the "Rebase" button in the merge request (which
requires contributors to check the "*Allow commits from members who can
merge to the target branch*" option) or by rebasing in the tree then
force-pushing when Gitlab is unable to merge.

When you push directly (for contributors with push rights), you are also
expected to never push a merge commit.

## C code
### Line width

The recommended line width for source files is _80 characters_ whenever possible.
Longer lines are usually an indication that you either need a function or a
pre-processor macro.

The point is to have clear code to read and not overly long lines. Don't break
code into ugly and choppy parts to blindly follow this rule.

Function definitions in the function forward declaration block don't have to
obey the 80 characters limit. The only general rule for headers is to align the
function definitions vertically in three columns.
See more information in [Headers sections](#Headers)

### Whitespaces
#### Indentation

Each new level is indented 2 or more spaces than the previous level:

```c
  if (condition)
    single_statement ();
```

Use only __space characters__ to achieve this. Code indented with tabs will not
be accepted into the GIMP codebase.

Even if two spaces for each indentation level allow deeper nesting, GIMP favors
self-documenting function names that can be quite long. For this reason, you
should avoid deeply nested code.

#### Vertical spaces (new lines)

Except for one single newline at the end of the file, other empty lines (at the
beginning and the end) of a file are not allowed.

On the other hand, empty lines in the middle of code are very encouraged
for well-ventilated code. For instance gathering and separating code by
logical parts making it easy to read and understand.

#### Horizontal spaces

Always put a space before an opening parenthesis but never after:

```c
  /* valid */
  if (condition)
    do_my_things ();

  /* valid */
  switch (condition)
    {
    }

  /* invalid */
  if(condition)
    do_my_things();

  /* invalid */
  if ( condition )
    do_my_things ( );
```

Do not eliminate whitespace and newlines just because something would
fit on 80 characters:

```c
  /* invalid */
  if (condition) foo (); else bar ();
```

#### Tab characters in strings

Use `\t` instead of literal tab inside the source code strings.

### Braces

#### If-else

Don't use curly braces for single statement blocks:

```c
  /* valid */
  if (condition)
    single_statement ();
  else
    another_single_statement (arg1);
```

In the case of multiple statements, put curly braces on another indentation level:

```c
  /* valid */
  if (condition)
    {
      statement_1 ();
      statement_2 ();
      statement_3 ();
    }

  /* invalid */
  if (condition) {
    statement_1 ();
    statement_2 ();
  }

  /* invalid */
  if (condition)
  {
    statement_1 ();
    statement_2 ();
  }
```

The "no block for single statements" rule has only three exceptions:

  ① _Both sides of the if-else statement_ must have curly braces when
     either side of this if-else statement has braces or when
     the single statement covers multiple lines, and it's followed
     by else or else if (e.g., for functions with many arguments).

```c
  /* valid */
  if (condition)
    {
      a_single_statement_with_many_arguments (some_lengthy_argument,
                                              another_lengthy_argument,
                                              and_another_one,
                                              plus_one);
    }
  else
    {
      another_single_statement (arg1, arg2);
    }
```

  ②  if the condition is composed of many lines:

```c
  /* valid */
  if (condition1                 ||
      (condition2 && condition3) ||
      condition4                 ||
      (condition5 && (condition6 || condition7)))
    {
      a_single_statement ();
    }
```

  ③  In the case of nested if's, the block should be placed on the outermost if:

```c
  /* valid */
  if (condition)
    {
      if (another_condition)
        single_statement ();
      else
        another_single_statement ();
    }

  /* invalid */
  if (condition)
    if (another_condition)
      single_statement ();
    else if (yet_another_condition)
      another_single_statement ();
```

#### Switch

A `switch()` should open a block on a new indentation level, and each case
should start on the same indentation level as the curly braces, with the
case block on a new indentation level:

```c
  /* valid */
  switch (condition)
    {
    case FOO:
      do_foo ();
      break;

    case BAR:
      do_bar ();
      break;
    }

  /* invalid */
  switch (condition) {
    case FOO: do_foo (); break;
    case BAR: do_bar (); break;
  }

  /* invalid */
  switch (condition)
    {
    case FOO: do_foo ();
      break;
    case BAR: do_bar ();
      break;
    }

  /* invalid */
  switch (condition)
    {
      case FOO:
      do_foo ();
      break;
      case BAR:
      do_bar ();
      break;
    }
```

It is preferable, though not mandatory, to separate the various cases with
a newline:

```c
  switch (condition)
    {
    case FOO:
      do_foo ();
      break;

    case BAR:
      do_bar ();
      break;

    default:
      do_default ();
      break;
    }
```

If a case block needs to declare new variables, the same rules as the inner
blocks (see above) apply; place the break statement outside of the inner block:

```c
  switch (condition)
    {
    case FOO:
      {
        int foo;

        foo = do_foo ();
      }
      break;

    ...
    }
```

Do not add `default:` case if your `switch ()` is supposed to handle _all cases_.

#### Random blocks

Using blocks to group code is discouraged and must not be used in newly
written code.

```c
int   retval    = 0;
gbool condition = retval >= 0;

statement_1 ();
statement_2 ();

  /* discouraged in newly written code */
  {
    int      var1 = 42;
    gboolean res  = FALSE;

    res = statement_3 (var1);
    retval = res ? -1 : 1;
  }
```

### Conditions

Do not check boolean values for equality:

```c
  /* valid */
  if (another_condition)
    do_bar ();

  /* invalid */
  if (condition == TRUE)
    do_foo ();
```

Even if C handles NULL equality like a boolean, be explicit:

```c
  /* valid */
  if (some_pointer == NULL)
    do_blah ();

  /* invalid */
  if (some_other_pointer)
    do_blurp ();
```

When conditions split over multiple lines, the logical operators should always
go at the end of the line. Align the same level boolean operators to show
explicitly which are on the same level and which are not:

```c
  /* valid */
  if (condition1  &&
      condition2  &&
      (condition3 || (condition4 && condition5)))
    {
      do_blah ();
    }

  /* invalid */
  if (condition1
      || condition2
      || condition3)
    {
      do_foo ();
    }
```

### Variables declaration and definition

Place each variable on a new line. The variable name must be right-aligned,
taking into account pointers:

```c
  /* valid */
  int         first  = 42;
  gboolean    second = TRUE;
  GimpObject *third  = NULL;
```

Blocks of variable declaration/initialization should align the variable names,
allowing quick skimming of the variable list.

### Functions

Function header has the return type on one line; the name starting in the first
column of the following line. Prototype each parameter and place each on a
new line.

In function names, each word must be lowercase and separated by an underscore.

In the function definition, place the return value on a separate line from the
function name:

```c
void
my_function (void)
{
}
```

The parameters list must be broken into a new line for each parameter, with the
parameter names right-aligned, taking into account pointers:

```c
void
my_function (some_type_t     some_param,
             another_type_t *a_pointer_param,
             final_type_t    final_param)
{
}
```

While curly braces for function definitions should rest on a new line they
should not add an indentation level:

```c
/* valid */
static void
my_function (int parameter)
{
  do_my_things ();
}
```

The alignment also holds when invoking a function:

```c
  align_function_arguments (first_argument,
                            second_argument,
                            third_argument);
```

If your function name is very long, it's always better to extract arguments into
separate variables to improve readability:

```c
  /* valid */
  first_a  = argument_the_first;
  second_a = argument_the_second;
  a_very_long_function_name_with_long_arguments (first_a, second_a);
```

Keep the function name and the arguments on the same line. Otherwise, it will
hurt readability.

```c
  /* invalid */
  a_very_long_function_name_with_long_arguments
    (argument_the_first, argument_the_second);
```

### Macros

Try to avoid private macros unless strictly necessary. Remember to `#undef`
them at the end of a block or a series of functions needing them.

Use inline functions instead of private macro definitions.
Do not use public macros unless they evaluate to a constant.

### Includes

GIMP source files should never include the global `gimp.h` header, but instead
include the individual headers that are needed.

Includes must be in the following order:

 0. `config.h` first;
 0. System and third-party headers;
 0. GIMP library headers (libgimp* headers);
 0. GIMP core/app headers that it needs including its own;

Sort alphabetically the includes within the blocks.


```c
/* valid */
#include "config.h"

#include <gegl.h>
#include <gtk/gtk.h>

#include "libgimpbase/gimpbase.h"
#include "libgimpcolor/gimpcolor.h"

#include "core/gimp.h"
#include "core/gimpcontainer.h"

#include "gimpcolorpanel.h"
#include "gimpcontainerentry.h"
```

### Structures

When declaring a structure type use newlines to separate logical sections:

```c
  /* preferred for new code*/
  typedef struct
  {
    gint  n_pages;
    gint *pages;

    gbool read_only;
  } Pages;
```

### Memory management

To dynamically allocate data on the heap, use `g_new()`. To allocate memory for
multiple small data structures, `g_slice_new()`.

When possible, all public structure types should be returned fully initialized,
either explicitly for each member or by using g_new0() or g_slice_new0().

As a general programming rule, it is better to allocate and free data on the
same level. It is much easier to review code because you know that when you
allocate something there, then you also free it there.

```c
GeglBuffer *buffer;
void       *p;

*buffer = gegl_buffer_new (some, args);
*p      = g_new (something, n);

/* do stuff */

g_object_unref (buffer);
g_free (p);
```

When a transfer of ownership is unavoidable make it clear in the function
documentation.

### Comments
#### In-code explanation

The only allowed style is C-style comments `/* */`. In particular C++
comments `//` are strictly forbidden from our source.

We are not asking contributors to over-comment their code, yet we highly
value quality comments to explain complicated algorithms or non-obvious
code. Just ask yourself this: what if someone sees my code 5 years later
(another contributor or even your future self)…

- will one easily understand what you meant to do?
- In particular: if it needs to be removed later, won't one be scared to
  delete now-useless code by fear of unexpected side-effects?
- Or oppositely: won't someone delete the code by mistake because it
  looks useless while it was actually dealing with a very particular
  (yet absolutely non-obvious) issue?

Adding links which explain well a problem or the reason for some
non-obvious code is also permitted. For instance a link to a bug report
(ours or some other projects') can sometimes be a good complement to a
comment.
Nevertheless it should not be overdone and in particular not for links
likely to disappear (personal blog posts, forums, corporate websites
which often revamp their design, breaking URLs, etc.).

#### Public API Documentation

All public APIs (i.e. any function exported in a header inside
`libgimp*/` folders) **MUST** have proper gtk-doc comments. For
functions, these should be placed in the source file directly above.

These annotations are also relevant for [GObject
Introspection](https://gi.readthedocs.io/en/latest/annotations/giannotations.html)
to generate bindings for other languages.

```c
/* valid */
/**
 * gimp_object_set_name:
 * @object: a #GimpObject
 * @name: the @object's new name (transfer none)
 *
 * Sets the @object's name. Takes care of freeing the old name and
 * emitting the ::name_changed signal if the old and new name differ.
 **/
void
gimp_object_set_name (GimpObject  *object,
                      const gchar *name)
{

  ...

}
```

Doc comments for macros, function types, class structs, etc., should be placed
next to the definitions, typically in headers.

#### Non-public API documentation

Project-only code (for instance any code from the `app/` folder) can be
less documented. For instance when a function has obvious naming, not
explaining it is perfectly acceptable.

Nevertheless adding documentation even for these private APIs is
welcome, especially when the usage is not as obvious as it looks, or
to make sure to advertize the proper memory handling (does it allocate
new memory? Which function to free it with? Or shouldn't the returned
memory be freed?), avoiding silly bugs and not wasting developer times
(when we have to look at the implementation to verify each time we use a
function).

In such a case, using gtk-docs syntax is still a nice idea as we will
understand it directly (even though we won't generate any docs from it).

### Public API
#### No variables

Avoid exporting variables as public API since this is cumbersome on some
platforms. It is always preferable to add getters and setters instead.

#### Def files for Windows

List all public functions alphabetically in the corresponding `.def` file.

 - `app/gimpcore.def`
 - `libgimp/gimp.def`
 - `libgimpbase/gimpbase.def`
 - etc

## Helping tools
### Git

We recommend enabling the default git pre-commit hook that detects trailing
whitespace and non-ASCII filenames for you and helps you to avoid corrupting
GIMP's tree with it.

In the terminal, navigate into your GIMP source code folder. Then do that as
follows (one command at a time):

```shell
  cp -v .git/hooks/pre-commit.sample .git/hooks/pre-commit
  chmod a+x .git/hooks/pre-commit
```

If any command above fails, visit your `.git/hooks/` folder and check for the
existence of `pre-commit.sample` or `pre-commit` files.

You might also find the `git-stripspace` utility helpful, which acts as a filter
to remove trailing whitespace as well as initial, final, and duplicate blank
lines.

### Code editor / Integrated Development Environment (IDE)

GIMP's codebase is not tied to a specific editor or IDE. The whole build
can be performed from anywhere, and we don't care what you write your
code with (as long as it follows syntax rules from this document).

Several configuration files were contributed across the years to
configure your favorite software to follow our coding style.
You are very welcome to use them (or improve them and contribute the
change when they are not perfect):

- [.dir-locals.el](.dir-locals.el) for Emacs;
- [.kateconfig](.kateconfig) for Kate;
- [c.vim](devel-docs/c.vim) for Vim (check the top comments to see how
  to enable it automatically when opening a file in the GIMP tree).

*Note: the Kate and Emacs config file should work out-of-the-box, but
the Vim one needs to be enabled explicitly because it is too powerful,
hence is basically [unsafe](https://github.com/vim/vim/issues/1015).*

If you use another software to write code, you are welcome to contribute
coding style files following our rules.

### Code Formatter

We don't use a code formatter to re-format code because it is unable to
handle special cases well as far as we know.

Nevertheless we would be interested to use these to perform at least
some soft verification of contributed patches. A CI-performed check
could help new contributors to fix their basic newcomer coding style
mistakes and free up reviewing contributors' time.

The tool Clang-format has been mentionned as relevant, though nobody has
written syntax files for this tool yet (contribution welcome for this
too). See also #950.
