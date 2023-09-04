<script>
  import Graph from "./graph.svelte";
  import { onMount } from "svelte";
  import * as math from "mathjs";


  export let height;
  export let width;

  let canvas;
  export let transfer_function;

  let last_mouse_down_position;

  onMount(() => {
    canvas.x_range = 6;
    canvas.y_range = 6;

    canvas.draw = function() {
        let ctx = canvas.canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        let SIZE = 5;
        ctx.lineWidth = 2;
        ctx.lineCap = "round";

        let all_poles = transfer_function.get_all_poles();
        let all_zeros = transfer_function.get_all_zeros();

        for (let i=0; i<all_poles.length; i++) {
            let value = [all_poles[i].re, all_poles[i].im];
            let pos = canvas.value_to_position(value);
            ctx.beginPath()
            ctx.moveTo(pos[0] - SIZE, pos[1] - SIZE);
            ctx.lineTo(pos[0] + SIZE, pos[1] + SIZE);
            ctx.stroke()
            ctx.beginPath()
            ctx.moveTo(pos[0] + SIZE, pos[1] - SIZE);
            ctx.lineTo(pos[0] - SIZE, pos[1] + SIZE);
            ctx.stroke()
        }
        for (let i=0; i<all_zeros.length; i++) {
            let value = [all_zeros[i].re, all_zeros[i].im];
            let pos = canvas.value_to_position(value);
            ctx.beginPath();
            ctx.arc(pos[0], pos[1], SIZE, 0, 2*math.pi);
            ctx.stroke();
        }
      
    }

    canvas.mouse_down_action = (e) => {
        last_mouse_down_position = [e.layerX, e.layerY];
        canvas.mouse_down_event(e);
    }

    canvas.mouse_up_action = (e) => {
        if (e.layerX == last_mouse_down_position[0] && e.layerY == last_mouse_down_position[1]) {
          let value = canvas.position_to_value([e.layerX, e.layerY]);
          transfer_function.add_double_pole(math.complex(value[0], value[1]));
          transfer_function = transfer_function;
        }
        canvas.mouse_up_event(e);
        canvas.draw();
    }

    canvas.draw();
  })
</script>

<Graph bind:canvas={canvas} height={height} width={width}/>
