<script>
  import { onMount, onDestroy } from "svelte";
  import { convertFileSrc } from "@tauri-apps/api/tauri";
  import Controller from "./Controller.svelte";

  // TODO
  //  [x] scale
  //  [x] scale with anchor
  //  [x] translate
  //  [x] rotate
  //  [x] reselecting anchor point
  //
  // Click
  //  w/ Alt key - Change anchor location
  //
  // Drag
  //  w/o keys - Ccale image
  //  w/ Shift key - Move/shift image
  //  w/ Command(meta) key - Rotate image

  let box;

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

  let top = 0;
  let left = 0;
  let w = 0;
  let h = 0;
  let org_w = 0;
  let org_h = 0;
  let anchor_size;
  let is_dragging;
  let anc_top = 0;
  let anc_left = 0;
  let scale = { x: 1, y: 1 };
  let rotation = 0;
  let opacity = [1, 1];
  let tmp_opacity = 1;
  let img_path = "";
  let g_img = "";
  let curr_tool = "move";
  let prev_tool = "move";
  let cursor = "default";

  let reset_state = false;

  // $: cursor = get_cursor();

  $: {
    if (reset_state) {
      init();
      reset_state = false;
    }
  }

  $: {
    cursor = get_cursor(curr_tool);
    // console.log("cursor", curr_tool);
  }

  $: {
    set_image(img_path);
  }

  onMount(async () => {
    init();
    window.addEventListener("keydown", (e) => {
      prev_tool = curr_tool;
      // console.log(e.key);
      switch (e.key) {
        case "Shift":
        case "m":
          curr_tool = "move";
          break;
        case "Alt":
        case "a":
          curr_tool = "anchor";
          break;
        case "Meta":
        case "r":
          curr_tool = "rotate";
          break;
        case "s":
        default:
          curr_tool = "scale";
          break;
      }
    });
    window.addEventListener("keyup", (e) => {
      if (["Shift", "Alt", "Meta"].includes(e.key)) {
        curr_tool = prev_tool;
      }

      if (e.key === " ") {
        toggle_opacity();
      }
    });
  });

  onDestroy(async () => {
    // window.removeEventListener("keyup");
    // window.removeEventListener("keydown");
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

    top = 0;
    left = 0;
    is_dragging = false;
    anchor_size = 10;
    scale = { x: 1, y: 1 };
    rotation = 0;
    opacity = [1, 1];
    box = document.querySelector(".image-canvas");
    set_anchor_pos(0, 0);

    if (w_img) {
      set_image(img_path);
    }
  }

  function mouse_down(e) {
    if (!e.alttKey) {
      is_dragging = true;
    }
  }

  function mouse_moving(e) {
    if (!is_dragging) {
      return;
    }

    switch (curr_tool) {
      case "move":
        shifting(e);
        break;

      case "rotate":
        rotating(e);
        break;

      case "scale":
        scaling(e);
        break;

      default:
        break;
    }
  }

  function mouse_up(e) {
    is_dragging = false;
  }

  function mouse_click(e) {
    if (curr_tool !== "anchor") {
      return;
    }
    const pos = get_mouse_pos(e);

    set_anchor_pos(pos[0], pos[1]);
  }

  function get_mouse_pos(e) {
    return [
      e.clientX - box.offsetLeft + window.pageXOffset,
      e.clientY - box.offsetTop + window.pageYOffset,
    ];
  }

  function get_anchor_pos() {
    return [
      anc_left + anchor_size / 2 - box.offsetLeft,
      anc_top + anchor_size / 2 - box.offsetTop,
    ];
  }

  function set_anchor_pos(x, y) {
    anc_left = x - anchor_size / 2 + box.offsetLeft;
    anc_top = y - anchor_size / 2 + box.offsetTop;
  }

  function shifting(e) {
    img_info.left += e.movementX;
    img_info.top += e.movementY;
    img_info = img_info;

    left += e.movementX;
    top += e.movementY;
    anc_left += e.movementX;
    anc_top += e.movementY;
  }

  function scaling(e) {
    const pos = get_mouse_pos(e);
    const anc_pos = get_anchor_pos();

    img_info.scale_x = 1 + e.movementX / (pos[0] - anc_pos[0]);
    img_info.scale_y = 1 + e.movementY / (pos[1] - anc_pos[1]);

    scale.x = 1 + e.movementX / (pos[0] - anc_pos[0]);
    scale.y = 1 + e.movementY / (pos[1] - anc_pos[1]);

    img_info.w *= img_info.scale_x;
    img_info.h *= img_info.scale_y;

    w *= scale.x;
    h *= scale.y;

    img_info.left -= (anc_pos[0] - img_info.left) * (img_info.scale_x - 1);
    img_info.top -= (anc_pos[1] - img_info.top) * (img_info.scale_y - 1);

    left -= (anc_pos[0] - left) * (scale.x - 1);
    top -= (anc_pos[1] - top) * (scale.y - 1);

    img_info = img_info;
  }

  function rotating(e) {
    const pos = get_mouse_pos(e);
    const anc_pos = get_anchor_pos();

    const ang1 = Math.atan2(pos[1] - anc_pos[1], pos[0] - anc_pos[0]);
    const ang2 = Math.atan2(
      pos[1] + e.movementY - anc_pos[1],
      pos[0] + e.movementX - anc_pos[0]
    );
    const theta = ang2 - ang1;

    const ctr_x = left + w / 2;
    const ctr_y = top + h / 2;

    const x = anc_pos[0] - ctr_x;
    const y = anc_pos[1] - ctr_y;

    const s = Math.sin(theta);
    const c = Math.cos(theta);

    const X = x * c - y * s;
    const Y = y * c + x * s;

    img_info.rotation += theta;
    img_info.left += x - X;
    img_info.top += y - Y;

    img_info = img_info;

    rotation += theta;
    left += x - X;
    top += y - Y;
  }

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

  function set_image(img_path) {
    if (img_path == "") {
      return;
    }

    const bw = box.offsetWidth;
    const bh = box.offsetHeight;

    const img = new Image();

    img.onload = () => {
      g_img = img.src;

      img_info.org_w = img.width;
      img_info.org_h = img.height;

      org_w = img.width;
      org_h = img.height;
      const r2 = img.height / img.width;

      if (bw < bh / r2) {
        w = bw;
        h = w * r2;
        img_info.w = bw;
        img_info.h = bw * r2;
      } else {
        h = bh;
        w = h / r2;
        img_info.h = bh;
        img_info.w = h / r2;
      }

      img_info = img_info;
    };

    img.src = convertFileSrc(img_path);
  }

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
      set_image(selected_path);
    } catch (err) {
      console.error(err);
    }
  }
