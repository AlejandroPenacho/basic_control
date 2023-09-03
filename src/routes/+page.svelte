<script>
    import { onMount } from 'svelte';
    import * as math from "mathjs";
    import Graph from "./graph.svelte";

    let my_num = 0;
    let side_gone = 0;
    let zoom_level = 0;

    class TransferFunction {
        constructor() {
            this.poles = [];
            this.zeros = [];
            this.double_poles = [];
            this.double_zeros = [];

        }

        add_pole(pole) {
            this.poles.push(pole)
        }

        add_zero(zero) {
            this.zeros.push(zero)
        }
        add_double_pole(double_pole) {
            this.double_poles.push(double_pole)
        }
        add_double_zero(double_zero) {
            this.double_zeros.push(double_zero)
        }

        get_all_poles() {
            let output = [];
            for (let i = 0; i < this.poles.length; i++) {
                output.push(this.poles[i])
            }
            for (let i = 0; i < this.double_poles.length; i++) {
                output.push(this.double_poles[i])
                output.push(math.conj(this.double_poles[i]))
            }
            return output
        }

        get_all_zeros() {
            let output = [];
            for (let i = 0; i < this.zeros.length; i++) {
                output.push(this.zeros[i])
            }
            for (let i = 0; i < this.double_zeros.length; i++) {
                output.push(this.double_zeros[i])
                output.push(math.conj(this.double_zeros[i]))
            }
            return output
        }

        get_frequency_response(frequencies) {
            let poles = this.get_all_poles();
            let zeros = this.get_all_zeros();

            let output = [];

            for (let i=0; i<frequencies.length; i++) {
                let x = math.complex(0, frequencies[i]);
                let value = math.complex(1, 0);

                for (let j=0; j < zeros.length; j++) {
                    value = math.multiply(value, math.add(x, -zeros[j]));
                }
                for (let j=0; j < poles.length; j++) {
                    value = math.divide(value, math.add(x, poles[j].neg()));
                }
                output.push(value)
            }
            return output
        }
    }


    let nyquist_values = [
        math.complex(2, 1),
        math.complex(0, -1),
        math.complex(-1, -0.5),
        math.complex(0, 0)
    ];
    let transfer_function = new TransferFunction();

    transfer_function.add_pole(math.complex(-3))
    transfer_function.add_pole(math.complex(-1))
    transfer_function.add_double_pole(math.complex(-1.5, 0.3))
    transfer_function.add_zero(math.complex(-2))

    let all_poles = transfer_function.get_all_poles();

    let frequencies = Array.from(
        {length: 200},
        (e,i) => math.pow(10, i/50 - 2)
    )

    nyquist_values = transfer_function.get_frequency_response(
        frequencies
    );

    $: {
        nyquist_values = transfer_function.get_frequency_response(
            frequencies
        );
        if (nyquist_canvas != undefined) {
            nyquist_canvas.draw();
        }
        if (bode_mag_canvas != undefined) {
            bode_mag_canvas.draw();
        }
        if (bode_phase_canvas != undefined) {
            bode_phase_canvas.draw();
        }
        if (pz_canvas != undefined) {
            pz_canvas.draw();
        }
    }

    let nyquist_canvas;
    let bode_mag_canvas;
    let bode_phase_canvas;
    let pz_canvas;


    onMount(() => {
        bode_mag_canvas.x_c = 1;
        bode_mag_canvas.x_range = 8;
        bode_mag_canvas.y_c = -1;
        bode_mag_canvas.y_range = 8;

        bode_phase_canvas.x_c = 1;
        bode_phase_canvas.x_range = 8;
        bode_phase_canvas.y_c = 0;
        bode_phase_canvas.y_range = 360;

        pz_canvas.x_range = 6;
        pz_canvas.y_range = 6;
    
        nyquist_canvas.draw = function() {
            let ctx = nyquist_canvas.canvas.getContext("2d");
            ctx.clearRect(0, 0, nyquist_canvas.width, nyquist_canvas.height);

            let values = [];
            for (let i=0; i<nyquist_values.length; i++) {
                values.push([
                    math.re(nyquist_values[i]),
                    math.im(nyquist_values[i])
                ])
            }

            let initial_point = nyquist_canvas.value_to_position(values[0]);
            ctx.beginPath();
            ctx.moveTo(initial_point[0], initial_point[1]);
            for (let i=1; i<values.length; i++) {
                let pos = nyquist_canvas.value_to_position(values[i]);
                ctx.lineTo(pos[0], pos[1]);
            }
            ctx.stroke();
        }

        bode_mag_canvas.draw = function() {
            console.log("Whatever")
            let ctx = bode_mag_canvas.canvas.getContext("2d");

            ctx.clearRect(0, 0, bode_mag_canvas.width, bode_mag_canvas.height);
            ctx.beginPath();

            for (let i=0; i<frequencies.length; i++) {
                let f = frequencies[i];
                let mag = nyquist_values[i].toPolar().r;
                let value = [
                    math.log(f, 10),
                    math.log(mag, 10)
                ]

                let position = bode_mag_canvas.value_to_position(value)
                if (i==0) {
                    ctx.moveTo(position[0], position[1]);
                } else {
                    ctx.lineTo(position[0], position[1]);
                }
                ctx.stroke();
            }
        }

        bode_phase_canvas.draw = function() {
            let ctx = bode_phase_canvas.canvas.getContext("2d");
            ctx.clearRect(0, 0, bode_phase_canvas.width, bode_phase_canvas.height);

            ctx.beginPath();

            for (let i=0; i<frequencies.length; i++) {
                let f = frequencies[i];
                let angle = nyquist_values[i].toPolar().phi;
                let value = [
                    math.log(f, 10),
                    angle * 180 / math.pi
                ]

                let position = bode_phase_canvas.value_to_position(value)
                if (i==0) {
                    ctx.moveTo(position[0], position[1]);
                } else {
                    ctx.lineTo(position[0], position[1]);
                }
                ctx.stroke();
            }
        }

        pz_canvas.draw = function() {
            let ctx = pz_canvas.canvas.getContext("2d");
            ctx.clearRect(0, 0, pz_canvas.width, pz_canvas.height);

            let SIZE = 5;
            ctx.lineWidth = 2;
            ctx.lineCap = "round";

            let all_poles = transfer_function.get_all_poles();
            let all_zeros = transfer_function.get_all_zeros();

            for (let i=0; i<all_poles.length; i++) {
                let value = [all_poles[i].re, all_poles[i].im];
                let pos = pz_canvas.value_to_position(value);
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
                let pos = pz_canvas.value_to_position(value);
                ctx.beginPath();
                ctx.arc(pos[0], pos[1], SIZE, 0, 2*math.pi);
                ctx.stroke();
            }
            
        }

        pz_canvas.mouse_down_mid = function(e) {
            let pos = [e.layerX, e.layerY];
            let value = pz_canvas.position_to_value(pos);

            transfer_function.add_double_pole(math.complex(value[0], value[1]));
            transfer_function = transfer_function;
            return true
        }

        bode_mag_canvas.draw();
        bode_phase_canvas.draw();
        nyquist_canvas.draw();
        pz_canvas.draw();
    })
</script>

<style>
    div#myCanvas {
        border: 1mm solid #000000;
        width: 500px;
        height: 500px;
        overflow: hidden;
    }
</style>


<input
    type="number"
    bind:value={my_num}
    min="0.1"
    max="10"
/>
<input
    type="range"
    bind:value={my_num}
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
    step="0.05"
    min="0.2"
    max="4"
/>

<div style="display: flex">
    <Graph bind:canvas={nyquist_canvas} height={500} width={500}/>
    <div style="width: 5mm"></div>
    <div style="
        display: flex;
        flex-direction: column;
        justify-content: space-between">
        <Graph bind:canvas={bode_mag_canvas} height={200} width={500}/>
        <Graph bind:canvas={bode_phase_canvas} height={200} width={500}/>
    </div>
    <div style="width: 5mm"></div>
    <Graph bind:canvas={pz_canvas} height={500} width={500}/>
</div>
