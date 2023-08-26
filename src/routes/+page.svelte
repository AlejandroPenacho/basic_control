<script>
    import { onMount } from 'svelte';
    import * as math from "mathjs";

    let x = math.complex(3, 4);

    let my_num = 1;
    let nyquist_values;

    $: {
        let poles = [ math.complex(-my_num), math.complex(-2)];

        let values = Array.from(
            {length: 200},
            (e,i) => math.complex(0, math.pow(10, i/50 - 2))
        )

        nyquist_values = values.map((x) => {
            let a = math.multiply(
                math.add(x, -poles[0]),
                math.add(x, -poles[1])
            )
            return math.divide(math.complex(1), a)
        });
    }

    let canvas;

    function draw_nyquist() {
        let ctx = canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.beginPath()
        ctx.moveTo(100*math.re(nyquist_values[0]) + 250, 100*math.im(nyquist_values[0]) + 250);
        for (let i=1; i < nyquist_values.length; i++) {
            ctx.lineTo(100*math.re(nyquist_values[i]) + 250, -100*math.im(nyquist_values[i]) + 250);
        }
        ctx.stroke();
    }

    onMount(() => {
        draw_nyquist();
    })

    function update_canvas() {
        draw_nyquist();
    }

    
</script>

<style>
    canvas#myCanvas {
        border: 1mm solid #000000;
    }
</style>

This is a {x}

Hail the pole:
<br>
<input
    type="number"
    bind:value={my_num}
    on:input={update_canvas}
    min="0.1"
    max="10"
/>
<input
    type="range"
    bind:value={my_num}
    on:input={update_canvas}
    step="0.05"
    min="0.1"
    max="10"
/>

<br>
And the {nyquist_values[4]}
<br>

<canvas bind:this={canvas} width={500} height={500} id="myCanvas"/>