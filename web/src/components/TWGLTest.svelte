<script lang="ts">
	import * as twgl from 'twgl.js';
	import {onMount} from 'svelte';

	// Returns a random integer from 0 to range - 1.
	function randomInt(range: number) {
		return Math.floor(Math.random() * range);
	}

	onMount(() => {
		const canvas = document.querySelector('#c2') as HTMLCanvasElement;
		const gl = canvas.getContext('webgl2');
		if (!gl) {
			throw new Error(`could not create WebGL2 context`);
		}

		const vertexShaderSource = `#version 300 es
     
// an attribute is an input (in) to a vertex shader.
// It will receive data from a buffer
in vec2 a_position;
 
uniform vec2 u_resolution;

void main() {
	// convert the position from pixels to 0.0 to 1.0
    vec2 zeroToOne = a_position / u_resolution;

	// convert from 0->1 to 0->2
	vec2 zeroToTwo = zeroToOne * 2.0;

	// convert from 0->2 to -1->+1 (clip space) and reverse it so y is down 
	vec2 clipSpace = (zeroToTwo - 1.0) * vec2(1, -1);

	gl_Position = vec4(clipSpace, 0, 1);
}
`;

		const fragmentShaderSource = `#version 300 es
// fragment shaders don't have a default precision so we need
// to pick one. highp is a good default. It means "high precision"
precision highp float;

uniform vec4 u_color;

// we need to declare an output for the fragment shader
out vec4 outColor;

void main() {
	// Just set the output to a constant reddish-purple
	outColor = u_color;
}
`;
		const programInfo = twgl.createProgramInfo(gl, [vertexShaderSource, fragmentShaderSource]);

		twgl.resizeCanvasToDisplaySize(gl.canvas);
		gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);

		const uniforms = {
			u_resolution: [gl.canvas.width, gl.canvas.height],
			u_color: [0, 0, 0, 1]
		};

		gl.clearColor(0, 0, 0, 0);
		gl.clear(gl.COLOR_BUFFER_BIT);

		gl.useProgram(programInfo.program);
		let bufferInfo;
		// draw 50 random rectangles in random colors
		for (var ii = 0; ii < 50; ++ii) {
			const x = randomInt(300);
			const y = randomInt(300);
			const width = randomInt(300);
			const height = randomInt(300);
			var x1 = x;
			var x2 = x + width;
			var y1 = y;
			var y2 = y + height;
			const arrays = {
				// a_position: [x1, y1, x2, y1, x1, y2, x1, y2, x2, y1, x2, y2]
				a_position: {numComponents: 2, data: [x1, y1, x2, y1, x1, y2, x1, y2, x2, y1, x2, y2]}
			};

			bufferInfo = twgl.createBufferInfoFromArrays(gl, arrays, bufferInfo);
			twgl.setBuffersAndAttributes(gl, programInfo, bufferInfo);
			console.log(bufferInfo);

			uniforms.u_color = [Math.random(), Math.random(), Math.random(), 1];
			twgl.setUniforms(programInfo, uniforms);

			twgl.drawBufferInfo(gl, bufferInfo, gl.TRIANGLES);
		}

		// function update() {
		// 	requestAnimationFrame(update);
		// 	if (gl) {
		// 		if (webglUtils.resizeCanvasToDisplaySize(gl.canvas)) {
		// 			gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
		// 		}
		// 	}
		// }
		// requestAnimationFrame(update);
	});
</script>

<canvas id="c2" style="width:100%; height: 100%; display: block;" />
