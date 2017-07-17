# GraphQL Code Generator

[![npm version](https://badge.fury.io/js/graphql-code-generator.svg)](https://badge.fury.io/js/graphql-code-generator) [![Build Status](https://travis-ci.org/dotansimha/graphql-code-generator.svg?branch=master)](https://travis-ci.org/dotansimha/graphql-code-generator) [![bitHound Overall Score](https://www.bithound.io/github/dotansimha/graphql-code-generator/badges/score.svg)](https://www.bithound.io/github/dotansimha/graphql-code-generator) [![codebeat badge](https://codebeat.co/badges/ec220cc6-31f0-4a80-85cc-0dfa162d8e53)](https://codebeat.co/projects/github-com-dotansimha-graphql-code-generator) [![bitHound Dependencies](https://www.bithound.io/github/dotansimha/graphql-code-generator/badges/dependencies.svg)](https://www.bithound.io/github/dotansimha/graphql-code-generator/master/dependencies/npm) [![bitHound Dev Dependencies](https://www.bithound.io/github/dotansimha/graphql-code-generator/badges/devDependencies.svg)](https://www.bithound.io/github/dotansimha/graphql-code-generator/master/dependencies/npm) [![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/) 

## Overview

GraphQL code generator, with flexible support for multiple languages and platforms, and the ability to create custom generated projects based on GraphQL schema or operations.

This generator generates both models (based on GraphQL server-side schema), and documents (client-side operations, such as `query`, `mutation` as `subscription`).

There are 3 types of template generators available at the moment:

* Single file - generates a single output file from GraphQL schema and operations
* Multiple files - generated multiple files (usually file per type/operation/enum...)
* Project - generated multiple/single file, based on custom templates in a directory.

**Supported languages/platforms (`0.8.0`):**

| Language        | Type           | CLI Name                                                                  |
|-----------------|----------------|---------------------------------------------------------------------------|
| TypeScript      | Single File    | ts, typescript, ts-single, typescript-single                              |
| TypeScript      | Multiple Files | ts-multiple, typescript-multiple                                          |

If you are looking for the **Flow** / **Swift** generators, please note that we will implement it soon again, but you can use `0.5.5` from NPM.

## Examples

This package

## Installation

To install the generator, use the following:

    $ npm install --save-dev graphql-code-generator

Or, using Yarn:
    
    $ yarn add -D graphql-code-generator

And then to use it, execute if from NPM script, for use `$(npm bin)/gql-gen ...` from the command line.

> You can also install it as global NPM module and use it with `gql-gen` executable.

## Usage

This package offers both modules exports (to use with NodeJS/JavaScript application), or CLI util.

### CLI 

CLI usage is as follow:

    $ gql-gen [options] [documents ...]
    
Allowed flags:    

| Flag Name          | Type     | Description                                                                            |
|--------------------|----------|----------------------------------------------------------------------------------------|
| -f,--file          | String   | Introspection JSON file, must provide file or URL flag                                 |
| -u,--url           | String   | GraphQL server endpoint to fetch the introspection from, must provide URL or file flag |
| -e,--export        | String   | Path to a JavaScript (es5/6) file that exports (as default export) your `GraphQLSchema` object |
| -h,--header        | String   | Header to add to the introspection HTTP request when using --url  |
| -t,--template      | String   | Template name, for example: "typescript" (not required when using `--project`)         |
| -p,--project       | String   | Project directory with templates (refer to "Project Generation" section)                |
| --project-config   | String   | Path to project config JSON file (refer to "Project Generation" section), defaults to `gqlgen.json` |
| -o,--out           | String   | Path for output file/directory. When using single-file generator specify filename, and when using multiple-files generator specify a directory                                     |
| -m,--no-schema     | void     | If specified, server side schema won't be generated through the template (enums won't omit) |
| -c,--no-documents  | void     | If specified, client side documents won't be generated through the template |
| -d,--dev           | void     | Turns ON development mode - prints output to console instead of files                  |
| documents...       | [String] | Space separated paths of `.graphql` files or code files (glob path is supported) that contains GraphQL documents inside strings, or with `gql` tag (JavaScript), this field is optional - if no documents specified, only server side schema types will be generated                           |

**Usage examples:***

> Note: when specifying a glob path (with `*` or `**`), make sure to wrap the argument with double quotes (`"..."`).

- With local introspection JSON file, generate TypeScript types:

        $ gql-gen --file mySchema.json --template typescript --out ./typings/ "./src/**/*.graphql"
    
- With local introspection JSON file, generate TypeScript files, from GraphQL documents inside code files (`.ts`):

        $ gql-gen --file mySchema.json --template typescript --out ./typings/ "./src/**/*.ts"
    
- With remote GraphQL endpoint that requires Authorization, generate TypeScript types:

        $ gql-gen --url http://localhost:3010/graphql --header "Authorization: MY_KEY" --template typescript --out ./typings/ "./src/**/*.graphql"
    
## Integration

To use inside an existing project, I recommend to add a pre-build script that executes the code generator, inside you `package.json`, for example:

```json
{
    "name": "my-project",
    "scripts": {
        "prebuild": "gql-gen ...",
        "build": "webpack"
    }
}
```

## Project Generation

This generator package also allow you to integrate it as a framework and be part of you whole development process.

This way you can generate code based on your GraphQL schema / operations.

To start using GraphQL code generator with your project, install the module, and then create a JSON file called `gqlgen.json` in your project's root directory, specifying the following:

```json
{
  "flattenTypes": true,
  "primitives": {
    "String": "string",
    "Int": "number",
    "Float": "number",
    "Boolean": "boolean",
    "ID": "string"
  }
}
```

> The purpose of this file to to indicate to the GraphQL code generate if you want to flatten selection set, and specify your environment's scalars transformation, for example: `String` from GraphQL is `string` in TypeScript.

Now create a simple template file with this special structure: `{file-prefix}.{file-extension}.{required-context}.gqlgen`, for example: `hoc.js.all.gqlgen`, and use any custom template, for example:

```handlebars
{{#each types}}
    GraphQL Type: {{ name }}
{{/each}}
```

This file will compile by the generator as Handlebars template, with the `all` context, and the result file name will be `hoc.js`.

There are a lot of available contexts, and you can read the [full documentation about how to generate a custom project with custom template here]().

## Packages

GraphQL code generator implementation is separated to multiple NPM packages:

- `graphql-codegen-core`: The core package, receives `GraphQLSchema` and GraphQL operations and input, and outputs a custom transformed JSON structure.
- `graphql-codegen-compiler`: receives `core` package output and modify the structure to support easy integration with Handlebars.
- `graphql-codegen-generators`: includes built-in generators with config and templates.
- `graphql-codegen-cli`: CLI executable, reading and writing file from/to the file-system or remote endpoint.

#### Other Environments

If you are using GraphQL with environment different from NodeJS and wish to generate types and interfaces for your platform, start by installing NodeJS and the package as global, and then add the generation command to your build process.

## Contributing

Feel free to open issues (for bugs/questions) and create pull requests (add generators / fix bugs).

## License

[![GitHub license](https://img.shields.io/badge/license-MIT-lightgrey.svg?maxAge=2592000)](https://raw.githubusercontent.com/apollostack/apollo-ios/master/LICENSE)

MIT
