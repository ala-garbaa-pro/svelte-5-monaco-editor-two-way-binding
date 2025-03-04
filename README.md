# Svelte Project with Monaco Editor and Textarea

This project is a Svelte-based application that integrates the Monaco Editor and a customizable Textarea for rich text editing. Below are the steps to set up and test the project.

![demo](https://github.com/user-attachments/assets/5c7be488-98ed-4255-8e60-88bc0b7580d6)


## Project Setup

### Create a New Svelte Project
To start a new Svelte project, use the following command:
```bash
npx sv create test-sv5-me-twb
```

### Install Dependencies
Install the required development and runtime dependencies:

**Development Dependencies:**
```bash
pnpm i -D @monaco-editor/loader
```

**Runtime Dependencies:**
```bash
pnpm i monaco-editor
```

### Optional: Install Shadcn-Svelte
To use a styled Textarea component, you can install `shadcn-svelte`. Alternatively, you can use a regular `<textarea>` element.

## Usage Example
Below is an example setup that uses a `Textarea` and a `CodeEditor` component:

```svelte
<script>
	import CodeEditor from '$lib/components/code-editor.svelte';
	import { Textarea } from '$lib/components/ui/textarea';

	let content = $state('<p>Hello, world!</p>');
</script>

<Textarea bind:value={content} rows={4} />

<CodeEditor bind:value={content} />
```

### CodeEditor Component
Below is the implementation of the `CodeEditor` component:

```svelte
<script lang="ts">
	import loader from '@monaco-editor/loader';
	import type * as Monaco from 'monaco-editor/esm/vs/editor/editor.api';
	import { onDestroy, onMount } from 'svelte';

	let editor: Monaco.editor.IStandaloneCodeEditor;
	let monaco: typeof Monaco;
	let editorContainer: HTMLElement;

	// Define props with Svelte 5 syntax
	interface Props {
		value: string;
		language?: string;
		theme?: string;
	}

	let { value = $bindable(), language = 'html', theme = 'vs-dark' }: Props = $props();

	onMount(() => {
		(async () => {
			// Remove the next two lines to load the monaco editor from a CDN
			// see https://www.npmjs.com/package/@monaco-editor/loader#config
			const monacoEditor = await import('monaco-editor');
			loader.config({ monaco: monacoEditor.default });

			monaco = await loader.init();

			// Your monaco instance is ready, let's display some code!
			editor = monaco.editor.create(editorContainer, {
				value,
				language,
				theme,
				automaticLayout: true,
				overviewRulerLanes: 0,
				overviewRulerBorder: false,
				wordWrap: 'on'
			});

			editor.onDidChangeModelContent((e) => {
				if (e.isFlush) {
					// true if setValue call
					//console.log('setValue call');
					/* editor.setValue(value); */
				} else {
					// console.log('user input');
					const updatedValue = editor?.getValue() ?? ' ';
					value = updatedValue;
				}
			});
		})();
	});

	$effect(() => {
		if (value) {
			if (editor) {
				// check if the editor is focused
				if (editor.hasWidgetFocus()) {
					// let the user edit with no interference
				} else {
					if (editor?.getValue() ?? ' ' !== value) {
						editor?.setValue(value);
					}
				}
			}
		}
		if (value === '') {
			editor?.setValue(' ');
		}
	});

	onDestroy(() => {
		monaco?.editor.getModels().forEach((model) => model.dispose());
		editor?.dispose();
	});
</script>

<div class="container" bind:this={editorContainer}></div>

<style>
	.container {
		width: 100%;
		height: 600px;
		padding: 0;
		border-radius: 50px;
	}
</style>
```

## Features
- **CodeEditor:** A customizable Monaco Editor instance with support for themes and languages.
- **Textarea:** A simple or styled Textarea component that can be bound to the editor.

## How to Test
1. Add the provided `CodeEditor` and `Textarea` components to your project.
2. Bind a shared value (`content`) to both components.
3. Edit text in either the editor or the textarea to see real-time updates.

## License
This project is licensed under [MIT License](LICENSE).

## Contributing
Contributions are welcome! Please submit a pull request or open an issue if you encounter any problems.
