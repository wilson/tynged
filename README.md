### Tynged
_tynged_ (TUNG-ed)  
Noun: fate, destiny  
Verb: to swear an oath  

This was always going to happen.

### Mission
> Build the most ergonomic, feature-complete, and trustworthy macro-based taint-tracking framework for production use.  
> Tynged uses compile-time taint tracking to prevent data from untrusted sources from contaminating sensitive sinks, eliminating vulnerabilities like SQL injection, command injection, and XSS.  

### Desired Properties
- Purely macro-based, with zero runtime overhead.
- Ergonomic, no incentives not to use it. Relatively terse, e.g. `Tyng<T, Taint>`.
- A suite of attribute and derive macros to make the process of tainting sources and sanitizing sinks as seamless as possible.
- Trustworthy and detailed documentation.
- Compile-time errors that are precise, helpful, and feel native to the Rust compiler, e.g. investigate `syn` and `proc-macro2`.
- Zero-friction `async/await` support.

### Non-Features
- Extensive backwards-compatibility with older Rust.
- Preventing implicit leaks via control flow, unless someone has an amazing idea.
- Anything lame.

### Definitions
- **Sources**: Where untrusted data originates (e.g., HTTP requests, file I/O).
- **Sinks**: Sensitive operations where untrusted data can cause harm (e.g., database queries, shell execution, HTML rendering).
- **Taint**: A marker applied to data indicating its trust level.
- **Sanitization**: The explicit process of validating or escaping data to remove its taint.

### Hypothetical Interface

```rust
use tynged::{Tyng, Untrusted, Trusted};

// [Source] Data coming from a web request (Tainted)
#[derive(Deserialize)]
struct UserInput {
    username: Tyng<String, Untrusted>,
}

// [Sink] A sensitive operation that requires sanitized input
fn execute_query(query: Tyng<String, Trusted>) {
    // ...
}

fn handler(input: UserInput) {
    // Fails to compile: expected `Trusted`, found `Untrusted`
    // execute_query(input.username);

    // [Sanitization] Success: Data has been explicitly validated
    let sanitized = input.username.sanitize_with(my_sql_escaper);
    execute_query(sanitized);
}
```
