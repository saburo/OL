<script>
  import Slider from "@bulatdashiev/svelte-slider";
  export let img_info;
  export let anc_top;
  export let anc_left;
  export let opacity;

  function round(n, digits = 2) {
    const d = Math.pow(10, digits);
    return Math.round(n * d) / d;
  }
</script>

<div class="info mono">
  <div class="info-item">
    <div class="info-item-section">Position</div>
    <div class="info-item-label">X</div>
    <div class="info-item-label2">Y</div>
    <input class="info-item-value" type="text" value={round(img_info.left)} />
    <input class="info-item-value2" type="text" value={round(img_info.top)} />
  </div>
  <div class="info-item">
    <div class="info-item-section">Size</div>
    <div class="info-item-label">W</div>
    <div class="info-item-label2">H</div>
    <input class="info-item-value" type="text" value={round(img_info.w)} />
    <input class="info-item-value2" type="text" value={round(img_info.h)} />
  </div>
  <div class="info-item">
    <div class="info-item-section">Original Size</div>
    <div class="info-item-label">W</div>
    <div class="info-item-label2">H</div>
    <input class="info-item-value" type="text" value={round(img_info.org_w)} />
    <input class="info-item-value2" type="text" value={round(img_info.org_h)} />
  </div>
  <div class="info-item">
    <div class="info-item-section">Scale</div>
    <div class="info-item-label">X</div>
    <div class="info-item-label2">Y</div>
    <input
      class="info-item-value"
      type="text"
      value={round(img_info.w / img_info.org_w, 4)}
    />
    <input
      class="info-item-value2"
      type="text"
      value={round(img_info.h / img_info.org_h, 4)}
    />
  </div>
  <div class="info-item">
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
      value={round((img_info.rotation * 180) / Math.PI)}
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

<style lang="postcss">
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
    height: 15px;
    padding: 0 2px;
    border-radius: 3px;
    border: 1px solid #cccccc;
    color: gray;
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
    grid-column: 1 / 3;
    grid-row: 2;
    text-align: center;
    margin: auto;
  }

  .info-item-slider {
    grid-column: 1;
    grid-row: 3;
    width: 100px;
  }
</style>
