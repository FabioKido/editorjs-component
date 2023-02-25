<script>
  import { getContext, onDestroy } from "svelte"
  import EditorJS from '@editorjs/editorjs'; 
  import Header from '@editorjs/header'; 
  import List from '@editorjs/list';

  export let label;
  export let field;

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

  // editor.save().then((outputData) => {
  //     console.log('Article data: ', outputData)
  //   }).catch((error) => {
  //     console.log('Saving failed: ', error)
  //   })

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
    <!-- <NumberSpinner 
      class={`spectrum-Textfield spectrum-Textfield-input spectrum-Heading--size${size || "M"}`}
      mainStyle={`text-align: ${align}; height:auto;`}
      value={fieldState?.value}
      on:change={(ev) => fieldApi.setValue(ev.detail)}
    /> -->

    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>

    <div class="spectrum-Form-itemField">
      <div id="editorjs"></div>
      <!-- <button on:click={() => console.log('salvou...')}>Salvar</button> -->
    </div>
  {/if}
</div>

<style>
	#editorjs {
    background: rgb(28, 30, 34);
    color: rgb(215, 217, 221);

    border: 1px solid #494F5A;
    border-radius: 4px;
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