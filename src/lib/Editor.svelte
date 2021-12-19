<script context="module" lang="ts">
	type Rec = {
		value: string;
		selectionStart: number;
		selectionEnd: number;
	};

	type History = {
		stack: Array<Rec & { timestamp: number }>;
		offset: number;
	};

	const HISTORY_LIMIT = 100;
	const HISTORY_TIME_GAP = 3000;

	const browser = typeof window !== 'undefined';
	const isWindows = browser && /Win/i.test(navigator.platform);
	const isMacLike = browser && /(Mac|iPhone|iPod|iPad)/i.test(navigator.platform);

	const getLines = (text: string, position: number) => text.substring(0, position).split('\n');
</script>

<script lang="ts">
	import { createEventDispatcher, onMount } from 'svelte';

	export let highlight: (value: string) => string;
	export let value: string;
	export let insertSpaces: boolean = true;
	export let tabSize = 2;
	export let ignoreTabKey = false;

	let input: HTMLTextAreaElement;

	let capture = true;

	let history: History = {
		stack: [],
		offset: -1,
	};

	$: highlighted = highlight(value);

	const dispatch = createEventDispatcher<{ 'value-change': string; keydown: KeyboardEvent }>();

	onMount(() => {
		recordCurrentState();
	});

	function recordCurrentState() {
		// Save current state of the input
		const { value, selectionStart, selectionEnd } = input;

		recordChange({ value, selectionStart, selectionEnd });
	}

	function recordChange(record: Rec, overwrite = false) {
		const { stack, offset } = history;

		if (stack.length && offset > -1) {
			// When something updates, drop the redo operations
			history.stack = stack.slice(0, offset + 1);

			// Limit the number of operations to 100
			const count = history.stack.length;

			if (count > HISTORY_LIMIT) {
				const extras = count - HISTORY_LIMIT;

				history.stack = stack.slice(extras, count);
				history.offset = Math.max(history.offset - extras, 0);
			}
		}

		const timestamp = Date.now();

		if (overwrite) {
			const last = history.stack[history.offset];

			if (last && timestamp - last.timestamp < HISTORY_TIME_GAP) {
				// A previous entry exists and was in short interval

				// Match the last word in the line
				const re = /[^a-z0-9]([a-z0-9]+)$/i;

				// Get the previous line
				const previous = getLines(last.value, last.selectionStart).pop().match(re);

				// Get the current line
				const current = getLines(record.value, record.selectionStart).pop().match(re);

				if (previous && current?.[1]?.startsWith(previous[1])) {
					// The last word of the previous line and current line match
					// Overwrite previous entry so that undo will remove whole word
					history.stack[history.offset] = { ...record, timestamp };

					return;
				}
			}
		}

		// Add the new operation to the stack
		history.stack.push({ ...record, timestamp });
		history.offset++;
	}

	function applyEdits(record: Rec) {
		// Save last selection state
		const last = history.stack[history.offset];

		if (last && input) {
			history.stack[history.offset] = {
				...last,
				selectionStart: input.selectionStart,
				selectionEnd: input.selectionEnd,
			};
		}

		// Save the changes
		recordChange(record);
		updateInput(record);
	}

	function undoEdit() {
		const { stack, offset } = history;

		// Get the previous edit
		const record = stack[offset - 1];

		if (record) {
			// Apply the changes and update the offset
			updateInput(record);
			history.offset = Math.max(offset - 1, 0);
		}
	}

	function redoEdit() {
		const { stack, offset } = history;

		// Get the next edit
		const record = stack[offset + 1];

		if (record) {
			// Apply the changes and update the offset
			updateInput(record);
			history.offset = Math.min(offset + 1, stack.length - 1);
		}
	}

	function updateInput(record: Rec) {
		input.value = record.value;
		input.selectionStart = record.selectionStart;
		input.selectionEnd = record.selectionEnd;

		dispatch('value-change', record.value);
	}

	function handleKeyDown(e: KeyboardEvent) {
		dispatch('keydown', e);

		if (e.key === 'Escape') {
			(e.target as HTMLTextAreaElement).blur();
		}

		const { value, selectionStart, selectionEnd } = e.target as HTMLTextAreaElement;

		const tabCharacter = (insertSpaces ? ' ' : '\t').repeat(tabSize);

		if (e.key === 'Tab' && !ignoreTabKey && capture) {
			// Prevent focus change
			e.preventDefault();

			if (e.shiftKey) {
				// Unindent selected lines
				const linesBeforeCaret = getLines(value, selectionStart);
				const startLine = linesBeforeCaret.length - 1;
				const endLine = getLines(value, selectionEnd).length - 1;
				const nextValue = value
					.split('\n')
					.map((line, i) => {
						if (i >= startLine && i <= endLine && line.startsWith(tabCharacter)) {
							return line.substring(tabCharacter.length);
						}

						return line;
					})
					.join('\n');

				if (value !== nextValue) {
					const startLineText = linesBeforeCaret[startLine];

					applyEdits({
						value: nextValue,
						// Move the start cursor if first line in selection was modified
						// It was modified only if it started with a tab
						selectionStart: startLineText.startsWith(tabCharacter)
							? selectionStart - tabCharacter.length
							: selectionStart,
						// Move the end cursor by total number of characters removed
						selectionEnd: selectionEnd - (value.length - nextValue.length),
					});
				}
			} else if (selectionStart !== selectionEnd) {
				// Indent selected lines
				const linesBeforeCaret = getLines(value, selectionStart);
				const startLine = linesBeforeCaret.length - 1;
				const endLine = getLines(value, selectionEnd).length - 1;
				const startLineText = linesBeforeCaret[startLine];

				applyEdits({
					value: value
						.split('\n')
						.map((line, i) => {
							if (i >= startLine && i <= endLine) {
								return tabCharacter + line;
							}

							return line;
						})
						.join('\n'),
					// Move the start cursor by number of characters added in first line of selection
					// Don't move it if it there was no text before cursor
					selectionStart: /\S/.test(startLineText)
						? selectionStart + tabCharacter.length
						: selectionStart,
					// Move the end cursor by total number of characters added
					selectionEnd: selectionEnd + tabCharacter.length * (endLine - startLine + 1),
				});
			} else {
				const updatedSelection = selectionStart + tabCharacter.length;

				applyEdits({
					// Insert tab character at caret
					value: value.substring(0, selectionStart) + tabCharacter + value.substring(selectionEnd),
					// Update caret position
					selectionStart: updatedSelection,
					selectionEnd: updatedSelection,
				});
			}
		} else if (e.key === 'Backspace') {
			const hasSelection = selectionStart !== selectionEnd;
			const textBeforeCaret = value.substring(0, selectionStart);

			if (textBeforeCaret.endsWith(tabCharacter) && !hasSelection) {
				// Prevent default delete behaviour
				e.preventDefault();

				const updatedSelection = selectionStart - tabCharacter.length;

				applyEdits({
					// Remove tab character at caret
					value:
						value.substring(0, selectionStart - tabCharacter.length) +
						value.substring(selectionEnd),
					// Update caret position
					selectionStart: updatedSelection,
					selectionEnd: updatedSelection,
				});
			}
		} else if (e.key === 'Enter') {
			// Ignore selections
			if (selectionStart === selectionEnd) {
				// Get the current line
				const line = getLines(value, selectionStart).pop();
				const matches = line.match(/^\s+/);

				if (matches?.[0]) {
					// Preserve indentation on inserting a new line
					const indent = '\n' + matches[0];
					const updatedSelection = selectionStart + indent.length;

					applyEdits({
						// Insert indentation character at caret
						value: value.substring(0, selectionStart) + indent + value.substring(selectionEnd),
						// Update caret position
						selectionStart: updatedSelection,
						selectionEnd: updatedSelection,
					});
				}
			}
		} else if (['(', '[', '{', `"`, `'`, '`'].includes(e.key)) {
			let chars = {
				'(': ['(', ')'],
				'[': ['[', ']'],
				'{': ['{', '}'],
				'"': ['"', '"'],
				"'": ["'", "'"],
				'`': ['`', '`'],
			}[e.key];

			// If text is selected, wrap them in the characters
			if (selectionStart !== selectionEnd && chars) {
				e.preventDefault();

				applyEdits({
					value:
						value.substring(0, selectionStart) +
						chars[0] +
						value.substring(selectionStart, selectionEnd) +
						chars[1] +
						value.substring(selectionEnd),
					// Update caret position
					selectionStart,
					selectionEnd: selectionEnd + 2,
				});
			}
		} else if (
			(isMacLike
				? // Trigger undo with ⌘+Z on Mac
				  e.metaKey && e.key === 'z'
				: // Trigger undo with Ctrl+Z on other platforms
				  e.ctrlKey && e.key === 'z') &&
			!e.shiftKey &&
			!e.altKey
		) {
			e.preventDefault();

			undoEdit();
		} else if (
			(isMacLike
				? // Trigger redo with ⌘+Shift+Z on Mac
				  e.metaKey && e.key === 'z' && e.shiftKey
				: isWindows
				? // Trigger redo with Ctrl+Y on Windows
				  e.ctrlKey && e.key === 'y'
				: // Trigger redo with Ctrl+Shift+Z on other platforms
				  e.ctrlKey && e.key === 'z' && e.shiftKey) &&
			!e.altKey
		) {
			e.preventDefault();

			redoEdit();
		} else if (e.key === 'm' && e.ctrlKey && (isMacLike ? e.shiftKey : true)) {
			e.preventDefault();

			// Toggle capturing tab key so users can focus away from the editor
			capture = !capture;
		}
	}

	function handleChange(
		e: Event & {
			currentTarget: EventTarget & HTMLTextAreaElement;
		},
	) {
		const { value, selectionStart, selectionEnd } = e.target as HTMLTextAreaElement;

		recordChange(
			{
				value,
				selectionStart,
				selectionEnd,
			},
			true,
		);

		dispatch('value-change', value);
	}
