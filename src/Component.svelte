<script>
  import { getContext, onDestroy } from "svelte"

  export let field
  export let label
  export let placeholder
  export let defaultValue
  export let delay = 0
  export let onChange
  export let onInput
  export let validation
  let inputValue = defaultValue
  const component = getContext("component")
  const formContext = getContext("form")
  const formStepContext = getContext("form-step") 
      
  const formApi = formContext?.formApi
  $: formStep = formStepContext ? $formStepContext || 1 : 1
  $: formField = formApi?.registerField(field, "string", defaultValue, false, validation, formStep)

  const { styleable } = getContext("sdk")

  let fieldApi;
  let fieldState; 

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister()
    unsubscribe?.()
  })

  const handleChange = e => {
    fieldApi?.setValue(e.target.value)
    if (onChange) {
      onChange({ value: e.target.value })
    }
  }

  let timer;
  function debounce(func, timeout = 300){
    return (...args) => {
      clearTimeout(timer);
      timer = setTimeout(() => { func.apply(this, args); }, timeout);
    };
  }
  
  function saveInput(tValue){
    console.log('triggered2')
    if(onInput) onInput({ value: tValue})
  }

  const handleInput = e => {
    return debounce(() => saveInput(e.target.value), delay)()
  }  


  

  const fieldGroupContext = getContext("field-group")
  const labelPos = fieldGroupContext?.labelPosition || "above"

  $: labelClass = labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Search bar needs to be wrapped in a form</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
    <div class="spectrum-Textfield">
      <input
        class="spectrum-Textfield-input"
        value={fieldState?.value}
        on:input={handleInput}
        on:change={handleChange}
        placeholder={placeholder}
        />
    </div>
    {#if fieldState?.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  {/if}
</div>

<style>
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
  .spectrum-Form-item {
    display: flex;
    flex-direction: column;
  }
  .spectrum-Textfield {
    width: 100%;
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
  .error {
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
</style>