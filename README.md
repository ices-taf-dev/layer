# Layer Mechanics

## R-exts (R Core Team 2021, 4.1.2)

**Depends** - packages that will be attached before the current package when
  `library` or `require` is called.

**Imports** - packages whose namespaces are imported from (as specified in the
  NAMESPACE file) but which do not need to be attached. Ideally this field will
  include all the standard packages that are used. Packages declared in the
  'Depends' field should not also be in the 'Imports' field.

**Suggests** - packages that are not necessarily needed but used in examples,
  tests, or vignettes. Packages that are useful but not necessary.

**Enhances** - packages that are enhanced by the package at hand, e.g. by
  providing methods for classes from these packages, or ways to handle objects
  from these packages. There are several packages enhancing the 'chron' package,
  because they can handle datetime objects from 'chron' functions.

'Depends' should nowadays be used rarely, only for packages which are intended
to be put on the search path to make their facilities available to the end users
(and not to the package itself). For example, it makes sense that a user of
package 'latticeExtra' would want the functions of package 'lattice' made
available.

Almost always packages mentioned in 'Depends' should also be imported from in
the NAMESPACE file. This ensures that any needed parts of the packages are
available when some other package imports the current package.

The 'Imports' field should not contain packages which are not imported from (via
the NAMESPACE file), as all the packages listed in that field need to be
installed for the current package to be installed. (This is checked by R CMD
check.)

**LinkingTo** - access C++ header files in other packages.

**Additional_repositories** - list of package repositories other than CRAN. For
  example, the 'msy' DESCRIPTION file lists http://flr-project.org/R as an
  additional repository to download packages from.

## R Packages (Wickham, r-pkgs.org)

### 8 Package Metadata

**Imports** - packages that must be present for your package to work. Any time
  your package is stalled, those packages will also be installed, if not already
  present. It does not mean that it will be attached with `library(yourpkg)`.

**Suggests** - packages that are either needed for development tasks or might
  unlock optional functionality for your users. Example datasets, run tests,
  build vignettes. A suggested package is not automatically installed. Suggests
  is a courtesy to users to accommodate lean installation.

**Depends** - before R 2.14 namespaces in 2011, this was the only way to depend
  on another package. Now, you should almost always use Imports instead of
  Depends. Chapter 13 describes cases when you should still use Depends.

**LinkingTo** - rely on C++ code in another package.

**Enhances** - packages that are enhanced by your package. Typically, your
  package provides methods for classes defined in other packages. Wickham does
  not recommend using Enhances, as it's hard to define what it means.

### 13 Namespace

**13.2 Search Path**

The search path is a list of all packages you have attached.

There's an important difference between loading and attaching a package.
Normally when you talk about loading a package you think of `library()`, but
that actually attaches the package.

*Loading* will load code, data, and any DLLs, register S3/S4 methods, and run
the `.onLoad()` function. After loading, the package is available in memory, but
because it's not in the search path, you won't be able to access its components
without using the `::` accessor. Confusingly, `::` will also load a package
automatically if it isn't already loaded. It's rare to load a package
explicitly, but you can do so with `requireNamespace()` or `loadNamespace()`.

*Attaching* puts the package in the search path. You can't attach a package
without first loading it, so `library()` and `require()` load then attach the
package. You can see the currently attached package with `search()`.

Of the four (`loadNamespace`, `requireNamespace`, `library`, `require`), you
should only ever use two: `library` and `requireNamespace`.

The main difference between Depends and Imports is that Imports just *loads* the
package, Depends *attaches* it.

Unless there is a good reason otherwise, you should always list packages in
Imports and not Depends. That's because a good package is self-contained, and
minimizes changes to the global environment, including the search path.

The only exception is if your package is designed for people to be used in
conjunction with another package, rather than by itself.

**13.3 The NAMESPACE**

In total, there are eight namespace directives. Four describe exports:

- `export()`: export functions (including S3 and S4 generics).
- `exportPattern()`: export all functions that match a pattern.
- `exportClasses()`, `exportMethods()`: export S4 classes and methods.
-  `S3method()`: export S3 methods.

And four describe imports:

- `import()`: import all functions from a package.
- `importFrom()`: import selected functions (including S4 generics).
- `importClassesFrom()`, `importMethodsFrom()`: import S4 classes and methods.
- `useDynLib()`: import a function from C. This is described in more detail in
  compiled code.

Wickham recommends handling the NAMESPACE file using the roxygen2 package.

### 13.6 Imports

It's common for packages to be listed in Imports in DESCRIPTION, but not in
NAMESPACE. Wickham recommends list the package as in import in DESCRIPTION so
that it's installed, but instead of importing functions always refer to it
explicitly with `pkg::fun()`.

If you are using functions repeatedly, you can avoid `::` by importing the
function with `@importFrom pkg fun`.

---

## Thoughts

The easiest approach is Depends.