</script>

<!-- @component
  --padding: sets padding on the textarea
 -->
<div>
	<textarea
		bind:this={input}
		bind:value
		on:keydown={handleKeyDown}
		on:change={handleChange}
		autoCapitalize="off"
		autoComplete="off"
		autoCorrect="off"
		spellCheck={false}
		data-gramm={false}
		class="editor"
		{...$$restProps}
	/>

	<pre aria-hidden="true" class="editor">
    {@html highlighted}
    <br />
  </pre>
</div>

<style>
	div {
		position: relative;
		text-align: left;
		box-sizing: border-box;
		padding: 0;
		overflow: hidden;
	}

	textarea {
		position: absolute;
		top: 0;
		left: 0;

		height: 100%;
		width: 100%;

		resize: none;
		overflow: hidden;

		color: inherit;

		-moz-osx-font-smoothing: grayscale;
		-webkit-font-smoothing: antialiased;
		-webkit-text-fill-color: transparent;
	}

	.editor {
		margin: 0;
		padding: var(--padding, 4px);

		border: 0;
		box-sizing: inherit;

		background: none;

		display: inherit;

		font-family: inherit;
		font-size: inherit;
		font-style: inherit;
		font-variant-ligatures: inherit;
		font-weight: inherit;
		letter-spacing: inherit;
		line-height: inherit;
		tab-size: inherit;
		text-indent: inherit;
		text-rendering: inherit;
		text-transform: inherit;
		white-space: pre-wrap;
		word-break: keep-all;
		overflow-wrap: break-word;
	}

	pre {
		margin: 0;

		border: 0;
		box-sizing: inherit;

		background: none;

		display: inherit;

		font-family: inherit;
		font-size: inherit;
		font-style: inherit;
		font-variant-ligatures: inherit;
		font-weight: inherit;
		letter-spacing: inherit;
		line-height: inherit;
		tab-size: inherit;
		text-indent: inherit;
		text-rendering: inherit;
		text-transform: inherit;
		white-space: pre-wrap;
		word-break: keep-all;
		overflow-wrap: break-word;
	}
</style>
