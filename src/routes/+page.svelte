<script>
    import { onMount } from 'svelte';
    import * as math from "mathjs";
    import Graph from "./graph.svelte";
    import Nyquist from "./nyquist.svelte";
    import PZMap from "./pz_map.svelte";
    import Bode from "./bode.svelte";

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
            for (let i=0; i < this.poles.length; i++) {
                if (pole.equals(this.poles[i][0])) {
                    this.poles[i][1] += 1;
                    return
                }
            }
            this.poles.push([pole, 1])
        }

        add_zero(zero) {
            for (let i=0; i < this.zeros.length; i++) {
                if (zero.equals(this.zeros[i][0])) {
                    this.zeros[i][1] += 1;
                    return
                }
            }
            this.zeros.push([zero, 1])
        }
        add_double_pole(double_pole) {
            if (double_pole.im < 0) {
                console.log("Negative double pole!!!");
            }
            for (let i=0; i < this.double_poles.length; i++) {
                if (double_pole.equals(this.double_poles[i][0])) {
                    this.double_poles[i][1] += 1;
                    return
                }
            }
            this.double_poles.push([double_pole, 1])
        }
        add_double_zero(double_zero) {
            if (double_zero.im < 0) {
                console.log("Negative double zero!!!");
            }
            for (let i=0; i < this.double_zeros.length; i++) {
                if (double_zero.equals(this.double_zeros[i][0])) {
                    this.double_zeros[i][1] += 1;
                    return
                }
            }
            this.double_zeros.push([double_zero, 1])
        }

        get_all_poles() {
            let output = [];
            for (let i = 0; i < this.poles.length; i++) {
                output.push(this.poles[i])
            }
            for (let i = 0; i < this.double_poles.length; i++) {
                output.push(this.double_poles[i])
                output.push(
                    [math.conj(this.double_poles[i][0]), this.double_poles[i][1]]
                )
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
                output.push(
                    [math.conj(this.double_zeros[i][0]), this.double_zeros[i][1]]
                )
            }
            return output
        }

        identify_singularity(val) {
            let value = math.complex(val[0], math.abs(val[1]));

            if (value.im == 0) {
                for (let i=0; i<this.zeros.length; i++) {
                    if (value.equals(this.zeros[i][0])) {
                        return ["zero", i]
                    }
                }
                for (let i=0; i<this.poles.length; i++) {
                    if (value.equals(this.poles[i][0])) {
                        return ["pole", i]
                    }
                }
            }

            for (let i=0; i<this.double_zeros.length; i++) {
                if (value.equals(this.double_zeros[i][0])) {
                    return ["double_zero", i]
                }
            }
            for (let i=0; i<this.double_poles.length; i++) {
                if (value.equals(this.double_poles[i][0])) {
                    return ["double_pole", i]
                }
            }
            return undefined
        }

        remove_element(element) {
            let type = element[0];
            let index = element[1];

            if (type == "pole") {
                this.poles.splice(index, 1);
            } else if (type == "zero") {
                this.zeros.splice(index, 1);
            } else if (type == "double_pole") {
                this.double_poles.splice(index, 1);
            } else if (type == "double_zero") {
                this.double_zeros.splice(index, 1);
            }
        }

        get_frequency_response(frequencies) {
            let poles = this.get_all_poles();
            let zeros = this.get_all_zeros();

            let output = [];

            for (let i=0; i<frequencies.length; i++) {
                let x = math.complex(0, frequencies[i]);
                let value = math.complex(1, 0);

                for (let j=0; j < zeros.length; j++) {
                    value = math.multiply(
                        value,
                        math.pow(
                            math.add(
                                1,
                                math.divide(x, zeros[j][0]).neg()
                            ),
                            zeros[j][1]
                        )
                    );
                }
                for (let j=0; j < poles.length; j++) {
                    value = math.divide(
                        value,
                        math.pow(
                            math.add(
                                1,
                                math.divide(x, poles[j][0]).neg()
                            ),
                            poles[j][1]
                        )
                    );
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

{nyquist_values[2]}

<div style="display: flex">
    <Nyquist bind:nyquist_values={nyquist_values} height={500} width={500}/>
    <div style="width: 5mm"></div>
    <Bode
        bind:frequencies={frequencies}
        bind:nyquist_values={nyquist_values}
        height={500}
        width={500}
    />
    <div style="width: 5mm"></div>
    <PZMap bind:transfer_function={transfer_function} height={500} width={500}/>
</div>
