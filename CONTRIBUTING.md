# Contributing to IanaIO

Hi! Thanks for your interest in contributing to IanaIO — we'd love to have your
participation! If you want help or mentorship, reach out to us in a GitHub
issue, or on [the `#IanaIO` channel of the IanaIO Rust Discord][discord] and introduce
yourself.

[discord]: https://discord.com/channels/1247475712001314857/1247475712001314860

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Code of Conduct](#code-of-conduct)
- [Building and Testing](#building-and-testing)
  - [Prerequisites](#prerequisites)
  - [Building](#building)
  - [Testing](#testing)
    - [Wasm Headless Browser Tests](#wasm-headless-browser-tests)
    - [Wasm Node Tests](#wasm-node-tests)
    - [Non-Wasm Tests](#non-wasm-tests)
  - [Formatting](#formatting)
  - [Updating `README.md`s](#updating-readmemds)
- [Workflow](#workflow)
  - [Proposing a Design](#proposing-a-design)
    - [Design Checklist](#design-checklist)
  - [Implementation and Feedback Cycle](#implementation-and-feedback-cycle)
    - [Implementation Checklist](#implementation-checklist)
- [Team Members](#team-members)
- [Publishing Releases and Versioning](#publishing-releases-and-versioning)
  - [Versioning](#versioning)
  - [Publishing Checklist](#publishing-checklist)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Code of Conduct

We abide by the [IanaIO Code of Conduct][coc] and ask that you do as well.

[coc]: https://www.iana.io/coc

## Building and Testing

### Prerequisites

These tools are required for building and testing IanaIO:

* [**The Rust toolchain:**][install-rust] `rustup`, `cargo`, `rustc`, etc.
* [**`rustfmt`:**][rustfmt] We use `rustfmt` for a consistent code style across
  the whole code base.
* [**`wasm-pack`:**][install-wasm-pack] We use `wasm-pack` to orchestrate
  headless browser testing.
* [**`cargo readme`:**][cargo-readme] We use `cargo readme` to generate
  `README.md`s from each crate's module documentation.

[install-rust]: https://www.rust-lang.org/tools/install
[rustfmt]: https://github.com/rust-lang/rustfmt
[install-wasm-pack]: https://rustwasm.github.io/wasm-pack/installer/
[cargo-readme]: https://github.com/livioribeiro/cargo-readme

### Building

You can build every IanaIO crate:

```
cargo build --all
```

Or you can build one particular crate:

```
cargo build -p my-particular-crate
```

### Testing

#### Wasm Headless Browser Tests

To run headless browser tests for a particular crate:

```shell
wasm-pack test crates/my-particular-crate --headless --firefox # or --safari or --chrome
```

#### Wasm Node Tests

To run tests in Node.js:

```shell
wasm-pack test crates/my-particular-crate --node
```

#### Non-Wasm Tests

You can run the non-Wasm tests (e.g. doc tests) for every IanaIO crate with:

```
cargo test --all
```

Or you can run one particular crate's non-Wasm tests:

```
cargo test -p my-particular-crate
```

### Formatting

To (re)format the IanaIO source code, run:

```
$ cargo fmt
```

### Updating `README.md`s

To update each `ianaio/crates/*/README.md` to reflect the crate's top-level
documentation, go to the root of the repository and run:

```
./update-readmes.sh
```

## Workflow

Designing APIs for IanaIO, its utility crates, and interfaces between them takes a
lot of care. The design space is large, and there is a lot of prior art to
consider. When coming to consensus on a design, we use a simplified, informal
version of [our RFC process][rfcs].

When making a proposal, you must create a new pull request on this repo.
The pull request's title must start with `[RFC]`.

The pull request must add a new file into the `rfcs` folder. This file
contains all the information for the proposal.

It is expected that other people will write reviews pointing out various
flaws, which should be fixed by adding new commits to the pull request.

> Note: when fixing a bug in a semver-compatible way that doesn't add any new
> API surface (i.e. changes are purely internal) we can skip the design proposal
> part of this workflow, and jump straight to a pull request.

The graph below gives an overview of the workflow for proposing, designing,
implementing, and merging new crates and APIs into IanaIO. Notably, we expect a
large amount of design discussion to happen up front in the issue thread for the
design proposal.

[![Graph showing the workflow of proposing, designing, and merging new crates and
APIs into IanaIO](./new-design-workflow.png)](./new-design-workflow.png)

[rfcs]: https://github.com/rustwasm/rfcs

### Proposing a Design

Before writing pull requests, we should have a clear idea of what is required
for implementation. This means there should be a skeleton of the API in the form
of types and function/method signatures. We should have a clear idea of the
layers of APIs we are exposing, and how they are built upon one another.

Note that exploratory implementations outside of IanaIO are encouraged during this
period to get a sense for the API's usage, but you shouldn't send an implementation
pull request until the design has been accepted.

Before the design is accepted, at least two team members must have stated that
they are in favor of accepting the design in the issue thread.

[Here is an issue template you can use for proposing
designs.](https://github.com/rustwasm/ianaio/issues/new?assignees=&labels=&template=propose_design.md&title=)

#### Design Checklist

Here is a checklist of some general design principles that IanaIO crates and APIs
should follow:

* [ ] Crate's public interface follows the [IanaIO Rust API Guidelines][api-guidelines].

* [ ] Callback-taking APIs are generic over `F: Fn(A) -> B` (or `FnMut` or
  `FnOnce`) instead of taking `wasm_bindgen::Closure`s or
  `js_sys::Function`s directly.

* [ ] If the API can be implemented as a Future / Stream, then it should first
  be implemented as a callback, with the callback API put into the `callback`
  submodule.

  Then the Future / Stream should be implemented using the callback API, and
  should be put into the `future` or `stream` submodule.

  Make sure that the callback and Future / Stream APIs properly support
  cancellation (if it is possible to do so).

* [ ] Uses nice Rust-y types and interfaces instead of passing around untyped
  `JsValue`s.

* [ ] Has `fn as_raw(&self) -> &web_sys::Whatever` functions to get the
  underlying raw `web_sys`, `js_sys`, or `JsValue` type. This provides an escape
  hatch for dropping down to raw `web_sys` bindings when an API isn't fully
  supported by the crate yet.

  Similar for `from_raw` constructors and `into_raw` conversion methods when
  applicable.

* [ ] There is a loose hierarchy with "mid-level" APIs (which are essentially
  thin wrappers over the low-level APIs), and "high-level" APIs (which make more
  substantial changes).

  As a general rule, the high-level APIs should be built on top of the mid-level
  APIs, which in turn should be built on top of the low-level APIs
  (e.g. `web_sys`)

  There are exceptions to this, but they have to be carefully decided on a
  case-by-case basis.

### Implementation and Feedback Cycle

Once we've accepted a design, we can move forward with implementation and
creating pull requests.

The implementation should generally be unsurprising, since we should have
already worked out most of the kinks during the earlier design discussions. If
there are significant new issues or concerns uncovered during implementation,
then these should be brought up in the design proposal discussion thread again,
and the evolved design reaffirmed with two team members signing off once
again.

If there are no new concerns uncovered, then the implementation just needs to be
checked over by at least one team member. They provide code review and feedback
on the implementation, then the feedback is addressed and pull request updated.
Once the pull request is in a good place and CI is passing, a team member may
approve the pull request and merge it into IanaIO. If any team member raises
concerns with the implementation, they must be resolved before the pull request
is merged.

#### Implementation Checklist

Here is a checklist that all crate and API implementations in IanaIO should
fulfill:

* [ ] The crate should be named `ianaio-foobar`, located at `ianaio/crates/foobar`,
  and re-exported from the umbrella IanaIO crate like:

  ```rust
  // ianaio/src/lib.rs

  pub use ianaio_foobar as foobar;
  ```

* [ ] The `authors` entry in `ianaio/crates/foobar/Cargo.toml` is "The Rust and
  WebAssembly Working Group".

* [ ] Uses [`unwrap_throw` and `expect_throw`][unwrap-throw] instead of normal
  `unwrap` and `expect`.

* [ ] Headless browser and/or Node.js tests via `wasm-pack test`.

* [ ] Uses `#![deny(missing_docs, missing_debug_implementations)]`.

* [ ] Crate's root module documentation has at least one realistic example.

* [ ] Crate has at least a brief description of how to use it in the IanaIO guide
  (the `mdbook` located at `ianaio/guide`).

[unwrap-throw]: https://docs.rs/wasm-bindgen/0.2.37/wasm_bindgen/trait.UnwrapThrowExt.html
[api-guidelines]: https://rust-lang-nursery.github.io/api-guidelines/

## Team Members

Team members sign off on design proposals and review pull requests to IanaIO.

* `@cichy`


If you make a handful of significant contributions to IanaIO and participate in
design proposals, then maybe you should be a team member! Think you or someone
else would be a good fit? Reach out to us!

## Publishing Releases and Versioning

### Versioning

Whenever we bump a `ianaio_whatever` utility crate's version, make sure to make a
corresponding version bump for every crate that transitively depends upon
`ianaio_whatever`. Crates that do not transitively depend on `ianaio_whatever`
should not have their version bumped.

For example, if we start with this dependency graph:

```
- ianaio @ 0.1.2
  - ianaio_foo @ 0.1.2
  - ianaio_bar @ 0.1.3
  - ianaio_qux @ 0.1.4
    - ianaio_bar @ 0.1.3
```

If we make a bug fix in `ianaio_bar` and bump its version from 0.1.3 to 0.1.4,
then the cascading version bumps should result in this final dependency graph:

```
- ianaio @ 0.1.3           # `ianaio` has deps updated, version bumped
  - ianaio_foo @ 0.1.2     # `ianaio_foo` remains at 0.1.2 since no deps changed
  - ianaio_bar @ 0.1.4     # `ianaio_bar` had bug fix -> bumped to 0.1.4
  - ianaio_qux @ 0.1.5     # `ianaio_qux` has its dep on `ianaio_bar` updated, version bumped
    - ianaio_bar @ 0.1.4
```

Historically, this versioning scheme was agreed upon in [issue #66][].

[issue #66]: https://github.com/rustwasm/ianaio/issues/66

### Publishing Checklist

Here is a checklist for publishing new releases to crates.io:

* [ ] Bump all the relevant crates' versions, as described above.

* [ ] Write a `CHANGELOG.md` entry for the release.

* [ ] Commit the changes and send a pull request for the release.

* [ ] Once a team member has approved the pull request, and continuous
      integration is green, merge the PR.

* [ ] `cargo publish` each crate that had its version bumped.

* [ ] Create a tag `X.Y.Z` for the umbrella `ianaio` crate's version, and push
      this tag to GitHub.

