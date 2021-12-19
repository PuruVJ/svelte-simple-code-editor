<script lang="ts">
	import { SimpleCodeEditor } from '$lib';
	import Prism from 'prismjs';
	// import 'prismjs/components/prism-clike';
	import 'prismjs/components/prism-typescript';
	// import 'prismjs/themes/prism.css'; //Example style, you can use another
	import '../themes/odp.css';

	const { highlight, languages } = Prism;
</script>

<div style="font-family: monospace; background-color: #282c34">
	<SimpleCodeEditor
		highlight={(val) => highlight(val, languages.typescript, 'typescript')}
		value={`
import { SvelteComponentTyped } from "svelte";
declare const __propDef: {
    props: {
        [x: string]: any;
        highlight: (value: string) => string;
        value: string;
    };
    events: {
        [evt: string]: CustomEvent<any>;
    };
    slots: {};
};
export declare type EditorProps = typeof __propDef.props;
export declare type EditorEvents = typeof __propDef.events;
export declare type EditorSlots = typeof __propDef.slots;
/** --padding: sets padding on the textarea */
export default class Editor extends SvelteComponentTyped<EditorProps, EditorEvents, EditorSlots> {
}
export {};
    `}
		preClass="language-"
		--bgcolor="#282c34"
	/>
</div>

<br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br
/><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br
/><br /><br /><br /><br /><br /><br /><br />

<a href="/page">brjbjegb</a>
