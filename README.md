# zerobrew

A faster homebrew.

zerobrew takes [uv](https://github.com/astral-sh/uv)'s model to Mac packages. The key design choice here is to use a content addressable store, allowing `zb` to reuse packages across installs, along with parallelizing downloads and extraction and aggressively caching HTTP requests. It interoperates with Homebrew's CDN, so you can use traditional packages. 

This leads to dramatic speedups, roughly 3x cold and 10x warm. 

## Install

```bash
curl -sSL https://raw.githubusercontent.com/lucasgelfond/zerobrew/main/install.sh | bash
```

After install, run the export command it prints, or restart your terminal.

##  Using `zb`

```bash
zb install jq        # install jq
zb install wget git  # install multiple
zb uninstall jq      # uninstall
zb reset         # uninstall everything
zb gc                # garbage collect unused store entries
```

## why is it faster?

- **Content-addressable store**: packages are stored by sha256 hash (at `/opt/zerobrew/store/{sha256}/`). Reinstalls are instant if the store entry exists.
- **APFS clonefile**: materializing from store uses copy-on-write (zero disk overhead).
- **Parallel downloads**: deduplicates in-flight requests, races across CDN connections.
- **Streaming execution**: downloads, extractions, and linking happen concurrently.


## storage layout

```
/opt/zerobrew/
├── store/          # content-addressable (sha256 keys)
├── prefix/
│   ├── Cellar/     # materialized packages
│   ├── bin/        # symlinked executables
│   └── opt/        # symlinked package directories
├── cache/          # downloaded bottle blobs
├── db/             # sqlite database
└── locks/          # per-entry file locks
```

## Build from source 

```bash
cargo build --release
cargo install --path zb_cli
```

## Benchmarking

```bash
./benchmark.sh                                # 100-package benchmark
./benchmark.sh --format html -o results.html  # html report
./benchmark.sh --format json -o results.json  # json output
./benchmark.sh -c 20 --quick                  # quick test (22 packages)
./benchmark.sh -h                             # show help
```

## Status

Experimental. works for most core homebrew packages. Some formulas may need more work - please submit issues / PRs! 

