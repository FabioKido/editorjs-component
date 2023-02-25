<script>
  import { getContext, onDestroy } from "svelte"
  import EditorJS from '@editorjs/editorjs'; 
  import Header from '@editorjs/header'; 
  import List from '@editorjs/list';
  import Underline from '@editorjs/underline'

  export let field;
  export let label;

  let fieldApi
  let fieldState

  const { styleable } = getContext("sdk")
  const component = getContext("component")
  const formContext = getContext("form")
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");
  
  const formApi = formContext?.formApi
  
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "text",
    "",
    false,
    null,
    formStep
  );

  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: labelClass = labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  const editor = new EditorJS({
    holder: 'editorjs',
    placeholder: 'Seu artigo aqui...',
    tools: {
      header: {
        class: Header,
        inlineToolbar: true,
        config: {
          placeholder: 'Header'
        },
        shortcut: 'CMD+SHIFT+H'
      },
      list: { 
        class: List, 
        inlineToolbar: true 
      },
      undefine: {
        class: Underline,
        inlineToolbar: true
      }
    },
    //data: {
      // "time": 1677354256991,
      // "blocks": [
      //     {
      //         "id": "2JVEkuKVN7",
      //         "type": "paragraph",
      //         "data": {
      //             "text": "Bom dia senhores 🗿🍷"
      //         }
      //     },
      //     {
      //         "id": "1n9eZ9KHPT",
      //         "type": "header",
      //         "data": {
      //             "text": "opkigdklkfg",
      //             "level": 2
      //         }
      //     }
      // ],
      // "version": "2.26.5"
      
    //},
    onReady: async () => {
      if(fieldState.value){
        await editor.blocks.renderFromHTML(fieldState.value);
      }
    }
  });

  function onHandleChange() {
    editor.save().then((outputData) => {
      console.log('Article data: ', outputData)
      fieldApi.setValue(outputData)
    }).catch((error) => {
      console.log('Saving failed: ', error)
    })
  }

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form.</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>

    <div class="spectrum-Form-itemField">
      <div id="editorjs"></div>
      <button on:click={() => onHandleChange()}>Salvar</button>
    </div>
  {/if}
</div>

<style>
	#editorjs {
    background: rgb(28, 30, 34);
    color: rgb(215, 217, 221);

    border: 1px solid #494F5A;
    border-radius: 4px;

    min-width: 670px;
	}

  label {
    white-space: nowrap;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>