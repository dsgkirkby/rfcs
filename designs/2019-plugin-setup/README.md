- Start Date: 2019-02-18
- RFC PR: (leave this empty, to be filled in later)
- Authors: @dsgkirkby

# Plugin Setup Step

## Summary

Allow plugins to perform expensive, cache-able computations once to be used throughout a linting run.

## Motivation

### Motivating use-cases

- [eslint-plugin-import](TODO) constructs a dependency graph of linted files before running
- [@typescript-eslint/typescript-estree](TODO) provides type checking services to plugins, which is done by running the typescript type checker on the linted files while they are being parsed

I'm not particularly familiar with the import use-case, but I've seen it mentioned in multiple places for related conversations.
I'd welcome anyone more familiar with eslint-plugin-import to add to this section (and in general give feedback on this RFC).

In typescript-eslint, in addition to running the typescript compiler on code to create the required AST for basic linting, the plugin also exposes a `getParserServices` function on the rule context that provides access to the typescript type-checker.
This is used for rules such as:
- `no-for-in-array`
- `require-array-sort-compare`
- `promise-function-async`
- `restrict-plus-operands`
- `no-unnecessary-type-assertion`
- `no-unnecessary-qualifier`

Which all rely in some way on type information.

The current implementation runs the typescript compiler on each file individually, which is ~10x slower than running it once for the whole codebase (see related discussion on typescript-eslint for more info).
This is particularly notable as `tslint` runs the compiler just once, and as such is an order of magnitude faster than typescript-eslint once these services are enabled.

### Motivating ESLint features

Any plugin that caches the result of an expensive up-front computation, like eslint-plugin-import does and typescript-eslint would like to do, is implicitly relying on ESLint running files serially.
If ESLint runs were instead done in parallel, this cache may be in the process of being populated while a file is being run (which is currently not possible), which could trigger undefined, buggy behaviour from these plugins (for example, re-performing this expensive work wastefully).

As such, an option such as this is useful for ESLint to provide to plugins that run in parallel mode, as it would avoid plugins needing to each implement complex, multi-threaded caching behaviour. 

## Detailed Design

<!--
   This is the bulk of the RFC.

   Explain the design with enough detail that someone familiar with ESLint
   can implement it by reading this document. Please get into specifics
   of your approach, corner cases, and examples of how the change will be
   used. Be sure to define any new terms in this section.
-->

## Documentation

As this is not a user-facing feature, it shouldn't need a blog post announcing it.
It should, however, have comprehensive documentation of the API and behaviour for plugin developers to use.

## Drawbacks

<!--
    Why should we *not* do this? Consider why adding this into ESLint
    might not benefit the project or the community. Attempt to think 
    about any opposing viewpoints that reviewers might bring up. 

    Any change has potential downsides, including increased maintenance
    burden, incompatibility with other tools, breaking existing user
    experience, etc. Try to identify as many potential problems with
    implementing this RFC as possible.
-->

## Backwards Compatibility Analysis

Existing ESLint users should not see any issues as this is an entirely additive feature.

However, `eslint-plugin-import` and `typescript-eslint` will need to manage the potential absence of this feature, either by maintaining current logic as a fallback or by establishing strict `peerDependencies` versioning. 

## Alternatives

<!--
    What other designs did you consider? Why did you decide against those?

    This section should also include prior art, such as whether similar
    projects have already implemented a similar feature.
-->

## Open Questions

<!--
    This section is optional, but is suggested for a first draft.

    What parts of this proposal are you unclear about? What do you
    need to know before you can finalize this RFC?

    List the questions that you'd like reviewers to focus on. When
    you've received the answers and updated the design to reflect them, 
    you can remove this section.
-->

## Help Needed

As mentioned above, I have very little/no familiarity with eslint-plugin-import; I've simply seen it mentioned it similar conversations.
I'd love for someone with more background there to comment on or contribute to this to make sure it covers that use case.

## Frequently Asked Questions

<!--
    This section is optional but suggested.

    Try to anticipate points of clarification that might be needed by
    the people reviewing this RFC. Include those questions and answers
    in this section.
-->

## Related Discussions

- https://github.com/eslint/rfcs/pull/11: adding something along these lines is necessary to allow parallel linting for some of the motivating use-cases
- https://github.com/typescript-eslint/typescript-eslint/issues/243: description of performance issues in typescript-eslint that can be resolved with this option
