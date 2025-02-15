<script lang="ts">
  import GridCell from "./GridCell.svelte";

  import { cubicOut } from "svelte/easing";
  import { slide } from "svelte/transition";
  import { COLS, make_picked, recipe, ROWS } from "./data";

  // slide but horizontal
  function horizontalSlide(node: HTMLElement, { delay = 0, duration = 400, easing: easing$1 = cubicOut } = {}) {
    const style = getComputedStyle(node);
    const opacity = +style.opacity;
    const width = parseFloat(style.width);
    const padding_left = parseFloat(style.paddingLeft);
    const padding_right = parseFloat(style.paddingRight);
    const margin_left = parseFloat(style.marginLeft);
    const margin_right = parseFloat(style.marginRight);
    const border_left_width = parseFloat(style.borderLeftWidth);
    const border_right_width = parseFloat(style.borderRightWidth);
    return {
      delay,
      duration,
      easing: easing$1,
      css: (t: number) => 'overflow: hidden;' +
        `opacity: ${Math.min(t * 20, 1) * opacity};` +
        `width: ${t * width}px;` +
        `padding-left: ${t * padding_left}px;` +
        `padding-right: ${t * padding_right}px;` +
        `margin-left: ${t * margin_left}px;` +
        `margin-right: ${t * margin_right}px;` +
        `border-left-width: ${t * border_left_width}px;` +
        `border-right-width: ${t * border_right_width}px;`
    };
  }

  let transition: any = slide;

  // https://dev.to/samanthaming/how-to-deep-clone-an-array-in-javascript-3cig
  const clone = (items: any[]): any[] => items.map(item => Array.isArray(item) ? clone(item) : item);

  $: if ($recipe.length != $ROWS || $recipe[0].length != $COLS) {
    let old: number[][] = $recipe;
    recipe.set(make_picked());

    // Copy old state
    if (old) {
      for (let r = 0; r < $ROWS; r++) {
        for (let c = 0; c < $COLS; c++) {
          if (old[r] && old[r][c] >= 0) {
            $recipe[r][c] = old[r][c];
          }
        }
      }
      recipe.set($recipe); 
    }
  }
</script>

<div class="workspace">
  <div class="grid">
    {#each {length: $ROWS} as _, r}
      <div class="row">
        {#each {length: $COLS} as _, c}
          <GridCell transition={transition} row={r} col={c}></GridCell>
        {/each}
      </div>
    {/each}
  </div>
</div>

<div class="config">
  <div class="input-row">
    <span class="text">Width</span>
    <input type="range" min="2" max="7" step="1" bind:value={$COLS} on:focus={() => {transition = horizontalSlide}}/>
  </div>
  <div class="input-row">
    <span class="text">Height</span>
    <input type="range" min="2" max="7" step="1" bind:value={$ROWS} on:focus={() => {transition = slide}}/>
  </div>
</div>

<style>
  .workspace {
    background-color: #fafafa;
    width: calc(100vw - 10vh);
    position: absolute;
    right: 0;
    top: 0;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .grid {
    display: flex;
    flex-direction: column;
  }

  .row {
    display: flex;
    flex-direction: row;
  }

  .input-row {
    display: flex;
    flex-direction: row;
    margin-bottom: 5px;
  }

  .config {
    position: absolute;
    bottom: 0;
    right: 0;
    display: flex;
    flex-direction: column;
    align-items: end;
  }

  .text {
    font-family: Arial, Helvetica, sans-serif;
    margin-right: 5px;
  }
</style>