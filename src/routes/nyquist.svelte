<script>
  import Graph from "./graph.svelte";
  import { onMount } from "svelte";
  import * as math from "mathjs";


  export let height;
  export let width;

  let canvas;
  export let nyquist_values;

    $: {
        nyquist_values = nyquist_values;
        if (canvas != undefined) {
            canvas.draw();
        }
    } 

  onMount(() => {
    canvas.draw = function() {
        let ctx = canvas.canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        let values = [];
        for (let i=0; i<nyquist_values.length; i++) {
            values.push([
                math.re(nyquist_values[i]),
                math.im(nyquist_values[i])
            ])
        }

        if (values.length == 0) { return }

        let initial_point = canvas.value_to_position(values[0]);
        ctx.beginPath();
        ctx.moveTo(initial_point[0], initial_point[1]);
        for (let i=1; i<values.length; i++) {
            let pos = canvas.value_to_position(values[i]);
            ctx.lineTo(pos[0], pos[1]);
        }
        ctx.stroke();
    }

    canvas.draw();
  })
</script>

<Graph bind:canvas={canvas} height={height} width={width}/>
