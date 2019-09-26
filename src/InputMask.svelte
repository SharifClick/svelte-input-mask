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


  const emptyString = '';
  const strNone = 'none';
  const strObject = 'object';
  const isAndroid = typeof navigator !== 'undefined' && /Android/i.test(navigator.userAgent);
  const defer = typeof requestAnimationFrame !== 'undefined' ? requestAnimationFrame : setTimeout;


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