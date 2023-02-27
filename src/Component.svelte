<script>
  import { getContext, onDestroy } from "svelte";

  import EditorJS from "@editorjs/editorjs";
  import Header from "@editorjs/header";
  import List from "@editorjs/list";
  import Underline from "@editorjs/underline";

  export let field;
  export let label;
  export let background;

  let fieldApi;
  let fieldState;

  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  const formApi = formContext?.formApi;

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

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  const editor = new EditorJS({
    holder: "editorjs",
    placeholder: "Seu artigo aqui...",
    tools: {
      header: {
        class: Header,
        inlineToolbar: true,
        config: {
          placeholder: "Header",
        },
        shortcut: "CMD+SHIFT+H",
      },
      list: {
        class: List,
        inlineToolbar: true,
      },
      underline: {
        class: Underline,
        inlineToolbar: true,
      },
    },
    onReady: async () => {
      if (fieldState.value) {
        await editor.blocks.renderFromHTML(fieldState.value);
      }
    },
  });

  function convertDataToHtml({ blocks }) {
    var convertedHtml = "";

    blocks.map(block => {
      switch (block.type) {
        case "header":
          convertedHtml += `<h${block.data.level}>${block.data.text}</h${block.data.level}>`;
          break;
        case "embded":
          convertedHtml += `<div><iframe width="560" height="315" src="${block.data.embed}" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe></div>`;
          break;
        case "paragraph":
          convertedHtml += `<p>${block.data.text}</p>`;
          break;
        case "delimiter":
          convertedHtml += "<hr />";
          break;
        case "image":
          convertedHtml += `<img class="img-fluid" src="${block.data.file.url}" title="${block.data.caption}" /><br /><em>${block.data.caption}</em>`;
          break;
        case "list":
          convertedHtml += "<ul>";
          block.data.items.forEach(function(li) {
            convertedHtml += `<li>${li}</li>`;
          });
          convertedHtml += "</ul>";
          break;
        default:
          console.log("Unknown block type", block.type);
          break;
      }
    });

    return convertedHtml;
  }

  function onHandleChange() {
    editor
      .save()
      .then((outputData) => {
        console.log("Article data: ", outputData);
        fieldApi.setValue(convertDataToHtml(outputData));
      })
      .catch((error) => {
        console.log("Saving failed: ", error);
      });
  }

  setInterval(() => onHandleChange(), 5000);
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form.</div>
  {:else if !field}
    <div class="error">Please select a field</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>

    <div class="spectrum-Form-itemField">
      <div
        on:change={() => onHandleChange()}
        style:background-color={background}
        id="editorjs"
      />
    </div>

    {#if fieldState?.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  {/if}
</div>

<style>
  #editorjs {
    color: rgb(215, 217, 221);

    border: 1px solid #494f5a;
    border-radius: 4px;

    padding-left: 12px;
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
