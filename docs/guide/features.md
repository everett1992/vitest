# Features

- [Vite](https://vitejs.dev/)'s config, transformers, resolvers, and plugins. Use the same setup from your app!
- [Jest Snapshot](https://jestjs.io/docs/snapshot-testing)
- [Chai](https://www.chaijs.com/) built-in for assertions, with [Jest expect](https://jestjs.io/docs/expect) compatible APIs.
- [Smart & instant watch mode](#watch-mode), like HMR for tests!
- [Native code coverage](#coverage) via [c8](https://github.com/bcoe/c8)
- [Sinon](https://sinonjs.org/) built-in for mocking, stubbing, and spies.
- [JSDOM](https://github.com/jsdom/jsdom) and [happy-dom](https://github.com/capricorn86/happy-dom) built-in for DOM and browser API mocking
- Components testing ([Vue](https://github.com/antfu-sponsors/vitest/test/vue), [React](https://github.com/antfu-sponsors/vitest/test/react), [Lit](https://github.com/antfu-sponsors/vitest/test/lit), [Vitesse](https://github.com/antfu-sponsors/vitest/test/vitesse))
- Workers multi-threading via [Piscina](https://github.com/piscinajs/piscina)
- ESM first, top level await
- Out-of-box TypeScript / JSX support
- Filtering, timeouts, concurrent for suite and tests

## Browser Mocking

Vitest supports both [happy-dom](https://github.com/capricorn86/happy-dom) or [jsdom](https://github.com/jsdom/jsdom) for mocking DOM and browser APIs. They don't come with Vitest, you might need to install them:

```bash
$ npm i -D happy-dom
# or
$ npm i -D jsdom
```

After that, change the `environment` option in your config file:

```ts
// vite.config.ts
import { defineConfig } from 'vite'

export default defineConfig({
  test: {
    environment: 'happy-dom' // or 'jsdom', 'node'
  }
})
```

## Watch Mode

```bash
$ vitest -w
```

Vitest smartly searches the module graph and only rerun the related tests (just like how HMR works in Vite!).

## Coverage

Vitest works perfectly with [c8](https://github.com/bcoe/c8)

```bash
$ npm i -D c8
$ c8 vitest
```

```json
{
  "scripts": {
    "test": "vitest",
    "coverage": "c8 vitest"
  }
}
```

For convenience, we also provide a shorthand for passing `--coverage` option to CLI, which will wrap the process with `c8` for you. Note when using the shorthand, you will lose the ability to pass additional options to `c8`.

```bash
$ vitest --coverage
```

For more configuration available, please refer to [c8](https://github.com/bcoe/c8)'s documentation.

## Filtering

### CLI

You can use CLI to filter test files my name:

```bash
$ vitest basic
```

Will only execute test files that contain `basic`, e.g.

```
basic.test.ts
basic-foo.test.ts
```

### Specifying a Timeout

You can optionally pass a timeout in milliseconds as third argument to tests. The default is 5 seconds.

```ts
test('name', async() => { ... }, 1000)
```

Hooks also can receive a timeout, with the same 5 seconds default.

```ts
beforeAll( async() => { ... }, 1000)
```

### Skipping suites and tests

Use `.skip` to avoid running certain suites or tests

```ts
describe.skip("skipped suite", () => {
  it("test", () => {
    // Suite skipped, no error
    assert.equal(Math.sqrt(4), 3);
  });
});

describe("suite", () => {
  it.skip("skipped test", () => {
    // Test skipped, no error
    assert.equal(Math.sqrt(4), 3);
  });
});
```

### Selecting suites and tests to run

Use `.only` to only run certain suites or tests

```ts
// Only this suite (and others marked with only) are run
describe.only("suite", () => {
  it("test", () => {
    assert.equal(Math.sqrt(4), 3);
  });
});

describe("another suite", () => {
  it("skipped test", () => {
    // Test skipped, as tests are running in Only mode
    assert.equal(Math.sqrt(4), 3);
  });

  it.only("test", () => {
    // Only this test (and others marked with only) are run
    assert.equal(Math.sqrt(4), 2);
  });
});
```

### Unimplemented suites and tests

Use `.todo` to stub suites and tests that should be implemented

```ts
// An entry will be shown in the report for this suite
describe.todo("unimplemented suite");

// An entry will be shown in the report for this test
describe("suite", () => {
  it.todo("unimplemented test");
});
```

### Running tests concurrently

Use `.concurrent` in consecutive tests to run them in parallel

```ts
// The two tests marked with concurrent will be run in parallel
describe("suite", () => {
  it("serial test", () => {
    assert.equal(Math.sqrt(4), 3);
  });
  it.concurrent("concurrent test 1", () => {
    assert.equal(Math.sqrt(4), 3);
  });
  it.concurrent("concurrent test 2", () => {
    assert.equal(Math.sqrt(4), 3);
  });
});
```

If you use `.concurrent` in a suite, every tests in it will be run in parallel

```ts
// The two tests marked with concurrent will be run in parallel
describe.concurrent("suite", () => {
  it("concurrent test 1", () => {
    assert.equal(Math.sqrt(4), 3);
  });
  it("concurrent test 2", () => {
    assert.equal(Math.sqrt(4), 3);
  });
  // No effect, same as not using .concurrent
  it.concurrent("concurrent test 3", () => {
    assert.equal(Math.sqrt(4), 3);
  });
});
```

You can also use `.skip`, `.only`, and `.todo` with concurrent suite and tests. All the following combinations are valid:

```js
describe.concurrent(...)

describe.skip.concurrent(...)
describe.concurrent.skip(...)

describe.only.concurrent(...)
describe.concurrent.only(...)

describe.todo.concurrent(...)
describe.concurrent.todo(...)

it.concurrent(...)

it.skip.concurrent(...)
it.concurrent.skip(...)

it.only.concurrent(...)
it.concurrent.only(...)

it.todo.concurrent(...)
it.concurrent.todo(...)
```