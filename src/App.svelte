<script>
	import { createStateMachine, COMMAND_RENDER, NO_OUTPUT } from "kingly"
	import Keypad from './Keypad.svelte';
	import Machine from './Machine.svelte';

	// Define the rendering props
	let pin;
	$: view = pin ? pin.replace(/\d(?!$)/g, '"') : 'enter your pin';
//	$: console.warn('pin', view(pin))
	$: console.warn('pin', pin)

// Define the state machine
// State monikers
const INIT = "OFF";
const PIN = "PIN";
const NO_PIN = "NO_PIN";
const DONE = "DONE";

// Event monikers
const START = "START";
const CLEAR_CLICKED = "CLEAR_CLICKED";
const SUBMIT_CLICKED = "SUBMIT_CLICKED";
const NUM_KEY_CLICKED = "NUM_KEY_CLICKED";

// Commands
const SUBMIT = "SUBMIT";

// State update
// Basically {a, b: {c, d}}, [{b:{e}]} -> {a, b:{e}}
// All Object.assign caveats apply
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
function updateState(extendedState, extendedStateUpdates) {
  const extendedStateCopy = Object.assign({}, extendedState);
  return extendedStateUpdates.reduce((acc, x) => Object.assign(acc, x), extendedStateCopy);
}

const events = [NUM_KEY_CLICKED, SUBMIT_CLICKED, CLEAR_CLICKED, START];
const states = {
  [INIT]: "",
  [PIN]: "",
  [NO_PIN]: "",
  [DONE]: "",
};
const initialControlState = INIT;
const initialExtendedState = {
  pin: ""
};
const transitions = [
  { from: INIT, event: START, to: NO_PIN, action: displayAndEmitNoPin },
  { from: NO_PIN, event: NUM_KEY_CLICKED, to: PIN, action: displayAndEmitPin },
  { from: PIN, event: CLEAR_CLICKED, to: NO_PIN, action: displayAndEmitNoPin },
  { from: PIN, event: NUM_KEY_CLICKED, to: PIN, action: displayAndEmitPin },
  { from: PIN, event: SUBMIT_CLICKED, to: DONE, action: displayAndSubmitPin },
];

// Actions
function displayAndEmitNoPin(extendedState, eventData, settings) {
  return {
    updates: [{pin:""}],
    outputs: [{
      command: COMMAND_RENDER,
      params: { pin:"" }
    }],
  }
}

function displayAndEmitPin(extendedState, eventData, settings) {
  const { pin } = extendedState;
  const digit = eventData;
  const newPin = pin + digit;
	
  return {
    updates: [{pin:newPin}],
    outputs: [{
      command: COMMAND_RENDER,
      params: { pin:newPin }
    }],
  }
}

function displayAndSubmitPin(extendedState, eventData, settings) {
  const { pin } = extendedState;
	
  return {
    updates: [],
    outputs: [{
      command: SUBMIT,
      params: { pin }
    }],
  }
}

const fsmDef = {
  initialControlState,
  initialExtendedState,
  states,
  events,
  transitions,
  updateState
};

// Create the machine
const fsm = createStateMachine(fsmDef, {debug: {console}});

// Command handlers
const commandHandlers = {
	[SUBMIT] : function(next, params, effectHandlers){
			alert('SUBMIT:'+ params.pin) 
		},
	[COMMAND_RENDER]: function(next, params, effectHandlers){
			if ('pin' in params) {pin = params.pin} 
		},
}
const effectHandlers = {
	};
const initEvent = {[START]: void 0}
</script>

<Machine let:dispatch={dispatch} commandHandlers={commandHandlers} fsm={fsm} initEvent={initEvent} {effectHandlers}>
  <h1 style="color: {pin ? '#333' : '#ccc'}">{view}</h1>
  <Keypad on:clear="{e => dispatch({CLEAR_CLICKED: void 0})}" on:pin="{e => dispatch({NUM_KEY_CLICKED: e.detail})}" on:submit="{e => dispatch({SUBMIT_CLICKED: void 0})}" />
</Machine>
