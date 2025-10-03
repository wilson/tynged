### Tynged

_tynged_ (TUNG-ed)  
Noun: fate, destiny  
Verb: to swear an oath  

This was always going to happen.

### Mission

> Build the most ergonomic, feature-complete, and trustworthy macro-based taint-tracking framework for production use.
> Tynged uses compile-time taint tracking to prevent data from untrusted sources from contaminating sensitive sinks, eliminating vulnerabilities like SQL injection, command injection, and XSS.

### Desired Properties

- Purely macro-based.
- Ergonomic, no incentives not to use it. Relatively terse, e.g. `Tyng<T, Taint>`.
- A suite of attribute and derive macros to make the process of tainting sources and sanitizing sinks as seamless as possible.
- Trustworthy and detailed documentation.
- Compile-time errors that are precise, helpful, and feel native to the Rust compiler, e.g. investigate `syn` and `proc-macro2`.
- Zero-friction `async/await` support.

### Non-Features

- Extensive backwards-compatibility with older Rust.
- Preventing implicit leaks via control flow, unless someone has an amazing idea.
- Anything lame.
