[![Build Status Linux and macOS][travis-badge]][travis-link]
[![Build Status Windows][appveyor-badge]][appveyor-link]
[![Code Coverage][codecov-badge]][codecov-link]
[![Join the chat at https://gitter.im/CLI11gitter/Lobby][gitter-badge]][gitter-link]
[![License: LGPL v2.1][license-badge]](./LICENSE)

# CLI11: Command line parser for C++11

CLI11 provides all the features you expect in a powerful command line parser, with a beautiful, minimal syntax and no dependencies beyond C++11. It is header only, and comes in a single file form for easy inclusion in projects. It is easy to use for small projects, but powerful enough for complex command line projects, and can be customized for frameworks.
It is tested on [Travis][travis-link] and [AppVeyor][appveyor-link], and is being included in the [GooFit GPU fitting framework][goofit-link]. It was inspired by [`plumbum.cli`][plumbum-link] for Python. It has both a user friendly introduction here, as well as [API documentation][api-docs] enerated by Travis.


### Why write another CLI parser?

An acceptable CLI parser library should be all of the following:

* Easy to include (i.e., header only, one file if possible, no external requirements): While many programs depend on Boost, that should not be a requirement if all you want is CLI parsing.
* Short Syntax: This is one of the main points of a CLI parser, it should make variables from the command line nearly as easy to define as any other variables. If most of your program is hidden in CLI parsing, this is a problem for readability.
* C++11 or better: Should work with GCC 4.7+ (such as GCC 4.8 on CentOS 7) or above, or Clang 3.5+, or MSVC 2015+. (Note: for CLI11, Clang 3.4 only fails because of tests, googlemock does not support it.)
* Work on Linux, macOS, and Windows.
* Well tested using [Travis][travis-link] (Linux and macOS) and [AppVeyor][appveyor-link] (Windows). "Well" is defined as having good coverage measured by [CodeCov][codecov-link].
* Clear help printing.
* Standard shell idioms supported naturally, like grouping flags, a positional separator, etc.
* Easy to execute, with help, parse errors, etc. providing correct exit and details.
* Easy to extend as part of a framework that provides "applications" to users.
* Usable subcommand syntax, with support for multiple subcommands, nested subcommands, and optional fallthrough (explained later).
* Ability to add a configuration file (`ini` format).
* Produce real values that can be used directly in code, not something you have pay compute time to look up, for HPC applications.

The major CLI parsers for C++ include:

* [Boost Program Options][boost-link]: A great library if you already depend on Boost, but its pre-C++11 syntax is really odd and setting up the correct call in the main function is poorly documented (and is nearly a page of code). A simple wrapper for the Boost library was originally developed, but was discarded as CLI11 became more powerful. The idea of capturing a value and setting it originated with Boost PO.
* [The Lean Mean C++ Option Parser][optionparser-link]: One header file is great, but the syntax is atrocious, in my opinion. It was quite impractical to wrap the syntax or to use in a complex project. It seems to handle standard parsing quite well.
* [TCLAP][tclap-link]: The not-quite-standard command line parsing causes common shortcuts to fail. It also seems to be poorly supported, with only minimal bugfixes accepted. Header only, but in quite a few files. Has not managed to get enough support to move to GitHub yet. No subcommands. Produces wrapped values.
* [Cxxopts][cxxopts-link]: C++11, single file, and nice CMake support, but requires regex, therefore GCC 4.8 (CentOS 7 default) does not work. Syntax closely based on Boost PO, so not ideal but familiar.

So, this library was designed to provide a great syntax, good compiler compatibility, and minimal installation fuss.

## Current status:

This library was built to supply the Application object for the GooFit CUDA/OMP fitting library. Before version 2.0 of GooFit is released, this library will reach version 1.0 status. The current tasks still planned are:

* Add new features of config to save
* Collect user feedback
* Evaluate compatibility with [ROOT][root-link]'s TApplication object.
* Test "adding to cmake" method

See the [changelog](./CHANGELOG.md) or [GitHub releases][github-releases] for details.

## Things not supported by this library

As you probably have guessed, the list of features above are all covered by this library. There are some other features that are intentionally not supported by this library:

* Non-standard variations on syntax, like `-long` options. This is non-standard and should be avoided, so that is enforced by this library.
* Completion of partial options, such as Python's `argparse` supplies for incomplete arguments. It's better not to guess. Most third party command line parsers for python actually reimplement command line parsing rather than using argparse because of this design flaw.
* In C++14, you could have a set of `callback` methods (tested in a branch). Not deemed worth having a C++14 variation on API and removed.
* Autocomplete: This might eventually be added to both Plumbum and CLI11, but it is not supported yet.


## Installing

To use, there are two methods:

1. Copy `CLI11.hpp` from the [most recent release][github-releases] into your include directory, and you are set. This is combined from the source files  for every release.
2. Checkout the repository and add as a subdirectory for CMake. You can use the CLI interface target. (CMake 3.4+ recommended)

To build the tests, checkout the repository and use CMake:

```bash
mkdir build
cd build
cmake ..
make
GTEST_COLOR=1 CTEST_OUTPUT_ON_FAILURE=1 make test
```

## Adding options

To set up, add options, and run, your main function will look something like this:

```cpp
CLI::App app{"App description"};

std::string filename = "default";
app.add_option("-f,--file", file, "A help string");

try {
    app.parse(argc, argv);
} catch (const CLI::ParseError &e) {
    return app.exit(e);
}
```

The initialization is just one line, adding options is just two each. The try/catch block ensures that `-h,--help` or a parse error will exit with the correct return code. (The return here should be inside `main`). After the app runs, the filename will be set to the correct value if it was passed, otherwise it will be set to the default. You can check to see if this was passed on the command line with `app.count("--file")`.

The supported values are:

```cpp
app.add_options(option_name,
                variable_to_bind_to, // int, float, vector, or string-like
                help_string="",
                default=false)

app.add_flag(option_name,
             int_or_bool = nothing,
             help_string="")

app.add_set(option_name,
            variable_to_bind_to,
            set_of_possible_options,
            help_string="",
            default=false)

app.add_set_ignore_case(... // String only

App* subcom = app.add_subcommand(name, discription);
```

An option name must start with a alphabetic character or underscore. For long options, anything but an equals sign or a comma is valid after that. Names are given as a comma separated string, with the dash or dashes. An option or flag can have as many names as you want, and afterward, using `count`, you can use any of the names, with dashes as needed, to count the options. One of the names is allowed to be given without proceeding dash(es); if present the option is a positional option, and that name will be used on help line for its positional form. If you want the default value to print in the help description, pass in `true` for the final parameter for `add_option` or `add_set`.

### Example

* `"one,-o,--one"`: Valid as long as not a flag, would create an option that can be specified positionally, or with `-o` or `--option`
* `"this"` Can only be passed positionally
* `"-a,-b,-c"` No limit to the number of non-positional option names


The add commands return a pointer to an internally stored `Option`. If you set the final argument to true, the default value is captured and printed on the command line with the help flag. This option can be used directly to check for the count (`->count()`) after parsing to avoid a string based lookup. Before parsing, you can set the following options:

* `->required()`: The program will quit if this option is not present. This is `mandatory` in Plumbum, but required options seems to be a more standard term. For compatibility, `->mandatory()` also works.
* `->expected(N)`: Take `N` values instead of as many as possible, only for vector args.
* `->requires(opt)`: This option requires another option to also be present, opt is an `Option` pointer.
* `->excludes(opt)`: This option cannot be given with `opt` present, opt is an `Option` pointer.
* `->envname(name)`: Gets the value from the environment if present and not passed on the command line.
* `->group(name)`: The help group to put the option in. No effect for positional options. Defaults to `"Options"`. `"Hidden"` will not show up in the help print.
* `->ignore_case()`: Ignore the case on the command line (also works on subcommands, does not affect arguments).
* `->check(CLI::ExistingFile)`: Requires that the file exists if given.
* `->check(CLI::ExistingDirectory)`: Requires that the directory exists.
* `->check(CLI::NonexistentPath)`: Requires that the path does not exist.
* `->check(CLI::Range(min,max))`: Requires that the option be between min and max (make sure to use floating point if needed). Min defaults to 0.

These options return the `Option` pointer, so you can chain them together, and even skip storing the pointer entirely. Check takes any function that has the signature `bool(std::string)`. If you want to change the default help option, it is available through `get_help_ptr`. If you just want to see the unconverted values, use `.results()` to get the `std::vector<std::string>` of results.


On the command line, options can be given as:

* `-a` (flag)
* `-abc` (flags can be combined)
* `-f filename` (option)
* `-ffilename` (no space required)
* `-abcf filename` (flags and option can be combined)
* `--long` (long flag)
* `--file filename` (space)
* `--file=filename` (equals)

Extra positional arguments will cause the program to exit, so at least one positional option with a vector is recommended if you want to allow extraneous arguments.
If you set `.allow_extras()` on the main `App`, the parse function will return the left over arguments instead of throwing an error.
If `--` is present in the command line,
everything after that is positional only.


## Subcommands

Subcommands are supported, and can be nested infinitly. To add a subcommand, call the `add_subcommand` method with a name and an optional description. This gives a pointer to an `App` that behaves just like the main app, and can take options or further subcommands. Add `->ignore_case()` to a subcommand to allow any variation of caps to also be accepted. Children inherit the current setting from the parent. You cannot add multiple matching subcommand names at the same level (including ignore
case).
If you want to require at least one subcommand is given, use `.require_subcommand()` on the parent app. You can optionally give an exact number of subcommands to require, as well.

All `App`s have a `get_subcommands()` method, which returns a list of pointers to the subcommand passed on the command line. A simple compare of these pointers to each subcommand allows choosing based on subcommand, facilitated by a `got_subcommand(App_or_name)` method that will check the list for you. For many cases, however, using an app's callback may be easier. Every app executes a callback function after it parses; just use a lambda function (with capture to get parsed values) to `.set_callback`. If you throw `CLI::Success`, you can
even exit the program through the callback. The main `App` has a callback slot, as well, but it is generally not as useful.
If you want only one, use `app.require_subcommand(1)`. You are allowed to throw `CLI::Success` in the callbacks.
Multiple subcommands are allowed, to allow [`Click`][click-link] like series of commands (order is preserved).

There are several options that are supported on the main app and subcommands. These are:

* `.ignore_case()`: Ignore the case of this subcommand. Inherited by added subcommands, so is usually used on the main `App`.
* `.fallthrough()`: Allow extra unmatched options and positionals to "fall through" and be matched on a parent command. Subcommands always are allowed to fall through.
* `.require_subcommand()`: Require 1 or more subcommands. Accepts an integer argument to require an exact number of subcommands.
* `.add_subcommand(name, description="")` Add a subcommand, returns a pointer to the internally stored subcommand.
* `.got_subcommand(App_or_name)`: Check to see if a subcommand was recieved on the command line
* `.get_subcommands()`: The list of subcommands given on the command line
* `.set_callback(void() function)`: Set the callback that runs at the end of parsing. The options have already run at this point.
* `.allow_extras()`: Do not throw an error if extra arguments are left over (Only useful on the main `App`, as that's the one that throws errors).

## Configuration file

```cpp
app.add_config(option_name,
               default_file_name="",
               help_string="Read an ini file",
               required=false)
```

Adding a configuration option is special. If it is present, it will be read along with the normal command line arguments. The file will be read if it exists, and does not throw an error unless `required` is `true`. Configuration files are in `ini` format. An example of a file:

```ini
; Commments are supported, using a ;
; The default section is [default], case insensitive

value = 1
str = "A string"
vector = 1 2 3

; Section map to subcommands
[subcommand]
in_subcommand = Wow
sub.subcommand = true
```

Spaces before and after the name and argument are ignored. Multiple arguments are separated by spaces. One set of quotes will be removed, preserving spaces (the same way the command line works). Boolean options can be `true`, `on`, `1`, `yes`; or `false`, `off`, `0`, `no` (case insensitive). Sections (and `.` separated names) are treated as subcommands (note: this does not mean that subcommand was passed, it just sets the "defaults".


## Subclassing

The App class was designed allow toolkits to subclass it, to provide default options and setup/teardown code. Subcommands remain an unsubclassed `App`, since those are not expected to need setup and teardown. The default `App` only adds a help flag, `-h,--help`, but provides an option to disable it in the constructor (and in `add_subcommand`). You can remove options if you have pointers to them using `.remove_option(opt)`. You can add a `pre_callback` override to customize the after parse
but before run behavoir, while
still giving the user freedom to `set_callback` on the main app.  

The most important parse function is `parse(std::vector<std::string>)`, which takes a reversed list of arguments (so that `pop_back` processes the args in the correct order). `get_help_ptr` and `get_config_ptr` give you access to the help/config option pointers. The standard `parse` manually sets the name from the first argument, so it should not be in this vector.

Also, in a related note, the `App` you get a pointer to is stored in the parent `App` in a `unique_ptr`s (like `Option`s) and are deleted when the main `App` goes out of scope.


## How it works

Every `add_` option you have seen so far depends on one method that takes a lambda function. Each of these methods is just making a different lambda function with capture to populate the option. The function has full access to the vector of strings, so it knows how many times an option was passed or how many arguments it recieved (flags add empty strings to keep the counts correct). The lambda returns `true` if it could validate the option strings, and
`false` if it failed.

### Example

```cpp
app.add_option("--fancy-count", [](std::vector<std::string> val){
    std::cout << "This option was given " << val.size() << " times." << std::endl
    });
```

## Contributing

To contribute, open an [issue][github-issues] or [pull request][github-pull] on GitHub, or ask a question on [gitter][gitter-link].

## Other libraries

If you use the [`rang`][rang-link] library to add color to your terminal in a safe, multi-platform way, you can combine it with CLI11 nicely:

```cpp
std::atexit([](){std::cout << rang::style::reset;});
try {
    app.parse(argc, argv);
} catch (const CLI::ParseError &e) {
    std::cout << (e.exit_code==0 ? rang::fg::blue : rang::fg::red);
    return app.exit(e);
}
```

This will print help in blue, errors in red, and will reset before returning the terminal to the user.


[travis-badge]:      https://img.shields.io/travis/henryiii/CLI11/master.svg?label=Linux/macOS
[travis-link]:       https://travis-ci.org/henryiii/CLI11
[appveyor-badge]:    https://img.shields.io/appveyor/ci/HenrySchreiner/cli11/master.svg?label=Windows
[appveyor-link]:     https://ci.appveyor.com/project/HenrySchreiner/cli11
[codecov-badge]:     https://codecov.io/gh/henryiii/CLI11/branch/master/graph/badge.svg
[codecov-link]:      https://codecov.io/gh/henryiii/CLI11
[gitter-badge]:      https://badges.gitter.im/CLI11gitter/Lobby.svg
[gitter-link]:       https://gitter.im/CLI11gitter/Lobby
[license-badge]:     https://img.shields.io/badge/License-LGPL%20v2.1-blue.svg
[github-releases]:   https://github.com/henryiii/CLI11/releases
[github-issues]:     https://github.com/henryiii/CLI11/issues
[github-pull]:       https://github.com/henryiii/CLI11/pulls
[goofit-link]:       https://GooFit.github.io 
[plumbum-link]:      https://plumbum.readthedocs.io/en/latest/
[click-link]:        http://click.pocoo.org
[api-docs]:          https://henryiii.github.io/CLI11/index.html
[rang-link]:         https://github.com/agauniyal/rang/wiki
[boost-link]:        http://www.boost.org/doc/libs/1_63_0/doc/html/program_options.html
[optionparser-link]: http://optionparser.sourceforge.net
[tclap-link]:        http://tclap.sourceforge.net
[cxxopts-link]:      https://github.com/jarro2783/cxxopts
[root-link]:         https://root.cern.ch
