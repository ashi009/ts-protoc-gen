[![Master Build](https://travis-ci.org/improbable-eng/ts-protoc-gen.svg?branch=master)](https://travis-ci.org/improbable-eng/ts-protoc-gen)
[![NPM](https://img.shields.io/npm/v/ts-protoc-gen.svg)](https://www.npmjs.com/package/ts-protoc-gen)
[![NPM](https://img.shields.io/npm/dm/ts-protoc-gen.svg)](https://www.npmjs.com/package/ts-protoc-gen)
[![Apache 2.0 License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

# ts-protoc-gen
> Protoc Plugin for generating TypeScript Declarations

This repository contains a [protoc](https://github.com/google/protobuf) plugin that generates TypeScript declarations 
(`.d.ts` files) that match the JavaScript output of `protoc --js_out=import_style=commonjs,binary`. This plugin can
also output service definitions as both `.js` and `.d.ts` files in the structure required by [grpc-web](https://github.com/improbable-eng/grpc-web).

This plugin is tested and written using TypeScript 2.7.

## Installation
As a prerequisite, install the standard C++ implementation of protocol buffers from [developers.google.com/protocol-buffers](https://developers.google.com/protocol-buffers)

For the latest stable version:

```bash
npm install ts-protoc-gen
```

For our latest build straight from master:

```bash
npm install ts-protoc-gen@next
```

## Contributing
Contributions are welcome! Please refer to [CONTRIBUTING.md](https://github.com/improbable-eng/ts-protoc-gen/blob/master/CONTRIBUTING.md) for more information.

## Usage
As mentioned above, this plugin for `protoc` serves two purposes:
1. Generating TypeScript Definitions for CommonJS modules generated by protoc
2. Generating gRPC Service Stubs for use with [grpc-web](https://github.com/improbable-eng/grpc-web).

### Generating TypeScript Definitions for CommonJS modules generated by protoc
By default, protoc will generate ES5 code when the `--js_out` flag is used (see [javascript compiler documentation](https://github.com/google/protobuf/tree/master/js)). You have the choice of two module syntaxes, [CommonJS](https://nodejs.org/docs/latest-v8.x/api/modules.html) or [closure](https://developers.google.com/closure/library/docs/tutorial). This plugin (`ts-protoc-gen`) can be used to generate Typescript definition files (`.d.ts`) to provide type hints for CommonJS modules only.

To generate TypeScript definitions you must first configure `protoc` to use this plugin and then specify where you want the TypeScript definitions to be written to using the `--ts_out` flag.

```bash
# Path to this plugin
PROTOC_GEN_TS_PATH="./node_modules/.bin/protoc-gen-ts"

# Directory to write generated code to (.js and .d.ts files) 
OUT_DIR="./generated"

protoc \
    --plugin="protoc-gen-ts=${PROTOC_GEN_TS_PATH}" \
    --js_out="import_style=commonjs,binary:${OUT_DIR}" \
    --ts_out="${OUT_DIR}" \
    users.proto base.proto
```

In the above example, the `generated` folder will contain both `.js` and `.d.ts` files which you can reference in your TypeScript project to get full type completion and make use of ES6-style import statements, eg:

```js
import { MyMessage } from "../generated/users_pb";

const msg = new MyMessage();
msg.setName("John Doe");
```

### Generating gRPC Service Stubs for use with grpc-web
[gRPC](https://grpc.io/) is a framework that enables client and server applications to communicate transparently, and makes it easier to build connected systems.

[grpc-web](https://github.com/improbable-eng/grpc-web) is a comparability layer on both the server and client-side which allows gRPC to function natively in modern web-browsers.

To generate client-side service stubs from your protobuf files you must configure ts-protoc-gen to emit service definitions by passing the `service=true` param to the `--ts_out` flag, eg:

```
# Path to this plugin
PROTOC_GEN_TS_PATH="./node_modules/.bin/protoc-gen-ts"

# Directory to write generated code to (.js and .d.ts files) 
OUT_DIR="./generated"

protoc \
    --plugin="protoc-gen-ts=${PROTOC_GEN_TS_PATH}" \
    --js_out="import_style=commonjs,binary:${OUT_DIR}" \
    --ts_out="service=true:${OUT_DIR}" \
    users.proto base.proto
```

The `generated` folder will now contain both `pb_service.js` and `pb_service.d.ts` files which you can reference in your TypeScript project to make RPCs.

```js
import { UserServiceClient, GetUserRequest } from "../generated/users_pb_service"

const client = new UserServiceClient("https://my.grpc/server");
const req = new GetUserRequest();
req.setUsername("johndoe");
client.getUser(req, (err, user) => { /* ... */ });
```

## Examples
For a sample of the generated protos and service definitions, see [examples](https://github.com/improbable-eng/ts-protoc-gen/tree/master/examples)
