---
layout:     post
title:      A comparison of regex engines
date:       2017-03-28
summary:    A performance comparison of regular expression engines including the Rust regex crate.
categories: regex
---

{% include chart.js.html %}

# Introduction
Regular expressions (or just regex) are commonly used in pattern search algorithms. There are many different
regex engines available with different support of expressions, performance constraints and language bindings.
Based on the previous work of John Maddock (See his own [regex comparison](http://www.boost.org/doc/libs/1_41_0/libs/regex/doc/gcc-performance.html))
and the sljit project (See their [regex comparison](http://sljit.sourceforge.net/regex_perf.html))
I want to give an overview of actively developed engines regarding their performance.

# Test setup
## Hardware
The performance is measured on my Dell notebook only. It's not the newest one, but it doesn't matter because
I used the same hardware for all engines and I'm interested in the performance results compared to each other.
But here are some hardware information:
- Chassis: Dell Latitude E7450
- CPU: Intel® Core™ i5-5300U
- RAM: 16GB
- SSD: Samsung PM85 256GB

## Software
The used software is also not the newest, but newer than the default packages of my installed Ubuntu 16.04:
- GCC 6.2.0
- Rustc 1.16.0 and 1.17.0-nightly

I want to get the execution time of each engine for the following regular expression:
- `Twain`
- `(?i)Twain`
- `[a-z]shing`
- `Huck[a-zA-Z]+|Saw[a-zA-Z]+`
- `\b\w+nn\b`
- `[a-q][^u-z]{13}x`
- `Tom|Sawyer|Huckleberry|Finn`
- `(?i)Tom|Sawyer|Huckleberry|Finn`
- `.{0,2}(Tom|Sawyer|Huckleberry|Finn)`
- `.{2,4}(Tom|Sawyer|Huckleberry|Finn)`
- `Tom.{10,25}river|river.{10,25}Tom`
- `[a-zA-Z]+ing`
- `\s[a-zA-Z]{0,12}ing\s`
- `([A-Za-z]awyer|[A-Za-z]inn)\s`
- `["'][^"']{0,30}[?!\.][\"']`
- `\u221E|\u2713`
- `\p{Sm}`

The set of expressions is not representative, but is sufficient to get an overview.

To measure the performance I modified the existing benchmark tool of the sljit project.
The tool is available on github: [regex-performance](https://github.com/rust-leipzig/regex-performance).
The base tool of the sljit project supported already the following regex engines:
- [Oniguruma](https://github.com/kkos/oniguruma), v6.1.3
- [RE2](https://github.com/google/re2)
- [Tre](https://github.com/laurikari/tre)
- [PCRE2](http://www.pcre.oashrg), v10.23

I added support for two more engines:
- [Hyperscan](https://github.com/01org/hyperscan), v4.4.1
- [Rust regex crate](https://doc.rust-lang.org/regex/regex/index.html), v0.2.1

### PCRE2
> Perl Compatible Regular Expressions (PCRE) is a regular expression C library inspired by the regular expression
capabilities in the Perl programming language. PCRE2 is the name used for a revised API for the PCRE library.

Beside the standard matching algorithm PCRE2 is shipped with an alternative algorithm based on a
deterministic finite automate (*DFA*) which operates in a different way and is not Perl-compatible.
A detailed description is provided within the [man pages](http://www.pcre.org/current/doc/html/pcre2matching.html).

A heavyweight optimized variant is shipped too: the just-in-time (*JIT*) compiling can greatly speed up pattern matching.

To get comparable results, the Unicode support has to be enabled with the configuration option `--enable-unicode`.
The JIT feature is optional and has to be enabled with the option `--enable-jit`.

### Hyperscan
Hyperscan is a [01.org](https://01.org/hyperscan) open source project:
> Hyperscan is a high-performance multiple regex matching library.
It follows the regular expression syntax of the commonly-used libpcre library, yet functions as a standalone library
with its own API written in C. Hyperscan uses hybrid automata techniques to allow simultaneous matching of large numbers 
of regular expressions, as well as matching of regular expressions across streams of data.

Hyperscan is a mature library with over 10 years of development. The focus of Hyperscan is the x86 platform and the library
uses hardware accelerators such as AVX to optimize the throughput.

By default, Hyperscan does not care about the start of a match. Getting the start of a match requires the set flag
`HS_FLAG_SOM_LEFTMOST` for compiling a pattern. The flag costs some performance but is required to get
comparable results.

### Rust regex crate
A Rust crate is a synonymous for a ‘library’ or ‘package’. The [Rust regex crate](https://doc.rust-lang.org/regex/regex/index.html)
provides functions for parsing, compiling, and executing regular expressions:
> Its syntax is similar to Perl-style regular expressions, but lacks a few features like look around and back-references.
In exchange, all searches execute in linear time with respect to the size of the regular expression and search text.

Except the Rust crate all engines are written in C or C++ including the test tool. It was a requirement to have
C-binding of the used engine. So an interface was necessary to call the Rust functions within C.
The solution uses the Rusts FFI (foreign function interface) to build a static
library which just counts the matches for a given expression. The complete library consists of 3 functions with less
than 50 lines of code. The main Rust function to get the matches is:
```rust
#[no_mangle]
pub extern fn regex_matches(raw_exp: *mut Regex, p: *const u8, len: u64) -> u64 {
    let exp = unsafe { Box::from_raw(raw_exp) };
    let s = unsafe { slice::from_raw_parts(p, len as usize) };

    let findings = exp.find_iter(s).count();
    Box::into_raw(exp);
    findings as u64
}
```
The function gets a raw pointer to a previously compiled expression (`raw_exp`), a raw pointer to the input c-string
(`p`) and the length of the input string (`len`). At the first the function gets the compiled expression from the
corresponding raw pointer and the input string. The converting from a raw pointer into a type is unsafe, that's why
the code parts have to be wrapped with `unsafe {}`. After all the number of matches is gathered with a call of
`exp.find_iter(s).count()`. To use the compiled expression in following function calls, the raw
pointer from the expression is gathered again. This effects that the lifetime of the expression still remains after returning.
At the end the function returns the number of matches as a 64bit value to the caller.

The corresping C function prototype is:
```c
struct Regex;       // anonymous declaration

extern uint64_t regex_matches(struct Regex const * const exp, uint8_t * const str, uint64_t str_len);
```

# Results
To get the test results of the test tool call within the build directory of the tool:
```bash
./src/regex_perf -f ../3200.txt -o results.csv
```
The tool prints the details and writes the results per test and engine to `results.csv`.
At the end a short summary of the results is printed:
```
Total Results:
[      pcre] time:  12626.7 ms, score:      8 points,
[  pcre-dfa] time:  14135.2 ms, score:      0 points,
[  pcre-jit] time:   1050.6 ms, score:     47 points,
[       re2] time:    946.1 ms, score:     26 points,
[      onig] time:   2475.8 ms, score:      4 points,
[       tre] time:  10508.4 ms, score:      0 points,
[     hscan] time:    299.7 ms, score:     72 points,
[rust_regex] time:   3681.5 ms, score:     47 points,
```

## Timings
On the basis of a CSV-file I did some analytics. At first I summarized the overall execution time per engine.
The following chart shows the details:

{% include regex_compare_timing_chart.html %}

Hyperscan is the fastest engine with a total execution time of ~300ms (~3x less time than 2nd) and the Rust regex crate
gets the 5th place with ~3700ms. It seems that the Rust regex crate is not the fastest solution in place.

But: what happens if one expression is really slow? This test will distort the overall result of the engine.
Therefor I implemented a simple result scoring. For each test the fastest engine can earn 5 points, the 2nd 4 points
and so on. This limits the impact of a single slow expression. The following chart shows the score points per engine:

{% include regex_compare_score_chart.html %}

Hyperscan is still the number one, but the Rust regex crate gets the 2nd place together with PCRE2-JIT.
The results look better than the absolute timings, but it seems that one or more expressions are slow in their execution time.

So it's time to take a look into the results for each expression. The following chart compares the average timings
of all engines per expression against the measured value of the Rust regex crate.
The secondary y-axis displays the ratio of the Rust value and the average value in percentage.

{% include regex_compare_rust_chart.html %}

The red curve has 3 major peeks of expressions for which the regex crate does not perform well.
The expressions are:
1. `[a-q][^u-z]{13}x`
2. `∞|✓`
3. `(?i)Twain`

Especially the execution of the first of those three expression is done very slow.

## Improvements
With the first results of the benchmark I opened the ticket [rust-lang/regex/350](https://github.com/rust-lang/regex/issues/350)
to get feedback regarding my findings. Andrew Gallant alias [BurntSushi](https://github.com/BurntSushi)
gave me great feedback with some improvement proposals.

One improvement is to use the SIMD feature of the regex crate. This feature is currently available in the Rust nightly
built, so I had to install the nightly toolchain too. I adjusted the projects cmake scripts to detect whether a nightly
compiler is used and the SIMD feature is supported. So the rust toolchain can be switched with
`rustup default nightly-x86_64-unknown-linux-gnu` and the tool has to be reconfigured and build again to get
the new results.

{% include regex_compare_rust_simd_chart.html %}

The chart shows that the expressions `∞|✓` and `(?i)Twain` benefit by using the SIMD feature,
but not the expression `[a-q][^u-z]{13}x`. This expression requires backtracking.
The Rust regex crates uses a finite state machine based algorithm, which lacks for back-references and backtracking.

## Matches
Regarding the found matches I found some deviations. At first, the libraries oniguruma and tre do not support
Unicode categories expressions like `\p{Sm}`. This expression matches all mathematical symbols like `=` or `|`.
The Rust regex crate matches additionally the symbol `∞`.

Hyperscan returns more matches than other engines, e.g. 977 for the expression `Huck[a-zA-Z]+|Saw[a-zA-Z]+` whereas
all other engines are finding 262 matches. Hyperscan reports all matches. The expression `Saw[a-zA-Z]+` returns the
following matches for input `Sawyer`:
- Sawy
- Sawye
- Sawyer

All other engines report just one match: Sawy (non-greedy semantics) or Sawyer (greedy semantics).

# Conclusion
The Rust regex crate is now something about 2 years old, but tends to edge mature engines like PCRE2 and
Hyperscan. Depending on the used expressions the Rust regex crate is a good choice for pattern matchings.
Thanks to all contributors of the regex crate for their awesome work.

# Related work
The regex crate contains it's own benchmark harness with a lot expressions and support for:
- PCRE
- PCRE2
- RE2
- Oniguruma
- TCL

This benchmark can be used to get another view on the performance of the engines.
Please see the [bench subdirectory of the crates repository](https://github.com/rust-lang/regex/tree/master/bench).
