# svelte-simple-code-editor

Simple no-frills code editor with syntax highlighting.

> This is the Svelte port of the amazing [react-simple-code-editor](https://github.com/satya164/react-simple-code-editor) by [@satya164](https://github.com/satya164).

## Features

- 🤏 Small - Less than 3KB min+gzip.
- 🐇 Simple - Quite simple to use, and effectively zero config required!
- 🧙 Elegant - Use a Svelte component rather than setting things up in `onMount` hook.
- 📤 Highly customizable - Offers tons of options that you can modify to get different behaviors.
- 💻 SSR friendly - Works seamlessly in Sveltekit and other Server Side Rendering environments!

## Installing

```sh
# pnpm
pnpm add svelte-simple-code-editor

# npm
npm install svelte-simple-code-editor

# yarn
yarn add svelte-simple-code-editor
```

## Usage

```svelte
<script>
	import { SimpleCodeEditor } from 'svelte-simple-code-editor';
</script>

<SimpleCodeEditor value="let x = 1;\n console.log(x);" />
```

## Props

There's tons of options available for this package. All of them are already documented within the code itself, so you'll never have to leave the code editor.

### value

Code to be shown.

Note: This value is changed from inside of this component. By default, you won't be able to detect changes.

But if you use it as `bind:value`, all changeswill flow back to the component where yur using `SimpleCodeEditor`.

**Type:** `string`

**Default:** `""`

**Example:**

```svelte
<SimpleCodeEditor value="var num = 10;" />

<!-- Update local variable `value` as user types into the editor -->
<SimpleCodeEditor bind:value={value} />
```

### highlight

Highlight function. takes a string and returns a string(the highlighted code).

**Type:** `(value: string) => string`

**Default:** `() => ''`

**Example:**

```svelte
<script>
	import Prism from 'prismjs';
</script>

<SimpleCodeEditor highlight={(code) => Prism.highlight(code, Prism.languages.javascript, 'javascript').value} />
```

### insertSpaces

Insert spaces instead of tabs.

**Type:** `boolean`

**Default:** `true`

**Example:**

```svelte
<SimpleCodeEditor insertSpaces={false} />
```

### tabSize

Tab size. When you hit Tab key, how much indentation should be added.

**Type:** `number`

**Default:** `2`

**Example:**

```svelte
<SimpleCodeEditor tabSize={4} />
```

### ignoreTabKey

When you hit tab, should it add indentation or focus out of the editor

**Type:** `boolean`

**Default:** `false`

**Example:**

```svelte
<SimpleCodeEditor ignoreTabKey={true} />
```

### preClass

Class to apply to pre tag used internally. This is necessary when using custom themes with PrismJS and you have to give your `<pre>` tag a class like `language-`.

**Type:** `string`

**Default:** `""`

**Example:**

```svelte
<SimpleCodeEditor preClass="language-javascript" />
```

### textAreaProps

Attributes to pass to the internal `<textarea>` tag.

**Type:** `object`

**Default:** `{}`

**Example:**

```svelte
<SimpleCodeEditor textAreaProps={{ 'aria-label': 'svelte-simple-code-editor-textarea' }} />
```

## Custom style props

You can specify the following style props:

- `--padding`: Padding of the editor. **Default**: `4px`

This is it for now. You can recommend more props to be added in the future here: [Custom style props to be allowed](https://github.com/PuruVJ/svelte-simple-code-editor/issues/1)

## License

MIT License © Puru Vijay
