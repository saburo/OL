<script>
  import { onMount } from "svelte";
  import futaba from "$lib/images/IMG_7739.jpg";

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

  let top, left;
  let w, h;
  let anchor_size;
  let is_dragging;
  let anc_top;
  let anc_left;
  let scale;
  let rotation;
  let anc_org_x;
  let anc_org_y;

  onMount(async () => {
    init();
  });

  function init() {
    top = 0;
    left = 0;
    w = 420;
    h = 560;
    is_dragging = false;
    anchor_size = 10;
    scale = { x: 1, y: 1 };
    rotation = 0;
    anc_org_x = 0;
    anc_org_y = 0;
    box = document.querySelector(".image-canvas");
    set_anchor_pos(0, 0);
  }

  function handle_reset_button() {
    init();
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

    if (e.shiftKey) {
      shifting(e);
      return;
    }

    if (e.metaKey) {
      rotating(e);
      return;
    }

    scaling(e);
  }

  function mouse_up(e) {
    is_dragging = false;
  }

  function mouse_click(e) {
    if (!e.altKey) {
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
    anc_org_x = x;
    anc_org_y = y;

    // console.log(anc_left, anc_top, x, y);
    // console.log(document.querySelector("#myimg").style.transformOrigin);
  }

  function shifting(e) {
    left += e.movementX;
    top += e.movementY;
    anc_left += e.movementX;
    anc_top += e.movementY;
  }

  function scaling(e) {
    const pos = get_mouse_pos(e);
    const anc_pos = get_anchor_pos();

    scale.x = 1 + e.movementX / (pos[0] - anc_pos[0]);
    scale.y = 1 + e.movementY / (pos[1] - anc_pos[1]);

    w *= scale.x;
    h *= scale.y;

    left -= (anc_pos[0] - left) * (scale.x - 1);
    top -= (anc_pos[1] - top) * (scale.y - 1);
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

    rotation += theta;
    left += x - X;
    top += y - Y;
  }

  function get_img_center() {
    // w/ rotation
    //
    // w/o rotation
    const c_x = left + w / 2;
    const c_y = top + h / 2;
  }

  // init();
</script>

<svelte:window on:mouseup={mouse_up} on:mousemove={mouse_moving} />

<section>
  <h1>test overlay</h1>
  <button on:click={handle_reset_button}>Reset</button>
  <div class="image-canvas">
    <div
      id="myimg"
      class="img-frame"
      on:click={mouse_click}
      style="
        top: {top}px; 
        left: {left}px; 
        width: {w}px; 
        height: {h}px; 
        rotate: {rotation}rad;
        background-image: url({futaba});
      "
    >
      <div class="img-handle-br" on:mousedown={mouse_down} />
    </div>
  </div>
  <div class="anchor" style="left: {anc_left}px; top: {anc_top}px;" />
</section>

<style>
  .image-canvas {
    position: relative;
    top: 0;
    left: 0;
    width: 1000px;
    height: 1000px;
    background-color: #eee;
    overflow: hidden;
  }
  .img-frame {
    position: relative;
    top: 0;
    left: 0;
    height: 560px;
    width: 420px;
    background-repeat: no-repeat;
    border: 1px solid #ff0000;
    background-size: 100% 100%;
  }

  .img-handle-br {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
    background-color: #ff000022;
    user-select: none;
    cursor: move;
  }

  .anchor {
    position: absolute;
    width: 10px;
    height: 10px;
    background-color: steelblue;
  }
</style>
