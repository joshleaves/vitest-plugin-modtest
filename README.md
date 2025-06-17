# vitest-plugin-modtest

_Rust-style in-file test discovery for Vitest._

[![npm](https://img.shields.io/npm/v/vitest-plugin-modtest?color=blue)](https://www.npmjs.com/package/vitest-plugin-modtest)
[![MIT License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## What is this?

**vitest-plugin-modtest** is a [Vitest](https://vitest.dev/) plugin that lets you write Rust-style, in-file unit tests in your JavaScript/TypeScript modules by exporting a `test` function. The plugin automatically generates runnable test files for each such module, so you can test modules without writing separate `.test.ts` files.

## Why?

Inspired by Rust's `#[test]` attribute and in-file `mod test { ... }` convention, this plugin lets you:

- Keep simple tests close to your code
- Avoid boilerplate test files for modules that export a single `test`
- Prototype testable modules quickly

## Installation

```sh
npm install --save-dev vitest-plugin-modtest
```
or
```sh
yarn add --dev vitest-plugin-modtest
```

> **Peer dependencies:**  
> Requires [Vitest](https://vitest.dev/) ^3.0.0 and [Vite](https://vitejs.dev/) ^6.0.0 or higher in your project.

## Usage

Add `vitest-plugin-modtest` to your Vite config:

```ts
// vite.config.ts or vite.config.js
import VitestPluginModtest from 'vitest-plugin-modtest'

export default {
  plugins: [
    VitestPluginModtest({
      srcDir: 'src',
      tmpTestDir: 'test/.tmp',
      extensions: ['.js', '.jsx', '.ts', '.tsx']
    })
  ]
}
```

Now, in any module in your `src/` directory, simply export a `test` function that receives a `Vitest`, from which you can extract the Vitest methods.

```ts
// src/foo.ts
const innerMethod = (left: number, right: number): number => {
  return left + right
}

export const outerMethod = (left: number, right: number): string => {
  return `${left} + ${right} = ${innerMethod(left, right)}`
}

export function test({ describe, test, expect }: VitestModtest) {
  describe('innerMethod(left: number, right: number): number', () => {
    test('it adds two numbers', () => {
      expect(innerMethod(1, 2)).toBe(3)
    })
  })
  
  describe('outerMethod(left: number, right: number): string', () => {
    test('it gives a string', () => {
      expect(outerMethod(1, 2)).toBe('1 + 2 = 3')
    })
  })
}
```

Whenever you run Vitest, this plugin will auto-generate a test file for each such module (in `test/.tmp/` by default), which Vitest will pick up and execute.

### TypeScript Support

This plugin provides a global type `VitestModtest` for use in your in-file `test` function.  
You don’t need to import it, it’s available automatically via a global type declaration.

If you use TypeScript, you can annotate your test function like this:

```ts
export function test(exportedVitestMethods: VitestModtest) {
  // exportedVitestMethods.describe(...)
  // ...your tests
}
```

## How it works

- Scans your source code for JS or TS files exporting a `test` function
- For each, generates a corresponding test file in a temporary folder that imports and runs the exported `test` function
- Cleans up the generated files automatically

### Example of generated test file

```ts
// test/.tmp/foo.test.ts
import { test as testModule } from '../../src/foo.ts'
import * as vitest from 'vitest'
if (typeof testModule !== 'function') throw new Error('No test export in ../../src/foo.ts')

await testModule(vitest)
```

You should **add `test/.tmp/` to your `.gitignore`** to avoid committing generated files.

## Options

Currently, the only supported options are:

- Source files directory (defaults to `src/`)
- Output test directory (defaults to `test/.tmp/`)
- Source file extensions (defaults to `['.js', '.jsx', '.ts', '.tsx']`)

## Caveats & Limitations

- Only works with modules that export a top-level `test` function
- Designed for small, simple, in-file tests. For complex suites, use standard Vitest test files.

## Running Tests

After setup, run your tests as usual with Vitest:

```sh
npx vitest
# or
npm test
```

## Contributing

PRs and issues are welcome! Please file an issue or submit a pull request if you have an idea or improvement.

## See Also

- [Rust's test attribute](https://doc.rust-lang.org/book/ch11-01-writing-tests.html)
- [Vitest Docs](https://vitest.dev/)

## License

[MIT](LICENSE)