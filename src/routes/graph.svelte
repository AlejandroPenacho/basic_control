
<script>
  import { onMount } from "svelte";
  import * as math from "mathjs";

  // What goes here...
  // We must provide:
  //    A context that can be used to draw whatever we want, but careful with dimensions.
  //    A way of zooming and moving the plot
  // And receive:
  //    Something than handles clicks and moves and stuff

  export let width;
  export let height;

  class Canvas {
    constructor() {
      this.x_c = 0;
      this.y_c = 0;
      this.x_range = 4;
      this.y_range = 4;
      this.width = width;
      this.height = height;
      this.main_canvas = false;
      this.cursor_canvas = false;
      this.cursor_position = undefined;

      this.mouse_wheel_action = (e) => {
        this.mouse_wheel_event(e)
      }
      this.mouse_up_action = (e) => { this.mouse_up_event(e) }
      this.mouse_down_action = (e) => { this.mouse_down_event(e) }
      this.mouse_move_action = (e) => { this.mouse_move_event(e) }
      this.mouse_out_action = (e) => { this.mouse_out_event(e) }
      this.draw = (e) => {}
    }

    value_to_position(value) {
      let x_scale = this.width / this.x_range;
      let y_scale = this.height / this.y_range;
      let x_0 = this.width/2 + this.x_range * this.x_c;
      let y_0 = this.height/2 + this.y_range * this.y_c;

      return [
        this.width/2 + x_scale * (value[0] - this.x_c),
        this.height/2 - y_scale * (value[1] - this.y_c),
      ]
    }

    position_to_value(position) {
      let delta_x = (position[0]/this.width - 0.5) * this.x_range;
      let delta_y = -(position[1]/this.height - 0.5) * this.y_range;

      return [
        delta_x + this.x_c,
        delta_y + this.y_c
      ]
    }

    mouse_wheel_event(e) {
      e.preventDefault();
      let p_x = e.layerX;
      let p_y = e.layerY;
      let m_value = this.position_to_value([p_x, p_y]);

      let scroll_factor = 0.8;
      let scale_ratio;

      if (e.deltaY > 0) {
          scale_ratio = 1/scroll_factor;
      } else {
          scale_ratio = scroll_factor;
      }

      canvas.x_range *= scale_ratio;
      canvas.y_range *= scale_ratio;

      canvas.x_c = m_value[0] - scale_ratio * (m_value[0] - this.x_c)
      canvas.y_c = m_value[1] - scale_ratio * (m_value[1] - this.y_c)

      this.draw();
    }

    mouse_down_event(e) {
      mouse_is_down = true;
      last_mouse_pos = [e.clientX, e.clientY];

      let pos = [e.layerX, e.layerY];
    }

    mouse_up_event(e) {
      mouse_is_down = false
    }

    mouse_out_event(e) {
      mouse_is_down = false;
      this.cursor_position = undefined;
      this.draw_cursor();
    }

    mouse_move_event(e) {
      if (mouse_is_down) {
        let delta_x = e.clientX - last_mouse_pos[0];
        let delta_y = e.clientY - last_mouse_pos[1];

        last_mouse_pos = [e.clientX, e.clientY];

        this.x_c -= delta_x * this.x_range / this.width;
        this.y_c += delta_y * this.y_range / this.height;

        this.draw();
      }
      this.cursor_position = [e.layerX, e.layerY];
      if (math.abs(this.cursor_position[0] - this.width/2) / this.width < 0.02) {
        this.cursor_position[0] = this.width / 2;
      }
      if (math.abs(this.cursor_position[1] - this.height/2)/this.height < 0.02) {
        this.cursor_position[1] = this.height / 2;
      }
      this.draw_cursor();
    }

    draw_cursor() {
      // Improve performance by a 200% for free!!! Front-end
      // "developers" hate him!!
      if (cursor_animation_request != undefined) {
        cancelAnimationFrame(cursor_animation_request);
        cursor_animation_request = undefined;
      }
      cursor_animation_request = requestAnimationFrame(() => {
        let ctx = this.cursor_canvas.getContext("2d");
        ctx.lineWidth = 0.2;
        ctx.clearRect(0, 0, this.width, this.height);

        if (this.cursor_position == undefined) { return }

        let x = this.cursor_position[0];
        let y = this.cursor_position[1];

        ctx.beginPath()
        ctx.moveTo(x, 0)
        ctx.lineTo(x, this.height)
        ctx.stroke()
        ctx.beginPath()
        ctx.moveTo(0, y)
        ctx.lineTo(this.width, y)
        ctx.stroke()

        let value = this.position_to_value(this.cursor_position);
        ctx.font = "10px Arial";
        ctx.fillText(
          value[0].toFixed(2),
          x + 10, this.height - 10,
          // x, y,
        )
        ctx.fillText(
          value[1].toFixed(2),
          this.width - 50, y - 10
        )
      })
      
    }
  }

  export let canvas = new Canvas();

  function value_to_position(value, parameters) {
    let x_scale = parameters.width / parameters.x_range;
    let y_scale = parameters.height / parameters.y_range;
    let x_0 = parameters.width/2 + parameters.x_range * parameters.x_c;
    let y_0 = parameters.height/2 + parameters.y_range * parameters.y_c;

    return [
      parameters.width/2 + x_scale * (value[0] - parameters.x_c),
      parameters.height/2 - y_scale * (value[1] - parameters.y_c),
    ]
  }

  function position_to_value(position, parameters) {
    let delta_x = (position[0]/parameters.width - 0.5) * parameters.x_range;
    let delta_y = -(position[1]/parameters.height - 0.5) * parameters.y_range;

    return [
      delta_x + parameters.x_c,
      delta_y + parameters.y_c
    ]
  }

  let mouse_is_down = false;
  let cursor_animation_request = undefined;
  let last_mouse_pos = [0, 0];


</script>

<style>
    div#myCanvas {
        border: 1mm solid #100000;
        overflow: hidden;
        position: relative;
    }
</style>

<div
  id="myCanvas"
  style="
    width: {canvas.width}px;
    height: {canvas.height}px;
  "
>
<canvas
    style="position: absolute; top: 0; left: 0"
    bind:this={canvas.main_canvas}
    width={canvas.width}
    height={canvas.height}
/>
<canvas
    style="position: absolute; top: 0; left: 0"
    bind:this={canvas.cursor_canvas}
    on:wheel={canvas.mouse_wheel_action}
    on:mousedown={canvas.mouse_down_action}
    on:mouseout={canvas.mouse_out_action}
    on:blur={canvas.mouse_out_action}
    on:mouseup={canvas.mouse_up_action}
    on:mousemove={canvas.mouse_move_action}
    width={canvas.width}
    height={canvas.height}
/>
</div>