Importantly, it fully attaches TAF when calling library(icesTAF), which is
exactly what we need.

### draft.data

There is mainly one aspect of the core TAF package that has ICES-specific
behavior. This is when `draft.data` and `process.entry` check the value of the
'access' field of DATA.bib and raise an error if the value is different from
`"OSPAR"`, `"Public"`, or `"Restricted"`.

This can be a minor nuisance for TAF users outside of ICES who might prefer not
to specify this, or are working in an organization that has data vocabularies
and policies that are slightly different from ICES.

In an ideal world, perhaps, the behavior of the core TAF package would not
enforce anything regarding the 'access' field, while icesTAF would enforce the
ICES vocabularies.

The way R seems to work, however, is that if we redefine `draft.data` in
icesTAF, this will display a warning every time library(icesTAF) is called. That
cost would be too high, replacing a minor nuisance with a major one :) Another
approach would be to have icesTAF import every TAF function except `draft.data`.

Even if R allowed the packages to have different versions of `draft.data` in the
two packages, it would come with a cost. Every ICES scientist would have to be
reminded to always use library(icesTAF) instead of library(TAF). It would result
in confusion, cognitive load, and an unnecessary schism.

---

It seems, perhaps, that this might be an example where TAF users will just have
to respect that ICES scientists get the highest priority support. TAF users will
have to live with the default access="Public" when using draft.data. After
trying access=FALSE, access=NA, acess=NULL, access="", they may realize that
they can manually delete the line from the DATA.bib file.

The current implementation of `process.entry()` is that it allows 'access' to be
missing, but raises an error if there is an 'access' entry with anything else
than `"OSPAR"`, `"Public"`, or `"Restricted"`. In other words, `draft.data` has
a stricter check than `process.entry`, probably to allow older TAF analyses
(before the 'access' field was introduced) to run without errors.

This seems like a happy medium. With `draft.data()` we guide ICES scientist to
use the ICES-specific 'access' entries, but with `process.entry()` we allow TAF
users to omit the 'access' entry if they want to.

An interesting situation arises if, for example, FAO would like to use
FAO-specific access entries. The practical solution would be to use a custom
field for that, such as `fao_access = {SOFIA}`. This field could be parsed and
checked by the SOFIA package.

---

To summarize, the ICES-specific 'access' in the core TAF package is a very minor
issue, but as a thought exercise it highlights interesting technicalities and
emerging issues that come with the icesTAF-as-thin-layer-on-top-of-TAF design.

---

## Second Look at Imports

One question is whether **icesTAF** can import and export functions from the
**TAF** package. For example, there is some benefit of providing
`icesTAF:mkdir`. (To clarify, is the main benefit to support existing scripts
that may have used namespace calls like this?)

An existing analogous example is how **devtools** tunnels into **remotes** and
provides `devtools::install_github`. Before the **devtools** package was split,
this call had been written like this in many scripts, people had memorized it,
and it is found in various R tutorials and textbooks.

### Experiments

A simple experimental approach is to try different things and see (1) if a build
check raises notes/warnings, (2) if `icesTAF::mkdir` works, (3) what `?mkdir`
does, and (4) if there are any startup messages when calling `library(icesTAF)`.

**Experiment 1** *- icesTAF-import.R to import everything and export everything*

File               | Entry            | Notes
------------------ | ---------------- | ------------------
`DESCRIPTION`      | `Imports: TAF`   |
`icesTAF-import.R` | `@import TAF`    |
`icesTAF-import.R` | `@aliases mkdir` | for every function
`icesTAF-import.R` | `@export mkdir`  | for every function

https://github.com/ices-taf-dev/icesTAF/tree/import

Results from this experiment:

1. Builds without notes or warnings
2. `icesTAF::mkdir` works
3. `?mkdir` brings up a nuisance pre-help page, asking the user to choose
   between two help pages
4. `library(icesTAF)` runs without startup messages

### The devtools approach

The **devtools** approach to tunnel into **remotes** to provide
`devtools::install_github` can be seen in the `remotes.R` file:

```
#' @importFrom remotes install_github
#' @rdname remote-reexports
#' @export
install_github <- with_pkgbuild_build_tools(with_ellipsis(remotes::install_github))
```

https://github.com/r-lib/devtools/blob/v2.4.3/R/remotes.R#L63-L66

This seems to give a similar result as Experiment 1 above, bringing up a
nuisance pre-help page that asks the user to choose between two help pages.

### Depends vs. Imports

Comparison     | Depends             | Imports
-------------- | ------------------- | ------------------------------------------------------------------------------------------
Maintenance    | None                | [icesTAF-import.R](https://github.com/ices-taf-dev/icesTAF/blob/import/R/icesTAF-import.R)
Help pages     | Instant             | nuisance pre-help page for every function
icesTAF::mkdir | No, only TAF::mkdir | Yes, both icesTAF::mkdir and TAF::mkdir
