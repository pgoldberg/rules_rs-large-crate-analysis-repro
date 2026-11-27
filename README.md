# rules_rs repro: analysis is O(n^2) in a crate's file count

`rquickjs-sys` from git has about 52,000 files. `rust_crate` globs all of them into `compile_data` and passes that list to `rust_library` and `rust_binary` (for its build script). For a target declared in a symbolic macro, Bazel's visibility check compares each of the target's dependencies against every label the macro was called with, so analyzing `rquickjs-sys` does billions of `Label.equals` calls.

The first build fetches rquickjs and its submodules:

```
$ bazel build --nobuild //:repro
```

Then, with a fresh server so nothing is cached in memory:

```
$ bazel shutdown && bazel build --nobuild //:repro
INFO: Analyzed target //:repro (204 packages loaded, 80872 targets configured, 12 aspect applications).
INFO: Elapsed time: 96.644s, Critical Path: 0.00s
```

With `compile_data` behind a `filegroup` in a local checkout of rules_rs:

```
$ bazel shutdown && bazel build --nobuild --lockfile_mode=off --override_module=rules_rs=$HOME/rules_rs //:repro
INFO: Analyzed target //:repro (204 packages loaded, 80881 targets configured, 12 aspect applications).
INFO: Elapsed time: 21.472s, Critical Path: 0.00s
```
