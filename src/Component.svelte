<script>
  import { getContext } from "svelte";
  import Tour from "./lib/Tour.svelte";
  const { styleable } = getContext("sdk");
  const component = getContext("component");

  export let useModalOverlay;
  export let confirmCancel;
  export let exitOnEsc;
  export let keyboardNavigation;
  export let confirmCancelMessage;
  export let tourName;
  export let autoStart;
  export let autoStartField;
  export let tourSteps;
  export let onCompletion;
  export let onCancel;

</script>

<div use:styleable={$component.styles}>
  <Tour {tourName} {useModalOverlay} {confirmCancel} {exitOnEsc} {keyboardNavigation} {confirmCancelMessage} {autoStart} {autoStartField} {onCompletion} {onCancel} {tourSteps}/>  
</div> 