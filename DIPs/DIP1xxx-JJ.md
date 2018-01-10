# Default in final switch

| Field           | Value                                                           |
|-----------------|-----------------------------------------------------------------|
| DIP:            | (number/id)                                                     |
| Review Count:   | 0
| Author:         | Jan Jurzitza - dip@webfreak.org                                 |
| Implementation: |                                                                 |
| Status:         | Will be set by the DIP manager (e.g. "Approved" or "Rejected")  |

## Abstract

Adding a `default` block to `final switch` with an enum value to replace the default
behaviour on unknown cases to allow safe and efficient error recovery in critical
parts of an application using `final switch` while still preserving compile-time
errors for development.

## Rationale

`final switch` is currently less useful in code with user input for switch values
because a `catch (core.exception.SwitchError)` statement would be needed if one does
not want their application to crash in that context. By allowing a `default`
statement the programmer would be able to replace the default behaviour of throwing
a `SwitchError` with error recovery or returning.

The cases where this applies the most are applications handling with user input such
as webservers accepting input from the user and then converting some inputs to an
enum value or file loading casting the raw byte value to an enum value and then
using these values in a `final switch`.

## Description

Allow `default` blocks in `final switch` statements to replace the default behaviour
of throwing a `SwitchError` on invalid input while still keeping the `case`-existence
check for all enum members of an enum.

To implement this change compilers would need to remove their default behaviour of
rejecting `default` blocks in a `final switch` statement and only emitting the
default `throw` statement if there is no `default` block present. The compiler
should still abort compilation when there is a missing case for an enum value like
it does now. The only behaviour that should change for the developer is removing
the rejection of `default` blocks.

This behaviour should only be allowed for enum values as otherwise this change would
make `final switch` behave the same as `switch` except for the implicit `default`.

The language documentation will need to be adjusted by removing the constraint that
`DefaultStatement` is not allowed, the grammar specification would stay unchanged
for the reference implementation.

### Breaking changes / deprecation process

Currently valid code will not change in behaviour and will continue to work the
same. Only when now adding a `default` block to a `final switch` the behaviour
will be different. This might be a problem for static analysis tools which expect
a value to be a valid enum member after the `final switch` block.

### Examples

```d
enum PluginType
{
    native, script
}

PluginType type = cast(PluginType) readFileBytes[0];
final switch (type) // when adding a member to PluginType the developer will
                    // still get an error here to actually implement it
{
case PluginType.native: /* ... */ return true;
case PluginType.script: /* ... */ return true;
default: return false; // but when the file is now invalid, instead of throwing, this
                       // function will return false and leave the application running.
}
```

## Copyright & License

Copyright (c) 2017 by the D Language Foundation

Licensed under [Creative Commons Zero 1.0](https://creativecommons.org/publicdomain/zero/1.0/legalcode.txt)

## Review

Will contain comments / requests from language authors once review is complete,
filled out by the DIP manager - can be both inline and linking to external
document.
