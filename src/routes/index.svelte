<script lang="ts">
	import { SimpleCodeEditor } from '$lib';
	import Prism from 'prismjs';
	// import 'prismjs/components/prism-clike';
	// import 'prismjs/components/prism-javascript';
	import 'prismjs/themes/prism.css'; //Example style, you can use another

	const { highlight, languages } = Prism;
</script>

<div style="font-family: monospace;">
	<SimpleCodeEditor
		highlight={(val) => highlight(val, languages.javascript, 'javascript')}
		value={`function add(a, b) {\n  return a + b;\n}`}
	/>
</div>
