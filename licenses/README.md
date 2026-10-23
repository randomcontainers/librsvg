# Extra license files

`LGPL-2.1` is the license of the AVIF image embedded in the fifth `test` entry of `package.yml`, which comes from librsvg's test suite. The container image gets the same text as librsvg's `COPYING.LIB`, copied from the release tarball, so the build does not use this file.

Two crates that `rsvg-convert` is built from publish no license text. Each `crates/<crate>/` directory here holds that text, and `scripts/cargo-licenses.py` adds it to the crate's own files in the image. The directories carry no version, so they are kept across versions; the `EXPRESSION` file in each holds the crate's license expression, and the build stops when the crate's expression changes or when the binary no longer records the crate.

| Files | Source |
|---|---|
| `LGPL-2.1` | `COPYING.LIB` of the librsvg 2.63.3 release tarball, unchanged: the GNU Lesser General Public License, version 2.1 |
| `crates/selectors/LICENSE` | the text of the [Mozilla Public License 2.0](https://www.mozilla.org/media/MPL/2.0/index.txt). selectors is developed in [servo/stylo](https://github.com/servo/stylo), which has no license file; each of its source files carries the MPL-2.0 notice |
| `crates/mutants/LICENSE` | `LICENSE` of [sourcefrog/cargo-mutants](https://github.com/sourcefrog/cargo-mutants), where the crate lives in `mutants_attrs/`, unchanged since it was added in 2021 |
| `crates/native-covered` | written for this repository: rayon-core and yeslogic-fontconfig-sys declare a `links` key, which the build treats as native code. rayon-core's build script compiles nothing, and yeslogic-fontconfig-sys, which the binary's crate list names because of a dev-dependency feature, is not compiled into `rsvg-convert` |
