<script>
  import { getContext} from "svelte";
  import "shepherd.js/dist/css/shepherd.css";
  import Shepherd from "shepherd.js/dist/js/shepherd.min.js";

  export let useModalOverlay;
  export let confirmCancel;
  export let exitOnEsc;
  export let keyboardNavigation;
  export let confirmCancelMessage;
  export let tourName;
 


  const { styleable,API } = getContext("sdk");
  const component = getContext("component");
  $: tour = new Shepherd.Tour({
    tourName: tourName,
    useModalOverlay: useModalOverlay,
    confirmCancel:  confirmCancel,
    exitOnEsc: exitOnEsc,
    keyboardNavigation: keyboardNavigation,
    confirmCancelMessage: confirmCancelMessage ?? '',
    tourName: tourName,
    defaultStepOptions: {
      cancelIcon: {
        enabled: true,
      },
      scrollTo: { behavior: "smooth", block: "center" },
    },
  });
  
 

  function handleClick() {

    let list_of_sheep = document.querySelectorAll(".sheep_step");
    for (let i = 0; i < list_of_sheep.length; i++) {
      tour.addStep({
        title: list_of_sheep[i].dataset.title ?? "",
        text: list_of_sheep[i].dataset.text ?? "Welcome to the walkthrough",
        attachTo: {
          element: list_of_sheep[i],
          on: list_of_sheep[i].dataset.pos ?? "auto",
        },
        buttons: [
          {
            action() {
              return this.back();
            },
            classes: "shepherd-button-secondary",
            text: "Back",
          },
          {
            action() {
              return this.next();
            },
            text: "Next",
          },
        ],
        id: "creating",
      });
    }
    tour.start()
  }

</script>
<div use:styleable={$component.styles}>
  <button type="button" id="blobby" on:click={handleClick}>Test</button>

</div>
