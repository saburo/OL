<script>
  import { open } from "@tauri-apps/api/dialog";
  import ImageInfo from "./ImageInfo.svelte";
  import ToolInfo from "./ToolInfo.svelte";

  export let opacity;
  export let anchor;
  export let img_path;
  export let reset_state;
  export let curr_tool;
  export let prev_tool;

  export let img_info;

  async function select_image_file() {
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
  <ImageInfo bind:img_info bind:opacity bind:anchor />
  <ToolInfo bind:curr_tool bind:prev_tool />
  <button
    class="rounded-md bg-blue-500 hover:bg-blue-800 text-white sm:px-5 sm:py-3 px-4 py-2 uppercase"
    on:click={select_image_file}
    >Image…
  </button>
  <button
    class="rounded-md bg-rose-500 hover:bg-rose-800 text-white sm:px-8 sm:py-3 px-4 py-2 uppercase"
    on:click={handle_reset_button}>Reset</button
  >
</div>

<style lang="postcss">
  .controller {
    position: absolute;
    top: 0; /* calc(100vh - var(--top-offset)); */
    height: var(--top-offset);
    width: 100vw;

    display: flex;
    flex-direction: row;
    justify-content: space-evenly;
    align-items: center;
    background: #eeeeee;
    font-family: monospace;
    user-select: none;
  }
</style>
