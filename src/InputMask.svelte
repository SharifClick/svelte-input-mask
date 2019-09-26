<script>

  import { onMount } from 'svelte';

  export let mask =  [];
  export let showMask = false;
  export let guide = true;
  export let guideOnOutput = false;
  export let keepCharPositions = true;
  export let placeholder = '' ;
  export let classes = '';

  let inputElement;
  let value;


  onMount(() => {
      let textMaskConfig = {
        inputElement,
        mask,
        showMask,
        guide,
        keepCharPositions
      }
  });

</script>

<div>
  <input class="{classes}" type='text' {placeholder} bind:value bind:this={inputElement}>
</div>