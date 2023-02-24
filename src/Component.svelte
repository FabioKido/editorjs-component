<script>
  import { getContext } from "svelte"
  import EditorJS from '@editorjs/editorjs'; 
  import Header from '@editorjs/header'; 
  import List from '@editorjs/list'; 

  export let label
  export let field;

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  const editor = new EditorJS({
    holder: 'editorjs',
    inlineToolbar: ['link', 'marker', 'bold', 'italic'],
    tools: {
      header: {
        class: Header,
        inlineToolbar: ['marker', 'link'],
        config: {
          placeholder: 'Header'
        },
        shortcut: 'CMD+SHIFT+H'
      },
      list: { 
        class: List, 
        inlineToolbar: true 
      } 
    }
  });
</script>

<div use:styleable={$component.styles}>
  <div id="editorjs" class=editor></div>
  <button>Save Article</button>
</div>

<style>
	.editor {
    background-color: black;
    color: white;
	}
</style>