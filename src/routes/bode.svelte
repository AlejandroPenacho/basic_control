<script>
  import Graph from "./graph.svelte";
  import { onMount } from "svelte";
  import * as math from "mathjs";


  export let height;
  export let width;

  let mag_canvas;
  let phase_canvas;
  export let nyquist_values;
  export let frequencies;

    $: {
        nyquist_values = nyquist_values;
        if (mag_canvas != undefined) {
            mag_canvas.draw();
        }
        if (phase_canvas != undefined) {
            phase_canvas.draw();
        }
    } 

  onMount(() => {
    mag_canvas.x_c = 1;
    mag_canvas.x_range = 8;
    mag_canvas.y_c = -1;
    mag_canvas.y_range = 8;

    phase_canvas.x_c = 1;
    phase_canvas.x_range = 8;
    phase_canvas.y_c = 0;
    phase_canvas.y_range = 360;



    mag_canvas.draw = function() {
      let ctx = mag_canvas.main_canvas.getContext("2d");

      ctx.clearRect(0, 0, mag_canvas.width, mag_canvas.height);
      ctx.beginPath();

      for (let i=0; i<frequencies.length; i++) {
        let f = frequencies[i];
        let mag = nyquist_values[i].toPolar().r;
        let value = [
            math.log(f, 10),
            math.log(mag, 10)
        ]

        let position = mag_canvas.value_to_position(value)
        if (i==0) {
            ctx.moveTo(position[0], position[1]);
        } else {
            ctx.lineTo(position[0], position[1]);
        }
        ctx.stroke();
      }
    }

    phase_canvas.draw = function() {
        let ctx = phase_canvas.main_canvas.getContext("2d");
        ctx.clearRect(0, 0, phase_canvas.width, phase_canvas.height);

        ctx.beginPath();

        for (let i=0; i<frequencies.length; i++) {
            let f = frequencies[i];
            let angle = nyquist_values[i].toPolar().phi;
            let value = [
                math.log(f, 10),
                angle * 180 / math.pi
            ]

            let position = phase_canvas.value_to_position(value)
            if (i==0) {
                ctx.moveTo(position[0], position[1]);
            } else {
                ctx.lineTo(position[0], position[1]);
            }
            ctx.stroke();
        }
    }

    mag_canvas.mouse_move_action = (e) => {
      mag_canvas.mouse_move_event(e);
      phase_canvas.x_c = mag_canvas.x_c;
      phase_canvas.x_range = mag_canvas.x_range;
      phase_canvas.draw();
    }
    mag_canvas.mouse_wheel_action = (e) => {
      mag_canvas.mouse_wheel_event(e);
      phase_canvas.x_c = mag_canvas.x_c;
      phase_canvas.x_range = mag_canvas.x_range;
      phase_canvas.draw();
    }

    phase_canvas.mouse_move_action = (e) => {
      phase_canvas.mouse_move_event(e);
      mag_canvas.x_c = phase_canvas.x_c;
      mag_canvas.x_range = phase_canvas.x_range;
      mag_canvas.draw();
    }
    phase_canvas.mouse_wheel_action = (e) => {
      phase_canvas.mouse_wheel_event(e);
      mag_canvas.x_c = phase_canvas.x_c;
      mag_canvas.x_range = phase_canvas.x_range;
      mag_canvas.draw();
    }


    mag_canvas.draw();
    phase_canvas.draw();
  })
</script>

<div style="
    display: flex;
    flex-direction: column;
    justify-content: space-between">
    <Graph bind:canvas={mag_canvas} height={200} width={500}/>
    <Graph bind:canvas={phase_canvas} height={200} width={500}/>
</div>
