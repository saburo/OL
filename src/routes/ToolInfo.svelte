<script>
  export let curr_tool;
  export let prev_tool;

  function update_tool_btn_status(btn) {
    prev_tool = curr_tool;

    switch (btn) {
      case "meta": // rotate
        curr_tool = "rotate";
        break;

      case "alt": // anchor
        curr_tool = "anchor";
        break;

      case "shift": // scal
        curr_tool = "scale";
        break;

      case "": // move
        curr_tool = "move";
    }
  }

  const tools = [
    { label: "Move (m)", key: "", tool: "move" },
    { label: "Scale (s)", key: "shift", tool: "scale" },
    { label: "Rotate (r)", key: "meta", tool: "rotate" },
    { label: "Anchor (a)", key: "alt", tool: "anchor" },
  ];
</script>

<div class="grid grid-cols-2 gap-x-2 gap-y-1 mr-2">
  {#each tools as tool}
    <div
      class="tool-btn w-16"
      role="none"
      on:click={() => {
        update_tool_btn_status(tool.key);
      }}
      class:active={curr_tool === tool.tool}
    >
      {tool.label}
    </div>
  {/each}
</div>

<!-- // .tool-display { -->
<!-- //   display: ; -->
<!-- //   flex-direction: row; -->
<!-- //   flex-wrap: wrap; -->
<!-- //   width: 200px; -->
<!-- //   align-content: stretch; -->
<!-- //   align-items: stretch; -->
<!-- //   cursor: pointer; -->
<!-- // } -->
<style lang="postcss">
  .tool-btn {
    font-size: 60%;
    text-align: center;
    height: calc(var(--top-offset) / 4);
    line-height: 1.8em;
    background-color: #ffffff00;
  }

  .tool-btn.active {
    background-color: #00995566;
    color: white;
  }
</style>
