<script>
  import {fsmDef, events, commands} from "keypad-fsm"
	import { createStateMachine } from "kingly"
	import Machine from 'svelte-machine';
	import Keypad from './Keypad.svelte';

	// Define the rendering props
	let pin;
	$: view = pin ? pin.replace(/\d(?!$)/g, '*') : 'enter your pin';

	// Destructure the machine interface (event/command)
	const [NUM_KEY_CLICKED, SUBMIT_CLICKED, CLEAR_CLICKED, START] = events;
	const [COMMAND_RENDER, SUBMIT] = commands ;

  // Create the machine
  const fsm = createStateMachine(fsmDef, {debug: {console}});

  // Command handlers
  const commandHandlers = {
    [SUBMIT] : function (dispatch, params, effectHandlers) {
      alert('SUBMIT:' + params.pin)
    },
    [COMMAND_RENDER]: function (dispatch, params, effectHandlers) {
      if ('pin' in params) {pin = params.pin}
    },
  }
  const effectHandlers = {};
  const initEvent = {[START]: void 0}
</script>

<Machine let:dispatch={dispatch} {commandHandlers} {fsm} {initEvent} {effectHandlers}>
  <h1 style="color: {pin ? '#333' : '#ccc'}">{view}</h1>
  <Keypad
    on:clear="{e => dispatch({CLEAR_CLICKED: void 0})}"
    on:pin="{e => dispatch({NUM_KEY_CLICKED: e.detail})}"
    on:submit="{e => dispatch({SUBMIT_CLICKED: void 0})}" />
</Machine>
