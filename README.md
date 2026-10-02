# audiowaveform

Container images with [audiowaveform](https://codeberg.org/chrisn/audiowaveform), the command-line tool from BBC Research & Development that generates waveform data and waveform images from audio files. It reads MP3, WAV, FLAC, Ogg Vorbis and Opus files and writes the binary `.dat` and JSON formats that [Peaks.js](https://codeberg.org/chrisn/peaks.js) displays, or PNG images. The default image also contains FFmpeg, for AAC, M4A and video files. The images are rebuilt when audiowaveform publishes a release and when the base image changes, for `linux/amd64` and `linux/arm64`.

This is an unofficial build, not affiliated with or endorsed by the BBC or the audiowaveform project. Report problems with the image in this repository and problems with audiowaveform itself [upstream on Codeberg](https://codeberg.org/chrisn/audiowaveform/issues). audiowaveform is developed on Codeberg; the [bbc/audiowaveform](https://github.com/bbc/audiowaveform) repository on GitHub stops at version 1.10.3.

## Quick start

Write waveform data for a file in the current directory:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/audiowaveform -i input.mp3 -o input.dat -z 256 -b 8
```

audiowaveform mixes the channels down to one, then stores the minimum and maximum sample of every 256 samples (`-z`, the zoom level) as one point. `-b 8` stores each value in 8 bits instead of 16, which halves the data. The output format follows the file extension: `.dat`, `.json`, `.png`, or `.wav` to convert the audio itself. `--pixels-per-second` sets the zoom by time instead of by samples, and `--split-channels` keeps one waveform per channel:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/audiowaveform -i input.wav -o input.json --pixels-per-second 20 -b 8
```

PNG images are rendered from audio or from a `.dat` or `.json` file. `-z auto` fits the whole file into the width, and `-s` and `-e` select a range in seconds:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/audiowaveform -i input.flac -o input.png -z auto -w 1800 -h 280 --no-axis-labels
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/audiowaveform -i input.dat -o chorus.png -s 45 -e 60 -w 1000 -h 200 \
  --waveform-style bars --bar-width 6 --bar-gap 2
```

`--help` lists the color, bar and axis options. Upstream's [README](https://codeberg.org/chrisn/audiowaveform/src/branch/master/README.md) describes every option, and [doc/DataFormat.md](https://codeberg.org/chrisn/audiowaveform/src/branch/master/doc/DataFormat.md) the `.dat` and JSON formats.

## Other formats and pipes

audiowaveform reads MP3 with libmad, and WAV, FLAC, Ogg Vorbis, Opus and raw PCM with libsndfile. Without `-i`, or with `-i -`, it reads stdin in the format given by `--input-format`; that works for MP3, WAV and raw audio, not for FLAC, Ogg Vorbis or Opus. Without `-o`, it writes to stdout in the format given by `--output-format`. Raw audio needs its layout on the command line:

```sh
docker run --rm -i ghcr.io/randomcontainers/audiowaveform --input-format raw \
  --raw-samplerate 48000 --raw-channels 2 --raw-format s16le \
  --output-format json -z 480 -b 8 < input.raw > input.json
```

For AAC, M4A, video files and anything else libsndfile cannot read, the default image has FFmpeg. Convert the audio to a WAV stream and pipe it to audiowaveform:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint sh \
  ghcr.io/randomcontainers/audiowaveform -c \
  'ffmpeg -loglevel error -i input.m4a -f wav - | audiowaveform --input-format wav --output-format dat -z 256 -b 8 > input.dat'
```

## What is in the image

| Area | Details |
|---|---|
| Tool | `audiowaveform` |
| Audio input | MP3 (libmad, with ID3 tags skipped by libid3tag), WAV, FLAC, Ogg Vorbis, Opus and raw PCM (libsndfile) |
| Output | Waveform data as `.dat` (format version 1, or 2 when `--split-channels` writes several channels) and JSON, PNG images (libgd, with its built-in font for the axis labels), and WAV |
| Default image | FFmpeg (`ffmpeg` and `ffprobe`), for AAC, M4A, video files and other formats audiowaveform cannot read |

Not included: upstream's unit tests and the man pages. The commit and the fingerprint of the key that signed it are in `/usr/local/share/randomcontainers/audiowaveform/buildinfo`.

## Default or slim

Use the default image (`latest`) for general work. It adds FFmpeg, so waveforms of AAC, M4A and video files are made in one container (see [Other formats and pipes](#other-formats-and-pipes)). `slim` has audiowaveform and the libraries it needs, without FFmpeg. Use it to build your own image, or when your audio is already MP3, WAV, FLAC, Ogg Vorbis or Opus.

The default image is also published as `ghcr.io/randomcontainers/audiowaveform-ffmpeg`, built in the [audiowaveform-ffmpeg](https://github.com/randomcontainers/audiowaveform-ffmpeg) repository with the same contents and a different digest.

## Tags

`<version>` is an audiowaveform release such as `1.11.1`. `<minor>` and `<major>` are its shorter forms, `1.11` and `1`, and follow the newest release in that series.

| Default (with FFmpeg) | Slim | Base |
|---|---|---|
| `latest`, `<version>`, `<minor>`, `<major>` | `slim`, `<version>-slim`, `<minor>-slim`, `<major>-slim` | Ubuntu |
| `ubuntu`, `<version>-ubuntu`, `<minor>-ubuntu`, `<major>-ubuntu` | `slim-ubuntu`, `<version>-slim-ubuntu`, `<minor>-slim-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04` | `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `<version>-alpine`, `<minor>-alpine`, `<major>-alpine` | `slim-alpine`, `<version>-slim-alpine`, `<minor>-slim-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24` | `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current audiowaveform version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Both are compiled natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. How to get output files owned by you depends on how you run containers:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

## Untrusted files

audiowaveform decodes audio with libmad, libid3tag and libsndfile, C libraries from Ubuntu or Alpine whose security fixes reach the images through the rebuilds described in [Updates](#updates). It does not use the network. For uploads and other files from unknown sources, mount the input read-only, write to a separate directory and take away what the container does not need:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD/uploads:/in:ro" -v "$PWD/out:/work" \
  --network none --read-only --cap-drop ALL --security-opt no-new-privileges \
  --memory 512m --pids-limit 16 \
  ghcr.io/randomcontainers/audiowaveform -i /in/upload.mp3 -o upload.dat -z 256 -b 8
```

## Extending the slim image

Use a `slim` tag as the base for your own image. It has no FFmpeg, so FFmpeg updates do not rebuild it. The packages audiowaveform needs are listed in `/usr/local/share/randomcontainers/audiowaveform/runtime-deps`. Switch to root to install more, then back:

```dockerfile
FROM ghcr.io/randomcontainers/audiowaveform:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends jq \
 && rm -rf /var/lib/apt/lists/*
USER 1000:1000
```

On Alpine, use `apk add --no-cache jq`. jq reads the JSON waveform data. The entrypoint is `["tini", "--", "audiowaveform"]`; set your own `ENTRYPOINT` if your image runs something else. To pick up new audiowaveform releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/audiowaveform:latest \
  --repo randomcontainers/audiowaveform --signer-repo randomcontainers/ci
```

Images from `ghcr.io/randomcontainers/audiowaveform-ffmpeg` are built in that repository, so verify them with `--repo randomcontainers/audiowaveform-ffmpeg` and the same `--signer-repo`.

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/audiowaveform:latest --format '{{ json .SBOM }}'
```

audiowaveform publishes no release tarballs, and its release tags are lightweight, so each version is pinned to the commit of its tag. The release commits are signed. The build clones the tag from [chrisn/audiowaveform](https://codeberg.org/chrisn/audiowaveform), stops unless it points to the commit recorded in `package.yml`, and checks the commit's signature against Chris Needham's key in `keys/audiowaveform-release.gpg` (fingerprint `48A6 2A0D 6B74 FF7C 78EF C8A2 B323 37F0 1336 8EC0`), the key that his [Codeberg](https://codeberg.org/chrisn.gpg) and [GitHub](https://github.com/chrisn.gpg) accounts publish.

## Updates

The project checks the tags of [chrisn/audiowaveform](https://codeberg.org/chrisn/audiowaveform/tags) on Codeberg every 15 minutes. A release is picked up once its tag is 24 hours old. The new version and the commit of its tag are then committed to `package.yml` and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes, the default ones when a new FFmpeg image is published, and all of them at least every 7 days, so distro security fixes reach the current tags.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_COMMIT=<commit from package.yml> \
  -t audiowaveform:local .
```

Use `Dockerfile.alpine` for the Alpine image. `--build-arg JOBS=<n>` limits the number of parallel compile jobs. The default image is generated from the `combos` entry in `package.yml` by [randomcontainers/ci](https://github.com/randomcontainers/ci).

## Licenses

audiowaveform is licensed under the GNU General Public License, version 3 or later (GPL-3.0-or-later). Parts of its waveform code are adapted from Audacity and are covered by that license. Its MP3 reader contains code adapted from madlld and reads files through madlld's `bstdfile.c`; the madlld code is under BSD-3-Clause. It also contains the pdjson JSON parser (public domain, SPDX `Unlicense`). The madlld and pdjson license texts are in `/usr/local/share/randomcontainers/audiowaveform/licenses/`. libmad and libid3tag (both GPL-2.0-or-later), libsndfile, libgd, Boost and the other libraries from Ubuntu or Alpine keep their own licenses; the SBOM lists them.

The corresponding source for each image:

- audiowaveform: every version has a GitHub release in this repository, named `v<version>`, with `audiowaveform-<version>.tar.gz`, a `git archive` of the commit that was compiled. `/usr/local/share/randomcontainers/audiowaveform/source` lists that release and the commit on Codeberg.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label. The Dockerfiles hold every build option.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

The default image adds FFmpeg, licensed under GPL-3.0-or-later; its license files and corresponding source are described in the [ffmpeg](https://github.com/randomcontainers/ffmpeg#licenses) repository.

The files in this repository are available under the MIT license, see [LICENSE](LICENSE).

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
