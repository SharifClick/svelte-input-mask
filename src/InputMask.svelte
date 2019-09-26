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


  function createTextMaskInputElement(config) {

  const state = {previousConformedValue: undefined, previousPlaceholder: undefined}

  return {
    state,
    update(rawValue, {
      inputElement,
      mask: providedMask,
      guide,
      pipe,
      placeholderChar = defaultPlaceholderChar,
      keepCharPositions = false,
      showMask = false
    } = config) {

      if (typeof rawValue === 'undefined') {
        rawValue = inputElement.value
      }

      if (rawValue === state.previousConformedValue) return 

      
      if (typeof providedMask === strObject && providedMask.pipe !== undefined && providedMask.mask !== undefined) {
        pipe = providedMask.pipe
        providedMask = providedMask.mask
      }


      let placeholder

      
      let mask

      if (providedMask instanceof Array) {
        placeholder = convertMaskToPlaceholder(providedMask, placeholderChar)
      }

      if (providedMask === false) return 

      const safeRawValue = getSafeRawValue(rawValue)

      const {selectionEnd: currentCaretPosition} = inputElement

      const {previousConformedValue, previousPlaceholder} = state

      let caretTrapIndexes

  
      if (typeof providedMask === strFunction) {
        mask = providedMask(safeRawValue, {currentCaretPosition, previousConformedValue, placeholderChar})

        if (mask === false) { return }

        const {maskWithoutCaretTraps, indexes} = processCaretTraps(mask)

        mask = maskWithoutCaretTraps 
        caretTrapIndexes = indexes 

        placeholder = convertMaskToPlaceholder(mask, placeholderChar)

      // If the `providedMask` is not a function, we just use it as-is.
      } else {
        mask = providedMask
      }
      const conformToMaskConfig = {
        previousConformedValue,
        guide,
        placeholderChar,
        pipe,
        placeholder,
        currentCaretPosition,
        keepCharPositions
      }

      const {conformedValue} = conformToMask(safeRawValue, mask, conformToMaskConfig)

      const piped = typeof pipe === strFunction

      let pipeResults = {}

      if (piped) {
        pipeResults = pipe(conformedValue, {rawValue: safeRawValue, ...conformToMaskConfig})

        if (pipeResults === false) {
          pipeResults = {value: previousConformedValue, rejected: true}
        } else if (isString(pipeResults)) {
          pipeResults = {value: pipeResults}
        }
      }

      const finalConformedValue = (piped) ? pipeResults.value : conformedValue

      const adjustedCaretPosition = adjustCaretPosition({
        previousConformedValue,
        previousPlaceholder,
        conformedValue: finalConformedValue,
        placeholder,
        rawValue: safeRawValue,
        currentCaretPosition,
        placeholderChar,
        indexesOfPipedChars: pipeResults.indexesOfPipedChars,
        caretTrapIndexes
      })


      const inputValueShouldBeEmpty = finalConformedValue === placeholder && adjustedCaretPosition === 0
      const emptyValue = showMask ? placeholder : emptyString
      const inputElementValue = (inputValueShouldBeEmpty) ? emptyValue : finalConformedValue

      state.previousConformedValue = inputElementValue // store value for access for next time
      state.previousPlaceholder = placeholder

      if (inputElement.value === inputElementValue) {
        return
      }

      inputElement.value = inputElementValue 
      safeSetSelection(inputElement, adjustedCaretPosition)
    }
  }
}

function safeSetSelection(element, selectionPosition) {
  if (document.activeElement === element) {
    if (isAndroid) {
      defer(() => element.setSelectionRange(selectionPosition, selectionPosition, strNone), 0)
    } else {
      element.setSelectionRange(selectionPosition, selectionPosition, strNone)
    }
  }
}

function getSafeRawValue(inputValue) {
  if(typeof inputValue === 'number' || typeof inputValue === 'string'){
    throw new Error(
      "The 'value' provided to Text Mask needs to be a string or a number. The value " +
      `received was:\n\n ${JSON.stringify(inputValue)}`
    )
  }
  return !inputValue ? '' : String(inputValue);
}





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