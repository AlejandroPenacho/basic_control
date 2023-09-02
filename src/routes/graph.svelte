
<script>
  import { onMount } from "svelte";

  // What goes here...
  // We must provide:
  //    A context that can be used to draw whatever we want, but careful with dimensions.
  //    A way of zooming and moving the plot
  // And receive:
  //    Something than handles clicks and moves and stuff

  export let canvas;
  export let canvas_parameters = {
    x_c: 0,
    y_c: 0,
    x_range: 4,
    y_range: 4
  }

  function value_to_position(value, canvas, parameters) {
    let x_scale = canvas.width / parameters.x_range;
    let y_scale = canvas.height / parameters.y_range;
    let x_0 = canvas.width/2 + canvas_parameters.x_range * canvas_parameters.x_c;
    let y_0 = canvas.height/2 + canvas_parameters.y_range * canvas_parameters.y_c;

    return [
      canvas.width/2 + x_scale * (value[0] - parameters.x_c),
      canvas.height/2 - y_scale * (value[1] - parameters.y_c),
    ]
  }

  function position_to_value(position, canvas, parameters) {
    let delta_x = (position[0]/canvas.width - 0.5) * parameters.x_range;
    let delta_y = -(position[1]/canvas.height - 0.5) * parameters.y_range;

    return [
      delta_x + parameters.x_c,
      delta_y + parameters.y_c
    ]
  }

  let mouse_is_down = false;
  let last_mouse_pos = [0, 0];

  let test_points = [
    [0, 0],
    [1, 0.5],
    [1, -1],
    [-1, -1],
    [-0.5, 1],
    [-2, 2]
  ]

  function draw() {
      let ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.beginPath();
      let initial_point = value_to_position(
        test_points[0],
        canvas,
        canvas_parameters
      );
      ctx.moveTo(initial_point[0], initial_point[1]);
      for (let i=1; i< test_points.length; i++) {
        let norm_point = value_to_position(
          test_points[i],
          canvas,
          canvas_parameters
        );
        ctx.lineTo(norm_point[0], norm_point[1]);
      }
      ctx.stroke();
  }


  export let mouse_wheel_mid = function(e) { return false }
  export let mouse_up_mid = function(e) { return false }
  export let mouse_down_mid = function(e) { return false }
  export let mouse_move_mid = function(e) { return false }

  function mouse_wheel_event(e) {
      let skip_scroll = mouse_wheel_mid();
      if (skip_scroll) { return }

      e.preventDefault();
      let p_x = e.layerX;
      let p_y = e.layerY;
      let m_value = position_to_value([p_x, p_y], canvas, canvas_parameters);

      let scroll_factor = 0.8;
      let scale_ratio;

      if (e.deltaY > 0) {
          scale_ratio = 1/scroll_factor;
      } else {
          scale_ratio = scroll_factor;
      }

      canvas_parameters.x_range *= scale_ratio;
      canvas_parameters.y_range *= scale_ratio;

      canvas_parameters.x_c = m_value[0] - scale_ratio * (m_value[0] - canvas_parameters.x_c)
      canvas_parameters.y_c = m_value[1] - scale_ratio * (m_value[1] - canvas_parameters.y_c)

      draw();
  }

  function mouse_down_event(e) {
    let skip_mouse_down = mouse_down_mid(e);
    if (skip_mouse_down) { return }

    mouse_is_down = true;
    last_mouse_pos = [e.clientX, e.clientY];

    let pos = [e.layerX, e.layerY];
  }

  function mouse_up_event(e) {
    let skip_mouse_up = mouse_up_mid(e);
    if (skip_mouse_up) { return }
    mouse_is_down = false
  }

  function mouse_move_event(e) {
    let skip_mouse_move = mouse_move_mid(e);
    if (skip_mouse_move) { return }
    
    if (!mouse_is_down) { return }
    let delta_x = e.clientX - last_mouse_pos[0];
    let delta_y = e.clientY - last_mouse_pos[1];

    last_mouse_pos = [e.clientX, e.clientY];

    canvas_parameters.x_c -= delta_x * canvas_parameters.x_range / canvas.width;
    canvas_parameters.y_c += delta_y * canvas_parameters.y_range / canvas.height;
    draw();
  }

  onMount(() => {
    draw();
  })
</script>

<style>
    div#myCanvas {
        border: 1mm solid #100000;
        width: 500px;
        height: 500px;
        overflow: hidden;
    }
</style>

<div id="myCanvas">
<canvas
    bind:this={canvas}
    on:wheel={mouse_wheel_event}
    on:mousedown={mouse_down_event}
    on:mouseout={mouse_up_event}
    on:blur={mouse_up_event}
    on:mouseup={mouse_up_event}
    on:mousemove={mouse_move_event}
    width={500}
    height={500}
/>
</div>
