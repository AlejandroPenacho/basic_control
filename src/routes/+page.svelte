<script>
    import { onMount } from 'svelte';
    import * as math from "mathjs";

    let x = math.complex(3, 4);

    let my_num = 1;
    let nyquist_values;

    let side_gone = 0;
    let zoom_level = 1;
    let diff_equation;

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

        diff_equation = (x) => my_num * (1 - x )
    }

    let canvas;
    let canvas_conf = {
        x_0: 0,
        y_0: 0,
        x_scale: 2,
        y_scale: 2
    };
    
    let second_canvas;

    let t_array = Array.from({length: 100000}, (e, i) => i/10000)
    let y_array = Array.from(100000)

    function update_trajectory() {
        let dt = 0.00001;
        y_array[0] = 0;
        let state = [0, 0];
        for (let i = 1; i < 100000; i++) {
            state[0] = state[0] + state[1] * dt;
            state[1] = state[1] + my_num * 2 * (1 - state[1]) - 4 * state[0];
            y_array[i] = state[0];
        }
        let ctx = second_canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.beginPath();
        for (let i=1; i < nyquist_values.length; i++) {
            ctx.lineTo(1000*t_array[i], -200*(y_array[i]) + 250);
        }
        ctx.stroke();
        ctx.beginPath;
        ctx.moveTo(0, 50)
        ctx.lineTo(500, 50)
        ctx.stroke();
    }

    function draw_nyquist() {
        let ctx = canvas.getContext("2d");
        let x_scale = canvas_conf.x_scale * canvas.width/2;
        let y_scale = canvas_conf.y_scale * canvas.height/2;

        let x_0 = canvas.width/2 + canvas_conf.x_scale * canvas_conf.x_0;
        let y_0 = canvas.height/2 + canvas_conf.y_scale * canvas_conf.y_0;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.beginPath()
        ctx.moveTo(
            x_scale*math.re(nyquist_values[0]) + x_0,
            -y_scale*math.im(nyquist_values[0]) + y_0,
        );

        for (let i=1; i < nyquist_values.length; i++) {
            ctx.lineTo(
                x_scale*math.re(nyquist_values[i]) + x_0,
                -y_scale*math.im(nyquist_values[i]) + y_0,
            );
        }
        ctx.stroke();
    }

    onMount(() => {
        draw_nyquist();
    })

    function update_canvas() {
        draw_nyquist();
        update_trajectory();
    }

    function canvas_scroll(e) {
        if (e.deltaY > 0) {
            canvas_conf.x_scale *= 0.8;
            canvas_conf.y_scale *= 0.8;
        } else {
            canvas_conf.x_scale /= 0.8;
            canvas_conf.y_scale /= 0.8;
        }
        draw_nyquist();
    }

    let ref_mouse_point = [0, 0];
    let mouse_point_pressed = false;
    function canvas_mouse_down(e) {
        mouse_point_pressed = true;
        ref_mouse_point = [
            e.clientX,
            e.clientY
        ]
    }
    function canvas_mouse_up(e) {
        mouse_point_pressed = false;
    }
    function canvas_mouse_move(e) {
        if (!mouse_point_pressed) {
            return
        }
        let delta_X = e.clientX - ref_mouse_point[0];
        let delta_Y = e.clientY - ref_mouse_point[1];
        ref_mouse_point = [
            e.clientX,
            e.clientY
        ]
        canvas_conf.x_0 += delta_X / canvas_conf.x_scale;
        canvas_conf.y_0 += delta_Y / canvas_conf.y_scale;
        console.log(canvas_conf.x_0)
        draw_nyquist();
    }

</script>

<style>
    div#myCanvas {
        border: 1mm solid #000000;
        width: 500px;
        height: 500px;
        overflow: hidden;
    }
</style>

This is a {x}
<br>
Hej check {nyquist_values[0]}

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
<input
    type="range"
    bind:value={side_gone}
    step="0.05"
    min="-10"
    max="10"
/>
<br>
<input
    type="range"
    bind:value={zoom_level}
    on:input={draw_nyquist}
    step="0.05"
    min="0.2"
    max="4"
/>
<br>
And the {nyquist_values[4]}
<br>

<div style="display: flex">
    <div id="myCanvas">
    <canvas
        bind:this={canvas}
        on:wheel={canvas_scroll}
        on:mousedown={canvas_mouse_down}
        on:mouseup={canvas_mouse_up}
        on:mousemove={canvas_mouse_move}
        width={500}
        height={500}
    />
    </div>

    <div id="myCanvas">
    <canvas
        bind:this={second_canvas}
        width={500}
        height={500}
        style="transform: translate({side_gone}cm)"
    />
    </div>
</div>