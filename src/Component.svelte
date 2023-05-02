<script>
  import { getContext, onDestroy, Input } from "svelte"
  import MaskInput from "./MaskInput.svelte"

  export let field
  export let label
  export let mask = "0000-000000-00000"
  export let validation
  export let onInput

  const component = getContext("component")
  const formContext = getContext("form")
  const formStepContext = getContext("form-step") 
      
  const formApi = formContext?.formApi
  $: formStep = formStepContext ? $formStepContext || 1 : 1
  $: formField = formApi?.registerField(field, "string", "", false, validation, formStep)

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

  const handleInput = e => {
    if (onInput) {
      console.log('fire input', e)
      onInput({ value: e.detail })
    }
  }

  const fieldGroupContext = getContext("field-group")
  const labelPos = fieldGroupContext?.labelPosition || "above"

  $: labelClass = labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  <input on:input={handleInput} />
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