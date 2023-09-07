<script>
  import Graph from "./graph.svelte";
  import { onMount } from "svelte";
  import * as math from "mathjs";


  export let height;
  export let width;

  let canvas;
  export let nyquist_values;

  let x_array = new Array(nyquist_values.length)
  let y_array = new Array(nyquist_values.length)

   $: {
      nyquist_values = nyquist_values;
      for (let i=0; i<nyquist_values.length; i++) {
        x_array[i] = nyquist_values[i].re;
        y_array[i] = nyquist_values[i].im;
      }
      if (canvas != undefined) {
        canvas.draw();
      }
    } 

  onMount(() => {
    canvas.draw = function() {
        let ctx = canvas.main_canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        let initial_point = canvas.value_to_position([x_array[0], y_array[0]]);
        ctx.beginPath();
        ctx.moveTo(initial_point[0], initial_point[1]);
        for (let i=1; i<x_array.length; i++) {
            let pos = canvas.value_to_position([x_array[i], y_array[i]]);
            ctx.lineTo(pos[0], pos[1]);
        }
        ctx.stroke();
    }

    canvas.draw();
  })
</script>

<Graph bind:canvas={canvas} height={height} width={width}/>
