### Tynged
_tynged_ (TUNG-ed)  
Noun: fate, destiny  
Verb: to swear an oath  

This was always going to happen.

### Mission
> Build the most ergonomic, feature-complete, and trustworthy macro-based framework for enforcing context-aware security guarantees at compile time.  
> Tynged ensures that data is correctly sanitized and encoded for its intended context (e.g., SQL, HTML, Shell), eliminating injection vulnerabilities.

### Desired Properties
- Purely macro-based, with zero runtime overhead.
- Ergonomic, no incentives not to use it. Relatively terse, e.g. `Tyng<T, Context>`.
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

### Type-Guided Safety

Tynged uses Rust's type system to track the context for which data is safe.  
The question is not "is this data trusted?"
Instead we ask, "is this data correctly encoded for where it is going?"
This approach prevents vulnerabilities caused by applying the wrong type of sanitization.

```rust
// We use marker types (Contexts) to represent the state of the data.
use tynged::{Tyng, context::{UserRaw, HtmlSafe, SqlSafe}};

// --- 1. Sources ---
// Data originating from an external source is explicitly typed as 'UserRaw'.
#[derive(Deserialize)]
struct UserInput {
    username: Tyng<String, UserRaw>,
}

// --- 2. Sinks ---
// Sinks explicitly declare the security context they require.

/// Requires data explicitly encoded for HTML output.
fn render_html(content: Tyng<String, HtmlSafe>) {
    // ... render page ...
}

/// Requires data explicitly escaped or parameterized for SQL queries.
fn execute_query(query_part: Tyng<String, SqlSafe>) {
    // ... execute query ...
}

// --- 3. Usage ---
fn handle_request(input: UserInput) {
    // --- ATTEMPT 1: Using raw data (XSS) ---
    render_html(input.username);
    /*
    error[E0308]: mismatched types
      --> src/main.rs:40:17
       |
    40 |     render_html(input.username);
       |                 ^^^^^^^^^^^^^^ expected `HtmlSafe`, found `UserRaw`
       |
       = note: expected struct `Tyng<String, HtmlSafe>`
                  found struct `Tyng<String, UserRaw>`
    */


    // --- ATTEMPT 2: Using the wrong context---
    // The developer sanitizes the data, but uses the incorrect sanitizer
    // for the destination sink. They escape for SQL but send it to HTML.
    let sql_data = input.username.encode_sql(); // UserRaw -> SqlSafe

    render_html(sql_data);
    /*
    error[E0308]: mismatched types
      --> src/main.rs:60:17
       |
    60 |     render_html(sql_data);
       |                 ^^^^^^^^ expected `HtmlSafe`, found `SqlSafe`
       |
       = note: expected struct `Tyng<String, HtmlSafe>`
                  found struct `Tyng<String, SqlSafe>`
    */

    // --- SUCCESS: Context mtches Sink ---
    // The compiler guides us to use the correct transformations.
    let html_data = input.username.encode_html(); // UserRaw -> HtmlSafe
    render_html(html_data);

    // And we can reuse the correctly encoded SQL data from above.
    execute_query(sql_data);
}
```
