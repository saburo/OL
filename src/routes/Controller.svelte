<script>
  import { open } from "@tauri-apps/api/dialog";
  import Slider from "@bulatdashiev/svelte-slider";

  export let opacity;
  export let rotation;
  export let left;
  export let top;
  export let anc_left;
  export let anc_top;
  export let w;
  export let h;
  export let org_w;
  export let org_h;
  export let img_path;
  export let reset_state;
  export let alt_state;
  export let shift_state;
  export let meta_state;

  async function read_file_contents() {
    try {
      const selected_path = await open({
        multiple: false,
        title: "Open Image File",
      });

      if (selected_path == null) {
        return;
      }

      img_path = selected_path;
    } catch (err) {
      console.error(err);
    }
  }

  function handle_reset_button() {
    reset_state = true;
  }

  function round(n, digits = 2) {
    const d = 10 ^ digits;
    return Math.round(n * d) / d;
  }
</script>

<div class="controller">
  <div class="info mono">
    <div class="info-item">
      <div class="info-item-section">Position</div>
      <div class="info-item-label">X</div>
      <div class="info-item-label2">Y</div>
      <input class="info-item-value" type="text" value={round(left)} />
      <input class="info-item-value2" type="text" value={round(top)} />
    </div>
    <div class="info-item">
      <div class="info-item-section">Size</div>
      <div class="info-item-label">W</div>
      <div class="info-item-label2">H</div>
      <input class="info-item-value" type="text" value={round(w)} />
      <input class="info-item-value2" type="text" value={round(h)} />
    </div>
    <div class="info-item">
      <div class="info-item-section">Original Size</div>
      <div class="info-item-label">W</div>
      <div class="info-item-label2">H</div>
      <input class="info-item-value" type="text" value={round(org_w)} />
      <input class="info-item-value2" type="text" value={round(org_h)} />
    </div>
    <div class="info-item">
      <div class="info-item-section">Scale</div>
      <div class="info-item-label">X</div>
      <div class="info-item-label2">Y</div>
      <input class="info-item-value" type="text" value={round(w / org_w, 4)} />
      <input class="info-item-value2" type="text" value={round(h / org_h, 4)} />
    </div>
    <div
      class="info-item"
      on:click={function () {
        console.log("------");
      }}
    >
      <div class="info-item-section">Anchor</div>
      <div class="info-item-label">X</div>
      <div class="info-item-label2">Y</div>
      <input class="info-item-value" type="text" value={round(anc_left)} />
      <input class="info-item-value2" type="text" value={round(anc_top)} />
    </div>
    <div class="info-item rotation">
      <div class="info-item-section">Rotation</div>
      <input
        style="grid-column: 1; grid-row: 2; align-self: center; justify-self: center; width: 80px"
        type="text"
        value={round((rotation * 180) / Math.PI)}
      />
    </div>
    <div class="info-item opacity">
      <div class="info-item-section">Opacity</div>
      <div class="info-item-value3">
        {round(opacity[0])}
      </div>
      <div class="info-item-slider">
        <Slider bind:value={opacity} min="0" max="1" step="0.001" />
      </div>
    </div>
  </div>
  <div class="status-display">
    <div
      class="status-btn"
      role="none"
      on:click={function () {
        alt_state = false;
        meta_state = false;
        shift_state = false;
      }}
      class:active={!(shift_state || meta_state || alt_state)}
    >
      Scale
    </div>
    <div
      class="status-btn"
      role="none"
      on:click={function () {
        alt_state = false;
        meta_state = false;
        shift_state = !shift_state;
      }}
      class:active={shift_state}
    >
      Move
    </div>
    <div
      class="status-btn"
      role="none"
      on:click={function () {
        alt_state = false;
        shift_state = false;
        meta_state = !meta_state;
      }}
      class:active={meta_state}
    >
      Rotate
    </div>
    <div
      class="status-btn"
      role="none"
      on:click={function () {
        meta_state = false;
        shift_state = false;
        alt_state = !alt_state;
      }}
      class:active={alt_state}
    >
      Anchor
    </div>
  </div>
  <button
    class="controller-button select-image-button"
    on:click={read_file_contents}>Select Image</button
  >
  <button class="controller-button reset-button" on:click={handle_reset_button}
    >Reset</button
  >
</div>

<style>
  .controller {
    display: flex;
    flex-direction: row;
    justify-content: space-evenly;
    align-items: center;
    height: var(--top-offset);
    width: 100%;
    background: #eeeeeeee;
    font-family: monospace;
    user-select: none;
  }

  .info {
    width: 70%;
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    padding: 0;
    font-size: 80%;
  }

  .info-item {
    display: grid;
    width: 105px;
    grid-template-columns: 30px 70px;
    grid-auto-rows: 14px auto auto;
    grid-auto-flow: column;
  }

  .info-item-section {
    grid-column: 1 / 3;
    grid-row: 1;
    text-align: center;
    font-weight: bold;
  }

  .info-item-label,
  .info-item-label2 {
    grid-row: 2;
    grid-column: 1;
    text-align: center;
  }

  .info-item-label2 {
    grid-row: 3;
  }

  .info-item-value {
    grid-column: 2;
  }

  .info-item-value2 {
    grid-column: 2;
  }

  .info input {
    width: 50px;
    height: 12px;
  }

  .info-item.rotation {
    grid-template-columns: 100px;
    grid-auto-rows: 14px auto;
  }

  .info-item.rotation > input {
    grid-column: 1;
    grid-row: 2;
    align-self: center;
    justify-self: center;
    width: 80px;
  }

  .info-item.opacity {
    grid-template-columns: 20%;
    grid-auto-rows: 14px auto auto;
    grid-auto-flow: column;
  }

  .info-item-value3 {
    grid-column: 1;
    grid-row: 2;
    text-align: center;
  }

  .info-item-slider {
    grid-column: 1;
    grid-row: 3;
    width: 100px;
  }

  .status-display {
    display: flex;
    flex-direction: column;
    width: 80px;
    align-content: stretch;
    align-items: stretch;
  }

  .status-btn {
    font-size: 60%;
    text-align: center;
    height: calc(var(--top-offset) / 4);
    line-height: 1.8em;
    background-color: #ffffffee;
    /* padding: 2px; */
  }

  .status-btn.active {
    background-color: #00995566;
    color: white;
  }

  .controller-button {
    width: 120px;
    height: 40px;
    padding: auto 10px;
    border: 1px solid darkblue;
    border-radius: 7px;
    background-color: steelblue;
    font-weight: bold;
    color: white;
    text-transform: uppercase;
  }

  .select-image-button {
    background-color: steelblue;
    border-color: darkblue;
  }

  .reset-button {
    background-color: indianred;
    border-color: maroon;
  }

  /* .opacity-slider { */
  /*   width: 120px; */
  /*   margin: 0; */
  /*   padding: 0; */
  /*   background-color: pink; */
  /* } */
</style>
