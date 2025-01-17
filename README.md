# rust-cached-path

[![crates.io](https://img.shields.io/crates/v/cached-path.svg)](https://crates.io/crates/cached-path)
[![Documentation](https://docs.rs/cached-path/badge.svg)](https://docs.rs/cached-path)
[![MIT/Apache-2 licensed](https://img.shields.io/crates/l/cached-path.svg)](./LICENSE)
[![CI](https://github.com/epwalsh/rust-cached-path/workflows/CI/badge.svg)](https://github.com/epwalsh/rust-cached-path/actions?query=workflow%3ACI)

The idea behind `cached-path` is to provide a unified, simple interface for
accessing both local and remote files. This can be used behind other APIs that need
to access files agnostic to where they are located.

This is based on
[`allennlp/common/file_utils.py`](https://github.com/allenai/allennlp/blob/master/allennlp/common/file_utils.py)
and
[`transformers/file_utils.py`](https://github.com/huggingface/transformers/blob/master/src/transformers/file_utils.py).

## Installation

`cached-path` can be used as both a library and a command-line tool. To install `cached-path`
as a command-line tool, run

```bash
cargo install --features build-binary cached-path
```

## Usage

For remote resources, `cached-path` downloads and caches the resource, using the ETAG
to know when to update the cache. The path returned is the local path to the latest
cached version:

```rust
use cached_path::cached_path;

let path =
cached_path("https://github.com/epwalsh/rust-cached-path/blob/master/README.md").unwrap();
assert!(path.is_file());
```

```bash
# From the command line:
$ cached-path https://github.com/epwalsh/rust-cached-path/blob/master/README.md
/tmp/cache/055968a99316f3a42e7bcff61d3f590227dd7b03d17e09c41282def7c622ba0f.efa33e7f611ef2d163fea874ce614bb6fa5ab2a9d39d5047425e39ebe59fe782
```

For local files, the path returned is just the original path supplied:

```rust
use cached_path::cached_path;

let path = cached_path("README.md").unwrap();
assert_eq!(path.to_str().unwrap(), "README.md");
```

```bash
# From the command line:
$ cached-path https://github.com/epwalsh/rust-cached-path/blob/master/README.md
README.md
```

It's easy to customize the cache location, the HTTP client, and other options
using a [`CacheBuilder`](https://docs.rs/cached-path/*/cached_path/struct.CacheBuilder.html) to construct a custom
[`Cache`](https://docs.rs/cached-path/*/cached_path/struct.Cache.html) object. This is also the recommend thing
to do if your application makes multiple calls to `cached_path`, since it avoids the overhead
of creating a new HTTP client on each call:

```rust
use cached_path::Cache;

let cache = Cache::builder()
    .dir(std::env::temp_dir().join("my-cache/"))
    .connect_timeout(std::time::Duration::from_secs(3))
    .build().unwrap();
let path = cache.cached_path("README.md").unwrap();
```

```bash
# From the command line:
$ cached-path --dir /tmp/my-cache/ --connect-timeout 3 README.md
README.md
```
