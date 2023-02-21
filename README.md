<!--
================= AUTOGENERATED FILE =================
================= DO NOT EDIT THIS   =================

If you want to edit this, then change doc/template.README.md instead.

================= DO NOT EDIT THIS   =================
================= AUTOGENERATED FILE =================
-->

# protoCURL

[![Tests Status](https://github.com/qaware/protocurl/actions/workflows/test.yml/badge.svg)](https://github.com/qaware/protocurl/actions/workflows/test.yml)
[![GitHub Release (latest SemVer)](https://img.shields.io/github/v/release/qaware/protocurl?label=Release&logoColor=white&logo=GitHub&sort=semver)](https://github.com/qaware/protocurl/releases)
[![DockerHub Version (latest semver)](https://img.shields.io/docker/v/qaware/protocurl?label=Docker&logo=Docker&logoColor=white&sort=semver)](https://hub.docker.com/r/qaware/protocurl/tags)

[![Pull Requests](https://img.shields.io/github/issues-pr/qaware/protocurl?color=lightgreen&label=Pull%20Requests)](https://github.com/qaware/protocurl/pulls)
[![Pull Requests (closed)](https://img.shields.io/github/issues-pr-closed/qaware/protocurl?color=blue&label=Pull%20Requests)](https://github.com/qaware/protocurl/pulls?q=is%3Apr+is%3Aclosed)

protoCURL is cURL for Protobuf: The command-line tool for interacting with Protobuf over HTTP REST endpoints using
human-readable text formats.

## Why?

Every data interchange format, which is used between service boundaries, benefits from a command-line-tool enabling one to directly talk to a service. We do not want to use a full-blown programming language and all its libraries just for quick and simple requests. Such a tool is very useful during development and debugging.

Text-based formats, like JSON and XML, can simply be used with tools like [curl](https://curl.se/) as the requests can be written by hand. However, since Protobuf is a binary-encoded format, writing these requests by hand is hard and cumbersome.

Hence, **protoCURL**.

protoCURL enables us to write requests in a human-readable textual-format while talking to a binary-encoded Protobuf over HTTP REST endpoint.

Without protoCURL, one would either need to duplicate the HTTP REST endpoints returning equivalent JSON/XML responses - or one needs to write custom code and run it from the IDE just to send simple requests. The first option is technical debt at its finest - and the second one is less ergonomic than using protoCURL.

For an introduction with an example: [Read the intro Blogpost](https://blog.qaware.de/posts/protocurl-intro)

## Install

`protocurl` includes and uses a bundled `protoc` by default. It is recommended to install `curl` into PATH for
configurable http requests. Otherwise `protocurl` will use a simple non-configurable fallback http implementation.

`protocurl` uses [semantic versioning](https://semver.org/spec/v2.0.0.html) when new releases are versioned.

### Native CLI

**Archive**

1. Download the latest release archive for your platform from https://github.com/qaware/protocurl/releases
2. Extract the archive into a folder, e.g. `/opt/protocurl`.
3. Add symlink to the binary in the folder. e.g. `ln -s /opt/protocurl/bin/protocurl /usr/bin/protocurl`
   Or add the binary folder `/opt/protocurl/bin` to your system-wide path.
4. Test it via `protocurl -h`

**Debian .deb package**

1. Download the latest release `.deb` for your architecture from https://github.com/qaware/protocurl/releases
2. Install dependency curl: `sudo apt install curl`
3. Install `sudo dpkg -i <downloaded-release>.deb`
4. Test it via `protocurl -h`

**Alpine .apk package**

1. Download the latest release `.apk` for your architecture from https://github.com/qaware/protocurl/releases
2. Install dependencies curl and gcompat: `sudo apk add curl gcompat`
3. Install `sudo apk add --allow-untrusted <downloaded-release>.apk`
4. Test it via `protocurl -h`

### Docker

Simply run `docker run -v "/path/to/proto/files:/proto" qaware/protocurl <args>`. See [Quick Start](#quick-start) for
how to use.

## Quick Start

After installing `protocurl` a request is as simple as:

```bash
protocurl -I test/proto -i ..HappyDayRequest -o ..HappyDayResponse \
  -u http://localhost:8080/happy-day/verify -d "includeReason: true"
```

where

* `-I test/proto` points to the directory of protobuf files of your service
    * with docker one needs to instead mount the directory to `/proto` via `-v $PWD/test/proto:/proto`
* `-i ..HappyDayRequest` and `-o ..HappyDayResponse` are Protobuf message types. The `..` makes protocurl infer their full package paths.
* `-u http://localhost:8080/happy-day/verify` is the url to the HTTP REST endpoint accepting and returning binary protobuf
  payloads
    * with docker one may additionally need `--network host`
* `-d "includeReason: true"` is the protobuf payload in Protobuf [Text](#protobuf-text-format)
  or [JSON](#protobuf-json-format) Format

Then protocurl will

* encode the textual Protobuf message to a binary request payload
* send the binary request to the HTTP REST endpoint (via `curl`, if possible) and receive the binary response payload
* decode the binary response payload back to text and display it

and produce the following output:

```
=========================== Request Text     =========================== >>>
includeReason: true
=========================== Response Text    =========================== <<<
isHappyDay: true
reason: "Thursday is a Happy Day! ⭐"
formattedDate: "Thu, 01 Jan 1970 00:00:00 GMT"
```

See below for usage notes and [EXAMPLES.md](EXAMPLES.md) for more information.

## Usage and Example

See [usage notes](doc/generated.usage.txt), [EXAMPLES.md](EXAMPLES.md) as well as the [intro Blogpost](https://blog.qaware.de/posts/protocurl-intro).

## Protobuf JSON Format

protoCURL supports the [Protobuf JSON Format](https://protobuf.dev/programming-guides/proto3/#json). Note,
that the JSON format is not a straightforward 1:1 mapping as it is in the case of the Protobuf Text Format (described
below). For instance,
the [JSON mapping for timestamp.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto)
uses a human-readable string representation, whereas the payload itself and the text format use a representation with
seconds and nanoseconds.

## Protobuf Text Format

Aside from JSON, Protobuf primarily and natively supports a text format which represents the fields 1:1 like in the
wire-format. For instance, repeated fields are condensed into an array in the JSON format - whereas they are simply '
repeated' without an array type in the text format.

For instance, for the following .proto file

```
syntax = "proto3";

import "google/protobuf/timestamp.proto";

message HappyDayRequest {
  google.protobuf.Timestamp date = 1;
  bool includeReason = 2;

  double myDouble = 3;
  int64 myInt64 = 5;
  repeated string myString = 6;
  repeated NestedMessage messages = 9;
}

message NestedMessage {
  Foo fooEnum = 1;
  repeated int32 i = 4;
}

enum Foo {
  BAR = 0;
  BAZ = 1;
}
```

a `HappyDayRequest` message in text format might look like this:

```
includeReason: true,
myInt64: 123123123123,
myString: "hello world"
myString: 'single quotes are also possible'
myDouble: 123.456
messages: { fooEnum: BAR, i: 0, i: 1, i: 1337 },
messages: { i: 15, fooEnum: BAZ, i: -1337 },
messages: { },
date: { seconds: 123, nanos: 321 }
```

In summary:

- No encapsulating `{ ... }` are used for the top level message (in contrast to JSON).
- fields are comma separated and described via `<fieldname>: <value>`.
    - Strictly speaking, the commas are optional and whitespace is sufficient
- repeated fields are simply repeated multiple times (instead of using an array) and they do not need to appear
  consecutively.
- nested messages are described with `{ ... }` opening a new context and describing their fields recursively
- scalar values are describes similar to JSON. Single and double quotes are both possible for strings.
- enum values are referenced by their name
- built-in messages (such
  as [google.protobuf.Timestamp](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)
  are described just like user-defined custom messages via `{ ... }` and their message fields

The text format is defined in the [Protobuf: Text Format Language Specification](https://protobuf.dev/reference/protobuf/textformat-spec/).

## Maintainer

The project was created and is currently maintained by [GollyTicker](https://github.com/GollyTicker).

[qaware](https://github.com/qaware/) provided initial sponsorship for the project.

## Development

For development it is recommended to use the a bash-like Terminal either natively (Linux, Mac) or via MinGW on Windows.

About the CI/CD tests: [TESTS.md](TESTS.md)

How to make a release: [RELEASE.md](RELEASE.md)

#### Setup

- As for script utilities, one needs `bash`, `jq`, `zip`, `unzip` and `curl`.
- One also needs to download the protoc binaries for the local development via `release/10-get-protoc-binaries.sh`.

For development the `generated-local.Dockerfile` (via [generate-local.Dockerfile.sh](dev/generate-local.Dockerfile.sh)) is used.
To build the image simply run `source test/suite/setup.sh` and then `buildProtocurl`

#### Updating Docs after changes

Generate the main docs (.md files etc.) in bash/WSL via `doc/generate-docs.sh <absolute-path-to-protocurl-repository>`.

Once a pull request is ready, run this to generate updated docs.

## Enhancements and Bugs

See [issues](https://github.com/qaware/protocurl/issues).

## FAQ

- **How is protocurl different from grpccurl?** [grpccurl](https://github.com/fullstorydev/grpcurl) only works with gRPC
  services with corresponding endpoints. However, classic REST HTTP endpoints with binary Protobuf payloads are only
  possible with `protocurl`.
- **Why is the use of a runtime curl recommended with protocurl?** curl is a simple, flexible and mature command line
  tool to interact with HTTP endpoints. In principle, we could simply use the HTTP implementation provided by the host
  programming language (Go) - and this is what we do if no curl was found in the PATH. However, as more people use
  protocurl, they will request for more features - leading to a feature creep in such a 'simple' tool as protocurl. We
  would like to avoid implementing the plentiful features which are necessary for a proper HTTP CLI tool, because HTTP
  can be complex. Since is essentially what curl already does, we recommend using curl and all advanced features are
  only possible with curl.
- **What are some nice features of protocurl?**
    - The implementation is well tested with end-2-end approval tests (see [TESTS.md](TESTS.md)). All features are
      tested based on their effect on the behavior/output. Furthermore, there are also a few cross-platform native CI
      tests running on Windows and MacOS runners.
    - The build and release process is optimised for minimal maintenance efforts. During release build, the latest
      versions of many dependencies are taken automatically (by looking up the release tags via the GitHub API).
    - The documentation and examples are generated via scripts and enable one to update the examples automatically
      rather than manually. The consistency of the outputs of the code with the checked in documentation is further
      tested in CI.
