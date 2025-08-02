---
title: Malicious aspect of cargo build scripts
description: Cargo build scripts can be easily leveraged to perform supply chain attack
date: 2025-02-10
draft: false
tags: [rust malware]
---

## Cargo build scripts
I first learned about Cargo build scripts at a local Rust meetup. There, a presenter demonstrated how they can automate various tasks. It quickly became clear how easy it is to insert a malicious build.rs into a single dependency, especially when a project has hundreds of them.
#### Whats are the cargo build scripts?
It's a Rust code that is compiled and executed when the package or main code is built. It's that simple. Build scripts can be used for various reasons. For example, they can execute pre-building scripts that adjust the build. These scripts can perform platform-specific configurations needed for the build or one of its components.

The cargo documentation provides a simple example of build script code.
```
// build.rs

use std::env;
use std::fs;
use std::path::Path;

fn main() {
    let out_dir = env::var_os("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("hello.rs");
    fs::write(
        &dest_path,
        "pub fn message() -> &'static str {
            \"Hello, World!\"
        }
        "
    ).unwrap();
    println!("cargo::rerun-if-changed=build.rs");
}
```

This code reads an environment variable and creates a new text file in a new directory. Additionally, the cargo format is used to print information to stdout. Thus, the code above is a simple example of the impact on the operating system; the file is created with content.

Scripts have a wide range of capabilities, such as operating files (also outside the OUT_DIR scope) and network operations.

In the cargo documentation we can also see a line, so by default nothing will appear in our cargo stdout logs.
```
The output of the script is hidden from the terminal during normal compilation
```
it makes easier to not to draw attention of our executed code.

#### So whats the risk?
It is reasonable to assume that not all included libraries have been thoroughly analyzed for security vulnerabilities in projects. Sometimes, only the number of CVEs submitted for a certain version are checked. For example, dependency check is the only way to prove that the code is not vulnerable; however, it still has the potential to include malicious code.

Currently, there is talk about sandboxing those executions (https://github.com/rust-lang/cargo/issues/5720). In my opinion, it is the only direct way to increase security and reduce potential risks. This issue is still open on GitHub, and there are no pre-production implementations ready at the time of writing this article.
#### What is the potential workaround now?
One solution would probably be to keep as much as possible in VMs. All of the code that I develop is coded, tested, and executed in virtual machines. This rule has another advantage: you keep your host clean, with no dependency issues or risk of malicious code. Not everything can be done in VMs, but most things can.

But what if you're developing huge code for a large-scale product? An interesting idea is short-lived instances in your infrastructure inspired by the AWS Spot Instances concept. These instances would only have one task: to perform a job and then terminate. This decreases the level of persistence of potential malware, for example. It's not a perfect solution, but it could be quite effective.