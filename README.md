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

**Default:** `""`

**Example:**

```svelte
<SimpleCodeEditor value="var num = 10;" />

<!-- Update local variable `value` as user types into the editor -->
<SimpleCodeEditor bind:value={value} />
```
