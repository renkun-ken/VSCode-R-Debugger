# R Debugger

This extension adds debugging capabilities for the R programming language.

**UPDATE:** The Debugger is now able to debug arbitrary R files and does not depend on a main()-function anymore!

## Using the Debugger
* Install the **R Debugger** extension in VS Code.
* Install the **vscDebugger** package in R (https://github.com/ManuelHentschel/vscDebugger).
* Make sure the settings `rdebugger.terminal.*` contain valid path to terminal program.
* If your R path is neither in Windows registry nor in `PATH` environment variable, make sure to provide valid R path in `rdebugger.rterm.*`.
* Press F5 and select `R Debugger` as debugger. With the default launch configuration, the debugger will start a new R session.
* To run a file, focus the file in the editor and press F5 (or the continue button in the debug controls)
* Output will be printed to the debug console, expressions entered into the debug console are evaluated in the currently active frame


## Installation
The VS code extension can be run from source by opening the project repo's root directory in vscode and pressing F5.

Alternatively the VS Code extension can be installed form the .vsix-file found on https://github.com/ManuelHentschel/VSCode-R-Debugger/actions?query=workflow%3Amain.
To download the correct file, filter the commits by branch (develop or master), click the latest commit,
and download the file `r-debugger.vsix` under the caption "Artifacts".


To install the latest development version from GitHub, run the following command in R:
```r
devtools::install_github("ManuelHentschel/vscDebugger", ref = "develop")
```
To install from the master branch, omit the argument `ref`.


**Warning:** Currently there is no proper versioning/dependency system in place, so make sure to download both packages/extensions from the same branch (Master/develop) and at the same time.


## Features
The debugger includes the following features:
* Controlling the program flow using *step*, *step in*, *step out*, *continue*
* Breakpoints 
* Exception handling to a very limited extent (breaks on exception)
* Information about the stack trace, scopes, and variables in each frame/scope
* Evaluation of arbitrary R code in the selected stack frame
* Overwriting `print()` and `cat()` with modified versions that also print the current source file and line the the debug console
* Overwriting `source()` with `.vsc.debugSource()` to allow recursive debugging (i.e. breakpoints in files that are `source()`d from within another file)


## How it works
The debugger works as follows:
* A child process running a terminal application (bash, cmd.exe, ...) is started
* An R process is started inside the child process
* The R package `vscDebugger` is loaded.
* The Debugger starts and controls R programs by sending input to stdin of the child process
* After each step, function call etc., the debugger calls functions from the package `vscDebugger` to get info about the stack/variables

The output of the R process is read and parsed as follows:
* Information sent by functions from `vscDebugger` is encoded as json and surrounded by keywords (e.g. "<v\\s\\c>").
These lines are parsed by the VS Code extension and not shown to the user.
* Information printed by the `browser()` function is parsed and used to update the source file/line highlighted inside VS Code.
These lines are also hidden from the user.
* Everything else is printed to the debug console


## Warning
Since the approach of parsing text output meant for human users is rather error prone, there are probably some cases that are not implemented correctly yet.
In case of unexpected results, use `browser()` statements and run the code directly from a terminal (or RStudio).

In the following cases the debugger might not work correctly:
* Custom `trace()` or `browser()`-statements in the code
* Custom error-handling (the debugger uses a custom `options(error=...)` to show stack trace etc. on error)
* Any form of (interactive) user input in the terminal during runtime
* Extensive usage of `cat()` without linebreaks (try disabling the `overwrite cat` and `overwrite print` options in the extion settings)
* Output to stdout that looks like output from `browser()`, the input prompt, or text meant for the debugger (e.g. `<v\s\c>...</v\s\c>`)
* Code that contains calls to `sys.calls()`, `sys.frames()`, `attr(..., 'srcref')` etc.,
since these results might be altered by intermediate calls to functions from the vscDebugger package
* Any use of graphical output/input, stdio-redirecting, `sink()`
* Extensive use of lazy evaluation, promises, side-effects:
In the general case, the debugger recognizes unevaluated promises and preserves them.
It might be possible, however, that the gathering of information about the stack/variables leads to unexpected side-effects.



## Debugging R Packages
In principle R packages can also be debugged using this extension.
Some details need to be considered:
* The package must be installed from code using `--with-keep.source` 
* The modified `print()` and `cat()` versions are not used by calls from within the package.
In order to use these, import the `vscDebugger` extension in your package, assign `print <- vscDebugger::.vsc.print` and `cat <- vscDebugger::.vsc.cat`, and deactivate the modified `print`/`cat` statements in the debugger settings.
Don't forget to remove these assignments after debugging.


## To do
The following topics could be improved/fixed in the future.

Variables/Stack view
* Properly display info about S3/S4 classes
* Summarize large lists (min, max, mean, ...)
* Load large workspaces/lists in chunks (currently hardcoded 1000 items maximum)
* Enable copying from variables list

Exception handling
* Properly display exception info (how do I show the large red box?)
* (Getting the corresponding info from R should be doable)
* Select behaviour for exceptions (enter browser, terminate, ...?)

Breakpoints
* Auto adjustment of breakpoint position to next valid position
* Conditional breakponts, data breakpoints
* Setting of breakpoints during runtime (currently these are silently ignored)

General
* Improve error handling
* Handling graphical output etc.?
* Source line info does not work for modified `print()` when called from a line with breakpoint
* Attach to currently open R process instead of spawning a new one?
* Nested formatting of output in the debug console (use existing functionality from variables view)

Give user more direct access to the R session:
* Use (visible) integrated terminal instead of background process
* Use `sink(..., split=TRUE)` to simultaneously show stdout to user and the debugger
* Return results from vscDebugger-Functions via a pipe etc. to keep stdout clean

If you have any problems, suggestions, bug fixes etc. feel free to open an issue at
https://github.com/ManuelHentschel/VSCode-R-Debugger/issues
or submit a pull request.
