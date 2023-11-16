<script>
  import { onMount } from "svelte";
  import { convertFileSrc } from "@tauri-apps/api/tauri";

  export let img_info;
  export let cursor;
  export let opacity;
  export let g_img;
  export let img_path;
  export let is_dragging;
  export let curr_tool;
  export let anchor;

  let box;

  $: {
    set_image(img_path);
  }

  onMount(() => {
    if (box == undefined) {
      box = window.document.querySelector(".image-canvas");
    }
  });

  export function set_image(img_path) {
    if (img_path == "" || box == undefined) return;

    set_anchor_pos(0, 0);

    const bw = box.offsetWidth;
    const bh = box.offsetHeight;

    const img = new Image();

    img.onload = () => {
      g_img = img.src;

      img_info.org_w = img.width;
      img_info.org_h = img.height;

      const r2 = img.height / img.width;

      if (bw < bh / r2) {
        img_info.w = bw;
        img_info.h = bw * r2;
      } else {
        img_info.h = bh;
        img_info.w = img_info.h / r2;
      }
      img_info.left = 0;
      img_info.top = 0;
      img_info.scale_x = 1;
      img_info.scale_y = 1;
      img_info.rotation = 0;
    };

    img.src = convertFileSrc(img_path);
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
    return [e.clientX - box.offsetLeft, e.clientY - box.offsetTop];
  }

  function shifting(e) {
    img_info.left += e.movementX;
    img_info.top += e.movementY;
    img_info = img_info;

    anchor.left += e.movementX;
    anchor.top += e.movementY;
  }

  function scaling(e) {
    const pos = get_mouse_pos(e);
    const anc_pos = get_anchor_pos();

    img_info.scale_x = 1 + e.movementX / (pos[0] - anc_pos[0]);
    img_info.scale_y = 1 + e.movementY / (pos[1] - anc_pos[1]);

    img_info.w *= img_info.scale_x;
    img_info.h *= img_info.scale_y;

    img_info.left -= (anc_pos[0] - img_info.left) * (img_info.scale_x - 1);
    img_info.top -= (anc_pos[1] - img_info.top) * (img_info.scale_y - 1);

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

    const ctr_x = img_info.left + img_info.w / 2;
    const ctr_y = img_info.top + img_info.h / 2;

    const x = anc_pos[0] - ctr_x;
    const y = anc_pos[1] - ctr_y;

    const sin = Math.sin(theta);
    const cos = Math.cos(theta);

    const X = x * cos - y * sin;
    const Y = y * cos + x * sin;

    img_info.rotation += theta;
    img_info.left += x - X;
    img_info.top += y - Y;

    img_info = img_info;
  }

  function get_anchor_pos() {
    if (box == undefined) return [0, 0];
    return [
      anchor.left + anchor.size / 2 - box.offsetLeft,
      anchor.top + anchor.size / 2 - box.offsetTop,
    ];
  }

  export function set_anchor_pos(x, y) {
    if (box == undefined) return;
    anchor.left = x - anchor.size / 2 + box.offsetLeft;
    anchor.top = y - anchor.size / 2 + box.offsetTop;
  }
</script>

<svelte:window on:mouseup={mouse_up} on:mousemove={mouse_moving} />

<div class="image-canvas" bind:this={box}>
  <div
    class="image-frame"
    on:click={mouse_click}
    on:mousedown={mouse_down}
    role="none"
    style="
      top: {img_info.top}px; 
      left: {img_info.left}px; 
      width: {img_info.w}px; 
      height: {img_info.h}px; 
      rotate: {img_info.rotation}rad;
      opacity: {opacity[0]};
      background-image: url({g_img});
      cursor: {cursor};
      "
  />
</div>

<style>
  .image-canvas {
    position: absolute;
    top: var(--top-offset);
    left: 0;
    width: 100vw;
    height: calc(100vh - var(--top-offset));
    background-color: var(--mybg);
    overflow: hidden;
    user-select: none;
  }

  .image-frame {
    position: relative;
    background-repeat: no-repeat;
    background-size: 100% 100%;
    user-select: none;
  }
</style>
