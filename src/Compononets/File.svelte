<script lang="ts">
    import { onMount } from 'svelte';
    import { picture } from '../stores';
    import { elt } from "../subroutines";

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;

    onMount(() => {
        canvas = document.querySelector('canvas') as HTMLCanvasElement;
        ctx = canvas.getContext('2d') as CanvasRenderingContext2D;
    });

    function handleSave() {
        let image = canvas.toDataURL("image/png").replace("image/png", "image/octet-stream"); 
        window.location.href=image;
    }

    function handleLoad() {
        startLoad(({pixels}) => {
            picture.subscribe(n=> {
                n.setPixels(pixels)
                n.redraw(ctx);
            });
        });
    }

    function startLoad(dispatch) {
        let input = elt("input", {
            type: "file",
            onchange: () => finishLoad(input.files[0], dispatch)
        });
        input.click();
        input.remove();
    }

    function finishLoad(file, dispatch) {
        if (file == null) return;
        let reader = new FileReader();
        reader.addEventListener("load", () => {
            let image = elt("img", {
            onload: () => dispatch({
                pixels: pictureFromImage(image)
            }),
            src: reader.result
            });
        });
        reader.readAsDataURL(file);
    }

    //Can be better
    function pictureFromImage(image) {
        let width = Math.min(900, image.width);
        let height = Math.min(600, image.height);

        let canvas = elt("canvas", {width: width, height: height});
        let cx = canvas.getContext("2d");
        cx.drawImage(image, 0, 0, width, height); 

        let pixels = [];
        let ret: string[][] =  new Array(height);
        for (let i = 0; i < height; i++)
            ret[i] = new Array(width);

        let {data} = cx.getImageData(0, 0, width, height);
        function hex(n) {
            return n.toString(16).padStart(2, "0");
        }
        for (let i = 0; i < data.length; i += 4) {
            let [r, g, b] = data.slice(i, i + 3);
            pixels.push("#" + hex(r) + hex(g) + hex(b));
        }
        for (let j = 0; j < (height / 10); j++)
            for (let i = 0; i < (width / 10); i++) 
                ret[j][i] = pixels[(j * width + i) * 10];

        return ret;
    }
</script>

<button class="save" on:click={handleSave}> Save </button>
<button class="load" on:click={handleLoad}> Load </button>

<style>
    button {
        margin-top: 1em;
        width: 7%;
    }
    .load{
        margin-left: 1em;
    }
</style>