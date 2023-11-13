<script>
  import { open } from "@tauri-apps/api/dialog";
  import ImageInfo from "./ImageInfo.svelte";
  import ToolInfo from "./ToolInfo.svelte";

  export let opacity;
  export let anc_left;
  export let anc_top;
  export let img_path;
  export let reset_state;
  export let curr_tool;
  export let prev_tool;

  export let img_info;

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
</script>

<div class="controller">
  <ImageInfo bind:img_info bind:opacity bind:anc_top bind:anc_left />
  <ToolInfo bind:curr_tool bind:prev_tool />
  <button
    class="rounded-md bg-blue-500 hover:bg-blue-800 text-white sm:px-5 sm:py-3 px-4 py-2 uppercase"
    on:click={read_file_contents}
    >Select Image…
  </button>
  <button
    class="rounded-md bg-rose-500 hover:bg-rose-800 text-white sm:px-8 sm:py-3 px-4 py-2 uppercase"
    on:click={handle_reset_button}>Reset</button
  >
</div>

<style lang="postcss">
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
</style>
