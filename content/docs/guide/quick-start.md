+++
title = "Quick Start"
description = "Install the game framework and initialize a game template."
date = 2021-05-01T08:20:00+00:00
updated = 2021-05-01T08:20:00+00:00
draft = false
weight = 20
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "Install the game framework and initialize a game template."
toc = true
top = false
+++

## Requirements

Install the [Rust](https://www.rust-lang.org/), recommended to use [rustup](https://rustup.rs/)

## Installation

```bash
cargo install z3
```

## Generation

Next, the starter template for game development will be generated.

### Step 1: Generate template

```bash
z3 init mygame
```

### Step 2: Run project

```bash
cargo run
```

A demo game web server accessible by default at `http://127.0.0.1:9000`.



## Directory and all files

```bash
tree
```
