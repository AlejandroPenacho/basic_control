<script>
  import Graph from "./graph.svelte";
  import { onMount } from "svelte";
  import * as math from "mathjs";

    // How to deal with the selection of poles/zeros/double poles/double zeros?
    // That is going to take some time, maybe?


  export let height;
  export let width;

  let canvas;

    let mode = "add_pole";

  export let transfer_function;

    let all_poles = [];
    let all_zeros = [];
    $:{
        all_poles = transfer_function.get_all_poles();
        all_zeros = transfer_function.get_all_zeros();
    }
  let last_mouse_down_position;
  let selected_singularity = undefined;

  onMount(() => {
    canvas.x_range = 6;
    canvas.y_range = 6;

    canvas.draw = function() {
        let ctx = canvas.main_canvas.getContext("2d");
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.font = "10px Arial";

        let SIZE = 5;
        ctx.lineWidth = 2;
        ctx.lineCap = "round";

        let all_poles = transfer_function.get_all_poles();
        let all_zeros = transfer_function.get_all_zeros();

        for (let i=0; i<all_poles.length; i++) {
            let value = [all_poles[i][0].re, all_poles[i][0].im];
            let power = all_poles[i][1];
            let pos = canvas.value_to_position(value);
            ctx.beginPath()
            ctx.moveTo(pos[0] - SIZE, pos[1] - SIZE);
            ctx.lineTo(pos[0] + SIZE, pos[1] + SIZE);
            ctx.stroke()
            ctx.beginPath()
            ctx.moveTo(pos[0] + SIZE, pos[1] - SIZE);
            ctx.lineTo(pos[0] - SIZE, pos[1] + SIZE);
            ctx.stroke()
            if (power > 1) {
                ctx.fillText(
                    power.toString(),
                    pos[0] + SIZE + 2,
                    pos[1] - SIZE - 2
                )
            }
        }
        for (let i=0; i<all_zeros.length; i++) {
            let value = [all_zeros[i][0].re, all_zeros[i][0].im];
            let power = all_zeros[i][1];

            let pos = canvas.value_to_position(value);
            ctx.beginPath();
            ctx.arc(pos[0], pos[1], SIZE, 0, 2*math.pi);
            ctx.stroke();
            if (power > 1) {
                ctx.fillText(
                    power.toString(),
                    pos[0] + SIZE + 2,
                    pos[1] - SIZE - 2
                )
            }
        }
      canvas.draw_grid();
    }

    canvas.mouse_down_action = (e) => {
        last_mouse_down_position = [e.layerX, e.layerY];
        let value = canvas.position_to_value(canvas.cursor_position);

        canvas.mouse_down_event(e);
    }

    canvas.mouse_up_action = (e) => {
        // Maybe we click outside and release inside here
        if (last_mouse_down_position == undefined) { return }

        if (e.layerX == last_mouse_down_position[0] && e.layerY == last_mouse_down_position[1]) {
          // Do not use the actual position, but the cursor one!!
          // let value = canvas.position_to_value([e.layerX, e.layerY]);
          let value = canvas.position_to_value(canvas.cursor_position);

          if (mode ==  "add_pole") {
            if (value[1] == 0) {
              transfer_function.add_pole(math.complex(value[0], 0));
            } else {
              transfer_function.add_double_pole(math.complex(value[0], math.abs(value[1])));
            }
          } else if (mode == "add_zero") {
            if (value[1] == 0) {
              transfer_function.add_zero(math.complex(value[0], 0));
            } else {
              transfer_function.add_double_zero(math.complex(value[0], math.abs(value[1])));
            }
          } else if (mode == "delete") {
            if (selected_singularity != undefined) {
              transfer_function.remove_element(selected_singularity);
            }
          }

          transfer_function = transfer_function;
        }
        canvas.mouse_up_event(e);
        canvas.draw();
    }


    canvas.mouse_move_action = (e) => {
        canvas.mouse_move_event(e);
        for (let i=0; i<all_poles.length; i++) {
            let pos = canvas.value_to_position([all_poles[i][0].re, all_poles[i][0].im])
            let dist = math.pow(
                math.pow(e.layerX - pos[0], 2) + math.pow(e.layerY - pos[1], 2),
                1/2
            )
            if (dist < 10) {
                canvas.cursor_position = pos;
                selected_singularity = transfer_function.identify_singularity([
                    all_poles[i][0].re,
                    all_poles[i][0].im,
                ])
                return
            }
        }
        for (let i=0; i<all_zeros.length; i++) {
            let pos = canvas.value_to_position([all_zeros[i][0].re, all_zeros[i][0].im])
            let dist = math.pow(
                math.pow(e.layerX - pos[0], 2) + math.pow(e.layerY - pos[1], 2),
                1/2
            )
            if (dist < 10) {
                canvas.cursor_position = pos;
                selected_singularity = transfer_function.identify_singularity([
                    all_zeros[i][0].re,
                    all_zeros[i][0].im,
                ])
                return
            }
        }
    }

    canvas.draw();
  })
</script>

<style>
    div.main_div {
        display: flex;
    }
    div.command_panel {
        display: flex;
        flex-direction: column;
    }
</style>

<div class="main_div">
    <Graph bind:canvas={canvas} height={height} width={width}/>
    <div class="command_panel">
        <label>
            <input type="radio" bind:group={mode} value={"add_pole"} />
            Add pole
        </label>
        <label>
            <input type="radio" bind:group={mode} value={"add_zero"} />
            Add zero
        </label>
        <label>
            <input type="radio" bind:group={mode} value={"delete"} />
            Remove element
        </label>
        <label>
            <input type="radio" bind:group={mode} value={"move"} />
            Move
        </label>
    </div>
</div>