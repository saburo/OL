<script>
  import Slider from "@bulatdashiev/svelte-slider";

  export let img_info;
  export let anchor;
  export let opacity;

  let info_data;

  $: {
    info_data = [
      {
        section: "Position",
        label: ["X", "Y"],
        value: [img_info["left"], img_info["top"]],
        digits: 2,
      },
      {
        section: "Size",
        label: ["W", "H"],
        value: [img_info["w"], img_info["h"]],
        digits: 2,
      },
      {
        section: "Original Size",
        label: ["W", "H"],
        value: [img_info["org_w"], img_info["org_h"]],
        digits: 2,
      },
      {
        section: "Scale",
        label: ["X", "Y"],
        value: [
          img_info["w"] / img_info["org_w"],
          img_info["h"] / img_info["org_h"],
        ],
        digits: 4,
      },
      {
        section: "Anchor",
        label: ["X", "Y"],
        value: [
          anchor.left + anchor.size / 2 - img_info["left"],
          anchor.top - 65 + anchor.size / 2 - img_info["top"],
        ],
        digits: 2,
      },
    ];
  }

  function round(n, digits = 2) {
    const d = Math.pow(10, digits);
    return Math.round(n * d) / d;
  }
</script>

<div class="info mono">
  {#each info_data as { section, label, value, digits }}
    <div class="info-item">
      <div class="info-item-section">{section}</div>
      <div class="info-item-label">{label[0]}</div>
      <div class="info-item-label2">{label[1]}</div>
      <input
        class="info-item-value"
        tabindex="-1"
        type="text"
        value={round(value[0], digits)}
        readonly
      />
      <input
        tabindex="-1"
        class="info-item-value2"
        type="text"
        value={round(value[1], digits)}
        readonly
      />
    </div>
  {/each}
  <div class="info-item rotation">
    <div class="info-item-section">Rotation</div>
    <input
      style="grid-column: 1; grid-row: 2; align-self: center; justify-self: center; width: 80px"
      type="text"
      tabindex="-1"
      value={round((img_info.rotation * 180) / Math.PI)}
      readonly
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
    grid-auto-rows: 25px auto auto;
    grid-auto-flow: column;
  }

  .info-item-section {
    grid-column: 1 / 3;
    grid-row: 1;
    text-align: center;
    line-height: 2em;
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
