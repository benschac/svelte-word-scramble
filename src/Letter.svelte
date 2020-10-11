<script lang="ts">
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let letter: string;
  export let color: string;
  export let idx: number;

  let hovering: boolean = false;

  function enter(e) {
    console.log("hiiii enter", e);
    hovering = true;
  }

  function leave(e) {
    console.log("hiiiii leave", e);
    hovering = false;
  }

  function click() {
    dispatch("letterclick", {
      detail: idx,
    });
  }
</script>

<span style={`color: ${color}`} on:click={click}>
  <slot prop={letter} {hovering}>{letter}</slot>
</span>
