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

  let log_magnitude_array = new Array(nyquist_values.length);
  let phase_array = new Array(nyquist_values.length);
  let log_freq_array = new Array(frequencies.length)

    $: {
        for (let i=0; i<nyquist_values.length; i++) {
          let polar = nyquist_values[i].toPolar();
          log_magnitude_array[i] = math.log(polar.r, 10);
          phase_array[i] = polar.phi * 180 / math.pi;
          log_freq_array[i] = math.log(frequencies[i], 10);
        }

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
      ctx.setLineDash([]);
      ctx.lineWidth = 1.5;
      ctx.beginPath();

      for (let i=0; i<frequencies.length; i++) {
        let f = log_freq_array[i];
        let mag = log_magnitude_array[i];
        let value = [
            f,
            mag
        ]

        let position = mag_canvas.value_to_position(value)
        if (i==0) {
            ctx.moveTo(position[0], position[1]);
        } else {
            ctx.lineTo(position[0], position[1]);
        }
        ctx.stroke();
      }
      mag_canvas.draw_grid();
    }

    phase_canvas.draw = function() {
        let ctx = phase_canvas.main_canvas.getContext("2d");
        ctx.clearRect(0, 0, phase_canvas.width, phase_canvas.height);
        ctx.setLineDash([]);
        ctx.lineWidth = 1.5;

        ctx.beginPath();

        for (let i=0; i<frequencies.length; i++) {
            let f = log_freq_array[i];
            let phase = phase_array[i];
            let value = [
                f,
                phase
            ]

            let position = phase_canvas.value_to_position(value)
            if (i==0) {
                ctx.moveTo(position[0], position[1]);
            } else {
                ctx.lineTo(position[0], position[1]);
            }
            ctx.stroke();
        }
        phase_canvas.draw_grid();
    }

    mag_canvas.mouse_move_action = (e) => {
      mag_canvas.mouse_move_event(e);

      if ((phase_canvas.x_c != mag_canvas.x_c) || (phase_canvas.x_range != mag_canvas.x_range)) {
        phase_canvas.x_c = mag_canvas.x_c;
        phase_canvas.x_range = mag_canvas.x_range;
        phase_canvas.draw();
      }
    }
    mag_canvas.mouse_wheel_action = (e) => {
      mag_canvas.mouse_wheel_event(e);

      if ((phase_canvas.x_c != mag_canvas.x_c) || (phase_canvas.x_range != mag_canvas.x_range)) {
        phase_canvas.x_c = mag_canvas.x_c;
        phase_canvas.x_range = mag_canvas.x_range;
        phase_canvas.draw();
      }
    }

    phase_canvas.mouse_move_action = (e) => {
      phase_canvas.mouse_move_event(e);

      if ((phase_canvas.x_c != mag_canvas.x_c) || (phase_canvas.x_range != mag_canvas.x_range)) {
        mag_canvas.x_c = phase_canvas.x_c;
        mag_canvas.x_range = phase_canvas.x_range;
        mag_canvas.draw();
      }
    }
    phase_canvas.mouse_wheel_action = (e) => {
      phase_canvas.mouse_wheel_event(e);

      if ((phase_canvas.x_c != mag_canvas.x_c) || (phase_canvas.x_range != mag_canvas.x_range)) {
        mag_canvas.x_c = phase_canvas.x_c;
        mag_canvas.x_range = phase_canvas.x_range;
        mag_canvas.draw();
      }
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
