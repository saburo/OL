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

      case "shift": // move
        curr_tool = "move";
        break;

      default: // scale
        curr_tool = "scale";
    }
  }
</script>

<div class="tool-display">
  <div
    class="tool-btn"
    role="none"
    on:click={() => {
      update_tool_btn_status("shift");
    }}
    class:active={curr_tool === "move"}
  >
    Move (M)
  </div>
  <div
    class="tool-btn"
    role="none"
    on:click={() => {
      update_tool_btn_status();
    }}
    class:active={curr_tool === "scale"}
  >
    Scale (S)
  </div>
  <div
    class="tool-btn"
    role="none"
    on:click={() => {
      update_tool_btn_status("meta");
    }}
    class:active={curr_tool === "rotate"}
  >
    Rotate (R)
  </div>
  <div
    class="tool-btn"
    role="none"
    on:click={() => {
      update_tool_btn_status("alt");
    }}
    class:active={curr_tool === "anchor"}
  >
    Anchor (A)
  </div>
</div>

<style lang="postcss">
  .tool-display {
    display: flex;
    flex-direction: column;
    width: 80px;
    align-content: stretch;
    align-items: stretch;
    cursor: pointer;
  }

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
