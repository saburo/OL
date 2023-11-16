<script>
  import { onMount } from "svelte";
  import Controller from "./Controller.svelte";
  import ImageCanvas from "./ImageCanvas.svelte";

  let setAnchorPos;
  let setMainImage;

  // TODO
  //  [x] scale
  //  [x] scale with anchor
  //  [x] translate
  //  [x] rotate
  //  [x] reselecting anchor point
  //  [ ] click on BadgerScope window
  //
  // Tool Shortcut Keys
  //  m: Move image
  //  s: Scale image
  //  r: Rotate image
  //  a: Change anchor position
  //
  // Temporary Tool Change)
  //  shift: Scale image
  //  meta (command): Rotate image
  //  alt/option: Change anchor position
  //
  // Shortcut Keys
  //  space: Toggle opacity (current <- -> 0)
  //  tab: Hide image to click BadgerScope window

  // const controller_height = 65;

  let img_info = {
    top: 0,
    left: 0,
    w: 0,
    h: 0,
    org_w: 0,
    org_h: 0,
    scale_x: 1,
    scale_y: 1,
    rotation: 0,
  };

  let anchor = {
    top: 0,
    left: 0,
    size: 10,
  };

  let opacity = [1, 1];
  let tmp_opacity = 1;
  let img_path = "";
  let g_img = "";

  let curr_tool = "move";
  let prev_tool = "move";
  let cursor = "default";

  let is_dragging;

  let reset_state = false;

  const win_width = 1200;
  const win_height = 800;
  const win_hide_height = 94;

  $: {
    if (reset_state) {
      init();
      reset_state = false;
    }
  }

  $: cursor = get_cursor(curr_tool);

  onMount(async () => {
    const { appWindow, LogicalSize } = window.__TAURI__.window;
    init();

    await appWindow.setAlwaysOnTop(true);

    // For transparent window in Windows
    // Window will be transparent after resizeing
    await appWindow.setSize(new LogicalSize(0, 0));
    await appWindow.setSize(new LogicalSize(win_width, win_height));

    window.addEventListener("keydown", async (e) => {
      prev_tool = curr_tool;
      switch (e.key) {
        case " ": // ignore space key
          return;

        case "Tab":
          // await toggle_window();
          const size = await appWindow.innerSize();
          if (size.height > 200) {
            await appWindow.setSize(
              new LogicalSize(win_width, win_hide_height)
            );
          } else {
            await appWindow.setSize(new LogicalSize(win_width, win_height));
          }
          break;

        case "Shift":
        case "s":
          curr_tool = "scale";
          break;
        case "Alt":
        case "a":
          curr_tool = "anchor";
          break;
        case "Meta":
        case "r":
          curr_tool = "rotate";
          break;
        default:
        case "m":
          curr_tool = "move";
          break;
      }
    });

    window.addEventListener("keyup", async (e) => {
      if (["Shift", "Alt", "Meta"].includes(e.key)) {
        curr_tool = prev_tool;
      }

      if (e.key === " ") {
        toggle_opacity();
      }
    });
  });

  function init(w_img = true) {
    img_info = {
      top: 0,
      left: 0,
      w: 0,
      h: 0,
      org_w: 0,
      org_h: 0,
      scale_x: 1,
      scale_y: 1,
      rotation: 0,
    };

    is_dragging = false;
    opacity = [1, 1];

    if (w_img) {
      setMainImage(img_path);
    }

    setAnchorPos(0, 0);
  }

  // async function toggle_window() {
  //   const size = await appWindow.innerSize();
  //   if (size.height > 200) {
  //     await appWindow.setSize(new LogicalSize(win_width, win_hide_height));
  //   } else {
  //     await appWindow.setSize(new LogicalSize(win_width, win_height));
  //   }
  // }

  function get_cursor(ct) {
    let c;
    switch (ct) {
      case "anchor":
        c = "crosshair";
        break;
      case "move":
        c = "move";
        break;
      case "rotate":
        c = "cell";
        break;
      default:
        c = "nwse-resize";
        break;
    }
    return c;
  }

  function toggle_opacity() {
    if (opacity[0] > 0) {
      tmp_opacity = [opacity[0], opacity[1]];
      opacity = [0, 1];
    } else {
      opacity = [tmp_opacity[0], tmp_opacity[1]];
    }
  }
</script>

<section>
  <div class="anchor" style="left: {anchor.left}px; top: {anchor.top}px;" />
  <Controller
    bind:img_path
    bind:reset_state
    bind:anchor
    bind:curr_tool
    bind:prev_tool
    bind:img_info
    bind:opacity
  />
  <ImageCanvas
    bind:img_info
    bind:img_path
    bind:opacity
    bind:cursor
    bind:g_img
    bind:is_dragging
    bind:curr_tool
    bind:anchor
    bind:set_anchor_pos={setAnchorPos}
    bind:set_image={setMainImage}
  />
</section>

<style lang="postcss">
  :root {
    --top-offset: 65px;
    --anchor-size: 10px;
    --mybg: #ffffff00;
  }
  .anchor {
    position: absolute;
    width: var(--anchor-size);
    height: var(--anchor-size);
    background-color: #fc0fc0;
    border-radius: 50%;
    z-index: 100;
  }
</style>
