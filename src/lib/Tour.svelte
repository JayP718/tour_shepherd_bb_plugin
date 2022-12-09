<script>
  import { tick, onMount, getContext, beforeUpdate, afterUpdate } from "svelte";
  import "shepherd.js/dist/css/shepherd.css";
  import Shepherd from "shepherd.js/dist/js/shepherd.min.js";
  import Button from "../../node_modules/@budibase/bbui/src/Button/Button.svelte";

  export let useModalOverlay;
  export let confirmCancel;
  export let exitOnEsc;
  export let keyboardNavigation;
  export let confirmCancelMessage;
  export let tourName;
  export let autoStart;
  export let autoStartField;
  export let error;
  export let tourSteps;
  export let onCompletion;
  export let onCancel;
  export let tour;

  export let inBuilder;
  const { styleable, builderStore, API } = getContext("sdk");
  const component = getContext("component");

  $: tour = new Shepherd.Tour({
    tourName: tourName,
    useModalOverlay: useModalOverlay,
    confirmCancel: confirmCancel,
    exitOnEsc: exitOnEsc,
    keyboardNavigation: keyboardNavigation,
    confirmCancelMessage: confirmCancelMessage ?? "",
    defaultStepOptions: {
      cancelIcon: {
        enabled: true,
      },
      scrollTo: { behavior: "smooth", block: "center" },
    },
  });

  function buttonHandler(buttons_booleans) {
    let buttons = [];
    for (var i = 0; i < buttons_booleans.length; i++) {
      if (Object.values(buttons_booleans[i])[0]) {
        buttons.push({
          text: Object.keys(buttons_booleans[i])[0],
          action:
            Object.keys(buttons_booleans[i])[0] == "Next"
              ? tour.next
              : Object.keys(buttons_booleans[i])[0] == "Previous"
              ? tour.back
              : Object.keys(buttons_booleans[i])[0] == "Complete"
              ? tour.complete
              : null,
        });
      }
    }
    return buttons;
  }

  //this is since certain variables
  $: {
    let tourStepsArray = [];
    try {
      tourStepsArray = JSON.parse(tourSteps);
    } catch (err) {
      getScreens().then((value) => {
        tourStepsArray = value;
      });
    }
    for (let i = 0; i < tourStepsArray.length; i++) {
      //if buttons exists pass to button handler
      if (tourStepsArray[i].buttons_booleans) {
        tourStepsArray[i].buttons = buttonHandler(
          tourStepsArray[i].buttons_booleans
        );
      }

      tour.addStep(tourStepsArray[i], tourStepsArray.ranking);
    }
  }

  async function startTour() {
    tour.start();
  }

  function setUpStep(componentStep) {
    let newStep = {
      ranking: componentStep.ranking,
      arrow: componentStep.arrow,
      buttons_booleans: [
        { Previous: componentStep.showPrevBtn },
        { Next: componentStep.showNextBtn },
        { Complete: componentStep.showCompleteBtn },
      ],
      modalStepFlag: componentStep.modalStepFlag,
      modalCloseEvent: componentStep.modalCloseEvent,

      //anything above here is only used to for additional data to sort and set buttons etc
      id: componentStep._id,
      text: componentStep.text,
      title: componentStep.title,
      cancelIcon: { enabled: componentStep.cancelIcon, label: "Cancel" },
      attachTo: {
        element: componentStep.modalStepFlag
          ? ":not(.spectrum-Dialog)"
          : "#" + componentStep._id,
        on: componentStep.popposition,
      },
      canClickTarget: componentStep.canClickTarget,
    };
    if (
      componentStep.advanceOn &&
      componentStep.eventTrigger &&
      componentStep.selector &&
      !componentStep.modalStepFlag // next step should only advance if this flag is false
    ) {
      newStep.advanceOn = {
        selector: componentStep.selector,
        event: componentStep.eventTrigger,
      };
    }
    return newStep;
  }

  function findAllByKey(obj, array) {
    for (const each in obj) {
      obj[each].tourName == tourName &&
      obj[each]._component == "plugin/tourstep" &&
      obj[each].ranking > 0
        ? typeof array[obj[each].ranking] === "undefined"
          ? (array[obj[each].ranking] = setUpStep(obj[each]))
          : (error =
              "Component " +
              obj[each]._instanceName +
              " and component " +
              array[obj[each].ranking]._instanceName +
              "  have a duplicate ranking. Please change the ranking based off the order of the steps you would like to appear.")
        : null;

      if (obj[each]._children) {
        //if it has children go deeper
        findAllByKey(obj[each]._children, array);
      }
    }
    return array;
  }

  $: getScreens = async function getScreens() {
    let allComponents = [];
    await API.get({ url: `/api/screens/` }).then((screens) =>
      screens.forEach((screen) => {
        allComponents.push(findAllByKey(screen.props._children, []));
      })
    );
    return allComponents.flat(1);
  };

  onMount(() => {
    callTourAutomatically();
  });

  builderStore.subscribe((value) => (inBuilder = value.inBuilder));

  async function callTourAutomatically() {
    //should not run automatically if inBuilder is false because in the builder it's a little wonky
    if (!inBuilder && autoStart && autoStartField) {
      await tick();
      startTour();
    }
  }
  $: addEventListeners = addTourEventListeners(tour);

  function addTourEventListeners(tour) {
    onCompletion ? tour.on("complete", onCompletion) : null;
    onCancel ? tour.on("cancel", onCancel) : null;
  }

  function refreshTour() {
    getScreens().then((value) => {
      builderStore.actions.updateProp("tourSteps", JSON.stringify(value));
    });
  }

  function checkActiveTour() {
    console.log(Shepherd.activeTour);
  }

  Shepherd.on("show", (e) => {
    if (e.step.options.modalStepFlag) {
      const targetNode = document.getElementsByClassName("modal-container")[0].firstChild.firstChild;
      console.log(targetNode);
      const config = { childList:true};

      const callback = (mutationList, observer) => {
        for (const mutation of mutationList) {
          if (mutation.removedNodes[0].classList.contains("is-open")) {
            tour.next()
            observer.disconnect()
          }
        }

      };
      const observer = new MutationObserver(callback);
      observer.observe(targetNode, config);
    }
  });
</script>

<div use:styleable={$component.styles}>
  {#if error}
    <div class="error">{error}</div>
  {:else}
    <Button cta on:click={startTour}>Start Tour</Button>
    {#if inBuilder}
      <Button cta on:click={refreshTour}>Refresh Tour</Button>
    {/if}
  {/if}
</div>

<style>
  .error {
    color: white;
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
    background: red;
  }
</style>
