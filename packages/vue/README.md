# @shutter/vue [![NPM Version](https://img.shields.io/npm/v/@shutter/vue.svg)](https://www.npmjs.com/package/@shutter/vue)

Vue.js visual snapshot testing library using [shutter.sh](https://shutter.sh).

Renders Vue components to PNG snapshots using the shutter.sh service and saves snapshots locally. Future test runs will re-render the content and diff it against your previously saved snapshot.

The tests will fail if the current snapshot does not match the expected one. If that visual change was intended, you can update your local snapshot.


## Installation

```bash
npm install --save-dev @shutter/vue
# or using yarn:
yarn add --dev @shutter/vue
```

## Usage

```js
import createVueShutter from '@shutter/vue'
import MyButton from './MyButton'

const shutter = createVueShutter(__dirname)

describe('Button component', function () {
  after(async function () {
    // Collect and evaluate results once we are done
    await shutter.finish()
  })

  it('matches visual snapshot', async function () {
    await Promise.all([
      shutter.snapshot('Default Button', {
        components: { MyButton },
        template: '<my-button label='Click me'></my-button>'
      }),
      shutter.snapshot('Primary Button',  {
        components: { MyButton },
        template: '<my-button primary label='Click me'></my-button>'
      })
    ])
  })
})
```

When running this test, `@shutter/vue` will render your button in the desired states and compare the resulting snapshots to your previous component snapshots.

It will also print an inspection URL which links to the shutter.sh app where you can see the rendered component, the expected outcome and a visual diff between them.

## API

```typescript
import createVueShutter from '@shutter/vue'

const shutter = createVueShutter(__dirname)

async function run () {
  await shutter.snapshot('List', { template: '<ul><li>List item</li></ul>' })
  await shutter.finish()
}

run()
```

### createVueShutter(testDirectoryPath: string, options: ShutterOptions): Shutter

Creates a shutter instance. You need to pass your testing directory (can usually just use `__dirname`), so it knows where to save the snapshots.

```typescript
interface ShutterOptions {
  /** Local files to upload, like stylesheets. Use `addFile()` to populate this array. */
  files?: File[],

  /** Custom content to go into the <head> tag of the document. */
  head?: string,

  /** Layout to use for rendering. Pass a custom layout to change the overall page structure. */
  layout?: (bodyContent: string, headContent: string) => string,

  /** Set a custom path to your local snapshot files here. */
  snapshotsPath?: string,

  /** Set custom image comparison options here. Used to compare the current snapshot to the expectation. */
  diffOptions?: DiffOptions,

  /** Set custom rendering options here. */
  renderOptions?: RenderOptions
}
```

Check out the [`@shutter/api` documentation](../api/README.md) for the `DiffOptions` and `RenderOptions` details.

### shutter.snapshot(testName: string, element: object, options: SnapshotOptions = {}): Promise<void>

Send page contents to the shutter.sh service to be rendered.

The returned promise will resolve once the upload is done, but before the rendering has finished. That is why you need to call `shutter.finish()` after calling `shutter.snapshot()` the last time.

```typescript
interface SnapshotOptions {
  layout?: (htmlContent: string) => string,
  diffOptions?: DiffOptions,
  renderOptions?: RenderOptions
}
```

The options are mostly the same as the `ShutterOptions`. They can be used to override the options per test case.

Check out the [`@shutter/api` documentation](../api/README.md) for the `DiffOptions` and `RenderOptions` details.

### shutter.finish(): Promise<TestResult[]>

Waits until all rendering tasks have finished, then collects and evaluates the results.

Will throw with a test results summary if snapshots don't match. Prints a success message and an inspection link if everything matched.

### addFile(localPath: string, serveAsPath: string): Promise<File>

Reads a local file and prepares it for submission along the HTML content to render. Use it to submit local stylesheets, images, etc.

Pass the resulting `File` to `createShutter()` as `options.files`.

**Note: The submitted file will be publicly accessible.**

### createFileFromBuffer(content: Buffer, fileName: string, options: FileCreationOptions = {}): File

Allows you to submit a file from in-memory contents.

```typescript
interface FileCreationOptions {
  contentType?: string
}
```

**Note: The submitted file will be publicly accessible.**

## See also

Check out the documentation at <https://docs.shutter.sh>.