</script>

<svelte:window on:mouseup={mouse_up} on:mousemove={mouse_moving} />

<section>
  <div class="anchor" style="left: {anc_left}px; top: {anc_top}px;" />
  <Controller
    bind:opacity
    bind:img_path
    bind:reset_state
    bind:rotation
    bind:top
    bind:left
    bind:w
    bind:h
    bind:anc_top
    bind:anc_left
    bind:org_w
    bind:org_h
    bind:curr_tool
    bind:prev_tool
    bind:img_info
  />
  <div class="image-canvas">
    <div
      class="image-frame"
      on:click={mouse_click}
      on:mousedown={mouse_down}
      role="none"
      style="
      top: {top}px; 
      left: {left}px; 
      width: {w}px; 
      height: {h}px; 
      rotate: {rotation}rad;
      opacity: {opacity[0]};
      background-image: url({g_img});
      cursor: {cursor};
      "
    />
  </div>
</section>

<style lang="postcss">
  :root {
    --top-offset: 65px;
  }
  .image-canvas {
    position: absolute;
    top: var(--top-offset);
    left: 0;
    width: 100%;
    height: calc(100% - var(--top-offset));
    background-color: #ffffff03;
    overflow: hidden;
    user-select: none;
  }

  .image-frame {
    position: relative;
    /* border: 1px solid #ff0000; */
    background-repeat: no-repeat;
    background-size: 100% 100%;
    user-select: none;
  }

  .anchor {
    position: absolute;
    width: 10px;
    height: 10px;
    /* background-color: steelblue; */
    background-color: #fc0fc0;
    border-radius: 50%;
    z-index: 100;
  }
</style>
