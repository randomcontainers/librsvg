# librsvg

Container images with `rsvg-convert` from [librsvg](https://gitlab.gnome.org/GNOME/librsvg), the SVG renderer of the GNOME project. `rsvg-convert` renders SVG files to PNG, PDF, PostScript, EPS and SVG, at the size or zoom you ask for, and lays out text with Pango and the fonts in the image. The images build `rsvg-convert` from GNOME's release tarball of each librsvg version with a pinned Rust release. They are built on Ubuntu and Alpine for `linux/amd64` and `linux/arm64`, and rebuilt when librsvg publishes a release and when the base image changes.

This is an unofficial build, not affiliated with or endorsed by the GNOME project or the librsvg maintainers. Report problems with the image in this repository and bugs in librsvg in the [librsvg issue tracker](https://gitlab.gnome.org/GNOME/librsvg/-/issues).

## Quick start

Render an SVG file as a PNG file 512 pixels wide:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/librsvg -w 512 -o icon.png icon.svg
```

The entrypoint runs `rsvg-convert` under `tini` in `/work`, so file names are relative to the directory you mount. With only `-w` or only `-h`, the other side follows the aspect ratio of the drawing. Without `-o`, the output goes to stdout. Run the image without arguments to see the options. A few more commands:

```sh
# One PDF file with a page for each SVG file
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/librsvg \
  -f pdf -o slides.pdf 1.svg 2.svg 3.svg

# Twice the natural size, on a white background instead of a transparent one
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/librsvg \
  --zoom 2 -b white -o drawing.png drawing.svg

# Only the element with id="logo", cropped to its own bounds
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/librsvg \
  -i logo -o logo.png drawing.svg

# A figure as Encapsulated PostScript
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/librsvg \
  -f eps -o figure.eps figure.svg

# Read from stdin (-) and write to stdout, without a mount
docker run --rm -i ghcr.io/randomcontainers/librsvg - < drawing.svg > drawing.png
```

`-f` takes `png` (the default), `pdf`, `pdf1.4` to `pdf1.7`, `ps`, `eps` and `svg`. With `-f pdf` or `-f ps`, each input file becomes a page; PNG and SVG output take one input file. `-h` is the height, so use `--help` or `-?` for help. `-s style.css` adds a user stylesheet, which overrides presentation attributes such as `fill="red"` but not the document's own CSS or `style` attributes, unless its rules are `!important`. `-l` picks the language for `systemLanguage` switches. Lengths in physical units such as `mm` or `in` are converted at 96 DPI; `-d` and `-p` change that. The [rsvg-convert manual](https://gitlab.gnome.org/GNOME/librsvg/-/blob/main/rsvg-convert.rst) describes every option.

## Fonts and text

`rsvg-convert` finds fonts through fontconfig, and the image has the DejaVu fonts. The generic families `sans-serif`, `serif` and `monospace` map to DejaVu Sans, DejaVu Serif and DejaVu Sans Mono, and a font that a document names but the image lacks is replaced by one of them. Text in Chinese, Japanese, Korean and other scripts that DejaVu does not cover needs more fonts. To render with other fonts, install them in an image built on `slim`, as shown under [Extending the slim image](#extending-the-slim-image).

## rsvg-convert or resvg

resvg, published by this project as `ghcr.io/randomcontainers/resvg` (see [randomcontainers/resvg](https://github.com/randomcontainers/resvg)), also renders SVG. It draws only PNG files, and its `usvg` tool writes simplified SVG. Use `rsvg-convert` for PDF, PostScript or EPS output, and when you want the same result as GNOME or pandoc, which calls `rsvg-convert` to put SVG images into PDF documents.

## What is in the image

- `rsvg-convert` in `/usr/local/bin`, built from the release tarball with cargo's release profile and the `avif` feature, as upstream's own build does when it finds dav1d. It is linked against the distro's C library and libgcc_s (glibc on Ubuntu, musl on Alpine) and the distro's cairo, Pango, GLib, HarfBuzz, FreeType, Fontconfig, libxml2 and dav1d, so their fixes reach the image when it is rebuilt (see [Updates](#updates)).
- Decoders for the images an SVG document can contain, as `data:` URLs or as files: PNG, JPEG, GIF, WebP and AVIF. A document can also include other SVG files.
- Fontconfig and the DejaVu fonts.
- The source of every Rust crate that the binary records (see [Licenses](#licenses)).

Because `rsvg-convert` needs the libraries of the image it was built in, copying `/usr/local/bin/rsvg-convert` into another image with `COPY --from` does not work.

On Alpine, librsvg refuses documents whose groups and layers are nested more than 50 levels deep; on Ubuntu the limit is 500. musl gives threads a smaller stack than glibc, and librsvg sets the lower limit to avoid running out of it. Use the Ubuntu image for such documents.

`/usr/local/share/randomcontainers/librsvg/buildinfo` has the tarball's SHA-256, the Rust release with the URL and SHA-256 of each archive it was installed from, the output of `rustc -vV`, the cargo-auditable version, the output of `rsvg-convert --version` with the versions of cairo, Pango, HarfBuzz and Fontconfig, and every crate the binary was built from, with its version, whether it is linked in or used only during the build, and its license.

Not included: the librsvg C library and its GObject introspection data, the gdk-pixbuf loader, the man page and the shell completions (`rsvg-convert --completion bash` prints those for bash).

## Default or slim

librsvg's default image adds no other tools, so `latest` and `slim` are the same image, with the contents listed above. Use `latest` to run it and the `slim` tags as a base for your own image.

## Tags

`<version>` is a librsvg release such as `2.63.3`. `<minor>` and `<major>` are its shorter forms, `2.63` and `2`, and follow the newest release in that series. Each row lists the default tag and its `slim` twin, which point to the same image.

| Tags | Base |
|---|---|
| `latest`, `slim` | Ubuntu |
| `<version>`, `<version>-slim` | Ubuntu |
| `<minor>`, `<minor>-slim`, `<major>`, `<major>-slim` | Ubuntu |
| `ubuntu`, `slim-ubuntu` | Ubuntu |
| `<version>-ubuntu`, `<version>-slim-ubuntu` | Ubuntu |
| `<minor>-ubuntu`, `<minor>-slim-ubuntu`, `<major>-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04`, `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `slim-alpine` | Alpine |
| `<version>-alpine`, `<version>-slim-alpine` | Alpine |
| `<minor>-alpine`, `<minor>-slim-alpine`, `<major>-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24`, `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current librsvg version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Each image contains `rsvg-convert` built for its own architecture and is built natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. How to get output files owned by you depends on how you run containers:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

`rsvg-convert -o` writes into the mounted directory, so it must be writable. Without `-o`, the output goes to stdout and the mount can be read-only (`-v "$PWD:/work:ro"`). With `--read-only`, add `--tmpfs /cache`, so that fontconfig can write its cache if it needs to.

## Untrusted files

An SVG document can refer to other files: images, other SVG documents, stylesheets and text included with XInclude. `rsvg-convert` loads such a file only when it is in the directory of the document or below it, and it never fetches `http:` or `https:` URLs. A file it loads can end up in the output, so a document from someone else can copy any file next to it into the image you produce. A document read from stdin has no directory, and `rsvg-convert` then loads nothing but `data:` URLs.

librsvg limits the number of elements and references in a document, but not the memory or time a document takes, and its developers advise a sandbox for untrusted input. Do not pass `--unlimited`, which lifts the limits on text size in the XML parser and on the size of decoded images. For documents from unknown sources, pass the document on stdin, mount nothing and take away what the container does not need:

```sh
docker run --rm -i \
  --network none --read-only --tmpfs /cache --cap-drop ALL --security-opt no-new-privileges \
  --memory 1g --cpus 1 --pids-limit 64 \
  ghcr.io/randomcontainers/librsvg - < untrusted.svg > untrusted.png
```

Stop jobs that run too long, for example with `timeout 60 docker run ...`. librsvg lists the releases with security fixes on its [security page](https://gnome.pages.gitlab.gnome.org/librsvg/devel-docs/security.html), and the images pick up a new release about a day after it is published.

## Extending the slim image

Use a `slim` tag as the base for your own image. `slim`, `slim-ubuntu` and `slim-alpine` move to each new librsvg release and are rebuilt when the base image changes. Switch to root to install more, then back. This adds the Noto fonts, which cover most scripts:

```dockerfile
FROM ghcr.io/randomcontainers/librsvg:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends fonts-noto-core fonts-noto-cjk \
 && rm -rf /var/lib/apt/lists/*
USER 1000:1000
```

`rsvg-convert` finds fonts installed this way through fontconfig. On Alpine, start from `slim-alpine` and use `apk add --no-cache font-noto font-noto-cjk`. The entrypoint is `["tini", "--", "rsvg-convert"]`; set your own `ENTRYPOINT` if your image runs something else. To pick up new librsvg releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

Everything the image adds is under `/usr/local`. `/usr/local/share/randomcontainers/librsvg/` holds the version, the source URLs, the build details, the license files, the crate sources and `runtime-deps`, the list of distro packages `rsvg-convert` needs at run time.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/librsvg:latest \
  --repo randomcontainers/librsvg --signer-repo randomcontainers/ci
```

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/librsvg:latest --format '{{ json .SBOM }}'
```

`rsvg-convert` is built with [cargo-auditable](https://github.com/rust-secure-code/cargo-auditable), which stores the list of crates compiled into it in the binary's `.dep-v0` section. Syft, Grype and Trivy read that list, so SBOM tools and image scanners see those crates. The Rust standard library is not in the list, and scanners do not report its advisories; `buildinfo` names the Rust release that built `rsvg-convert`, and [Updates](#updates) says when it moves.

GNOME publishes a `.sha256sum` file next to each tarball but no signatures. Before building, the build checks the librsvg tarball against the SHA-256 recorded in `package.yml`. cargo downloads the crates in the tarball's `Cargo.lock` and checks each against the checksum there, and the build that follows has no network access. It also stops if a release adds a cargo configuration file, which would change how it is built.

Rust comes from static.rust-lang.org. The release manifest named in `package.yml` is checked against the SHA-256 recorded there and against its signature, made with the Rust release key (fingerprint `108F 6620 5EAE B0AA A8DD 5E1C 85AB 96E6 FA1B E5FE`, kept in [keys/rust.gpg](keys/rust.gpg)). Each archive installed from it is checked against the SHA-256 that the manifest lists. cargo-auditable, which runs only during the build, is compiled from its crates.io crate, checked against the SHA-256 in `package.yml`, with the crates listed in its own `Cargo.lock`.

## Updates

The project checks the [releases of GNOME/librsvg](https://gitlab.gnome.org/GNOME/librsvg/-/releases) on GNOME's GitLab every 15 minutes. A release is picked up once it is 24 hours old. Its tarball is checked against the `librsvg-<version>.sha256sum` file on download.gnome.org, the new version and the tarball's SHA-256 are committed to `package.yml`, and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built, and fixes released only for an older series are not built. Versions with a patch number of 90 or more, such as `2.62.91`, are betas and are skipped.

The Rust release that builds `rsvg-convert` follows Rust's stable channel. Each new stable release is committed to `package.yml` once it is 7 days old, and the images are rebuilt with it. cargo-auditable is pinned by hand. A librsvg release that needs a newer Rust than the pinned one does not build until the pin moves. A release also waits for a person when it adds a crate under a license that the image's label does not cover, adds a crate without a license file, adds a crate that builds or links native code, changes the license of selectors or mutants, stops using selectors, mutants, rayon-core or yeslogic-fontconfig-sys, for which this repository keeps license texts or a note (see [licenses](licenses)), or adds a cargo configuration file.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes and at least every 7 days, so distro security fixes, including those to cairo, Pango, libxml2 and the other libraries `rsvg-convert` loads, reach the current tags.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_SHA256=<sha256 from package.yml> \
  --build-arg RUST_TOOLCHAIN_VERSION=<rust-toolchain version from package.yml> \
  --build-arg RUST_TOOLCHAIN_SHA256=<rust-toolchain sha256 from package.yml> \
  --build-arg CARGO_AUDITABLE_VERSION=<cargo-auditable version from package.yml> \
  --build-arg CARGO_AUDITABLE_SHA256=<cargo-auditable sha256 from package.yml> \
  -t librsvg:local .
```

Use `Dockerfile.alpine`, with the same arguments, for the Alpine image. The build downloads 115 to 135 MB of Rust archives on Ubuntu and about 170 MB on Alpine, and the crates in `Cargo.lock`. `--build-arg JOBS=<n>` limits the number of crates cargo compiles in parallel.

## Licenses

librsvg is licensed under the GNU Lesser General Public License, version 2.1 or later (LGPL-2.1-or-later), and so is `rsvg-convert`. The binary also contains the Rust crates librsvg depends on. Most of them are under `MIT OR Apache-2.0` or MIT, and the others offer a choice of licenses that includes MIT, except these:

- Apache-2.0: nalgebra, simba and approx, which librsvg uses for matrix math, and moxcms, the color management crate of the image decoders, and pxfm, the math crate it uses, which offer `BSD-3-Clause OR Apache-2.0`.
- MPL-2.0: cssparser, cssparser-color, selectors and dtoa-short, which librsvg's CSS code uses, and mp4parse, which reads AVIF files.
- BSD-3-Clause: the WHATWG data in encoding_rs.
- Unicode-3.0: the ICU4X crates, which the url crate uses for international domain names.
- Zlib: zlib-rs, the zlib implementation that the flate2 crate uses.

The Rust standard library and the crates it uses are linked in too. They are under `MIT OR Apache-2.0` or another choice that includes MIT, except compiler-builtins, which is under `MIT AND Apache-2.0 WITH LLVM-exception`. The image's license label is `LGPL-2.1-or-later AND MPL-2.0 AND MIT AND Apache-2.0 AND BSD-3-Clause AND ISC AND Unicode-3.0 AND Zlib AND Apache-2.0 WITH LLVM-exception`. The label has ISC for libloading. The crate list that cargo-auditable stores in the binary names libloading, dlib and yeslogic-fontconfig-sys (both MIT), because a dev-dependency of rsvg_convert turns on librsvg's `test-utils` feature and `cargo metadata`, which cargo-auditable reads, counts that feature as enabled. None of the three is compiled into `rsvg-convert`, but scanners report them, so the label covers them. The build stops when a crate linked into `rsvg-convert` has a license that this label does not cover. The Ubuntu and Alpine packages in the image keep their own licenses, for example LGPL for cairo, Pango and GLib, BSD-2-Clause for dav1d and the DejaVu font license for the fonts. The SBOM lists them.

The license files are in `/usr/local/share/randomcontainers/librsvg/licenses/`: librsvg's `COPYING.LIB` from the tarball; `workspace/` with the same file from each librsvg crate compiled in; `crates/<crate>-<version>/` with the license files of every crate that the binary records, including those used only while it is built, and `crates/CRATES.tsv`, which lists each of them with its license; and `rust/` with every license file in the `rust-src` archive of the Rust release that built `rsvg-convert`, which covers rust-lang/rust, the standard library and the crates it vendors. Two crates publish no license file: selectors, whose source files carry the MPL-2.0 notice, and mutants (MIT), which is used only to build. This repository keeps their license texts in [licenses](licenses), and the build copies them into the image.

`rsvg-convert` links its Rust crates statically, so its corresponding source includes the source of each of them. The source for each image:

- librsvg: every version has a GitHub release in this repository, named `v<version>`, with the exact `librsvg-<version>.tar.xz` that was built, the Rust release manifest with its signature, and the cargo-auditable crate. The build applies no patches.
- Rust crates: the `.crate` file of every crate that the binary records, in `/usr/local/share/randomcontainers/librsvg/crate-sources/`, as cargo downloaded it and checked it against the tarball's `Cargo.lock`. `/usr/local/share/randomcontainers/librsvg/source` lists the tarball, the Rust release manifest and its rust-src archive, the cargo-auditable crate, the crates.io URL of each of these crates and the release.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

The files in this repository are available under the MIT license, see [LICENSE](LICENSE), except the license texts in `licenses/`, which keep their own terms, and the base64-encoded AVIF image in the fifth `test` entry of `package.yml`, which is `rsvg/tests/fixtures/reftests/rectangle.avif` from librsvg's test suite and is under LGPL-2.1-or-later, see [licenses/LGPL-2.1](licenses/LGPL-2.1).

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
