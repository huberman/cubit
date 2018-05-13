# Cubit
A unit-testing framework for embedded C

# An example
```C
#define CUBIT_DEFINE_FRAMEWORK
#include "cubit.h"

TEST(first, failing_test)
{
    ASSERT(0);      // test fails
}

TEST(first, string_eq)
{
    const char * expected = "hello";
    STRCMP_EQ(expected, "hello");   // test passes
}

// fixture setup called before each test that is 
// part of fxt1

static int x = 0;

void fxt1_setup(void) { x = 3; }

TEST(fxt1, before)
{
    LONG_EQ(3, x);      // test passes
}

// disabled test
TEST(DISABLED_foo, not_called)
{
    ASSERT(false);      // not called
}
```

# Introduction
Cubit has been designed with the following goals in mind:
* Usability
* Lightweight
* Full-featured
* Configurable
* Testable

It is written in C99 and uses GCC features to make it easy to use.

## Usability
A good developer experience is key to adopting a framework.  Cubit has been designed to be easy to use with no need to register tests manually, create fixtures manually or to run external scripts.  It is a header-only library that can be used with a single test file or it can be used as part of a multi-file project.

## Lightweight
Cubit is one single header file with no link-time requirements other than the standard C library.  It is only a few hundred lines long so compilation times are minimal.  The code is simple and should be readily understandable.  It does not use dynamic memory allocation.

## Full-featured
Cubit offers fixtures with setup and teardown before and after tests, as well as setup and teardown before and after all tests are run.  Reporting can be extended with custom reporters.  Tests can also be disabled.

## Configurable
Embedded development environments vary considerably in their capabilities and in what they require.  Cubit can be configured in multiple ways to suit the environment in which it is used.  

## Testable
A unit-testing framework should itself be well tested.  Cubit has been designed to be testable and has its own internal self tests that can be run using Cubit.  The functionality to do this can be used by reporting extensions so that they too can be unit tested effectively.

# Using Cubit
## Single-file use
The Cubit header file cubit.h contains both the client code (the code used by the developer using Cubit) and the code for the framework itself.  By default the framework code is not compiled (it is #ifdef'd out) to avoid multiple definition problems.  By defining CUBIT_DEFINE_FRAMEWORK, either using a #define before including cubit.h or on the compiler command line, the framework code will be included and compiled.  Do not include a main() function in your own code; Cubit provides its own main().

## Multi-file use
If multiple test files are used then the easiest way of using Cubit is to create a separate C file containing just Cubit and then linking it into the test executable.  For example, you can compile these three C files together:
```C
// cubit_main.c
#define CUBIT_DEFINE_FRAMEWORK
#include "cubit.h"

// test1.c
#include "cubit.h"
TEST(test1, a) { ASSERT(1); }

// test2.c
#include "cubit.h"
TEST(test2, a) { ASSERT(2); }
```

# Using fixtures
Tests often need something to happen before each test.  This is often setting up state for the test, such as initialising variables.  Some tests may also need some post-test action to be performed too, such as cleaning up state by deallocating memory or closing a file.

All tests in Cubit are by default fixture-aware, meaning that they will look for functions with a given name and execute them if they are present.  The naming convention for a pre-test function is <fixture>_setup and the corresponding teardown function is <fixture>_teardown.  For example, if a test is called TEST(mysuite, x) then the corresponding fixture functions are called mysuite_setup() and mysuite_teardown().  There is no need to create dummy or empty versions of these functions if you don't require them.

# Disabling tests
Any test whose suite name starts with "DISABLED_" will not be executed.  For instance, TEST(DISABLED_foo, x) {} will not be executed.

# Reporting test status
By default, Cubit reports the output of tests in a compact format using one character per test: | for a passing test, F for a failing test, I for an ignored test, and ? for an unknown status.  In addition, failing tests show the text from the failing assertion.

ANSI console colours are used to provide a "green bar/red bar" experience similar to other frameworks.  If you do not want the ANSI colouring then you can define CUBIT_NO_ANSI to turn off colouring.

Cubit reports the number of failing tests as the return value of main() and zero if no tests failed.

# Configuring Cubit
Cubit is configured using C preprocessor defines.  This method was chosen for a number of reasons:
* It keeps the code small and simple
* It allows pieces of code that may not be supported on a given platform to be completely removed.  For example, there may not be a command line CLI, or a standard output device, or a filesystem available
* It allows some of the implementation constants to be overridden (such as the maximum number of tests for which to allocate memory)

Here is a list of all of the preprocessor values that can be configured.  All of these can be set either directly in the code or using gcc's -D command-line flag (e.g. gcc -D CUBIT_DEFINE_FRAMEWORK test.c):

    CUBIT_DEFINE_FRAMEWORK
    CUBIT_NO_ANSI
    CUBIT_NO_DEFAULT_REPORTER
    CUBIT_MAX_TESTS

# Limits
Cubit uses a fixed size static buffer to hold its state.  The size of this buffer is set using CUBIT_MAX_TESTS, which by default is 10000.  If more tests than this are created then Cubit will print out an error message for each test beyond the maximum but it will still execute the non-overflowing tests.  CUBIT_MAX_TESTS can be defined to be some other value if this limit is too low, or if the amount of memory used by Cubit is too high when testing on a memory-limited target system.

# Creating a custom reporter
TODO
