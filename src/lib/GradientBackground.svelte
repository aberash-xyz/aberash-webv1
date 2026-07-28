<script>
	import { onMount } from 'svelte';

	let canvas;
	let ready = false;

	const VERT = `#version 300 es
	in vec2 p; out vec2 vUv;
	void main(){ vUv=p*0.5+0.5; gl_Position=vec4(p,0.,1.); }`;

	const FRAG = `#version 300 es
	precision highp float;
	in vec2 vUv; out vec4 o;
	uniform float uT, uA; uniform vec2 uM;

	float hash(vec2 p){ p=fract(p*vec2(123.34,456.21)); p+=dot(p,p+45.32); return fract(p.x*p.y); }
	float noise(vec2 p){ vec2 i=floor(p),f=fract(p);
		float a=hash(i),b=hash(i+vec2(1,0)),c=hash(i+vec2(0,1)),d=hash(i+vec2(1,1));
		vec2 u=f*f*(3.-2.*f); return mix(mix(a,b,u.x),mix(c,d,u.x),u.y); }
	const mat2 M=mat2(1.6,1.2,-1.2,1.6);
	float fbm(vec2 p){ float s=0.,a=.5; for(int i=0;i<5;i++){ s+=a*noise(p); p=M*p; a*=.5; } return s; }

	void main(){
		vec2 uv=vUv; vec2 p=uv; p.x*=uA;
		// gentle cursor influence — alive, not gimmicky
		vec2 md=uv-uM; float infl=exp(-dot(md,md)*7.0);
		p += md*infl*0.45;

		float t=uT*0.035;
		vec2 q=vec2(fbm(p*1.5+vec2(0.,0.)+t), fbm(p*1.5+vec2(5.2,1.3)-t));
		vec2 r=vec2(fbm(p*1.5+1.7*q+vec2(1.7,9.2)+0.15*t), fbm(p*1.5+1.7*q+vec2(8.3,2.8)-0.13*t));
		float f=fbm(p*1.5+2.0*r);

		vec3 navy =vec3(0.020,0.030,0.060);
		vec3 blue =vec3(0.055,0.120,0.290);
		vec3 ember=vec3(0.860,0.340,0.110);
		vec3 bright=vec3(0.970,0.640,0.290);

		// bias warmth toward the lower-left; keep centre + upper-right dark for the type
		float ll = pow((1.0-uv.x)*0.6 + (1.0-uv.y)*0.6, 1.3);
		vec3 col=navy;
		col=mix(col, blue,  clamp(f*f*2.2,0.,1.));
		col=mix(col, ember, clamp(length(r)*1.0*ll,0.,1.));
		col=mix(col, bright,clamp(r.x*r.x*1.2*ll,0.,1.));

		// keep the centre calm so "aberash." reads
		float dc=length((uv-0.5)*vec2(uA,1.0));
		col*=mix(0.5,1.0, smoothstep(0.12,0.6,dc));
		// edge vignette
		col*=mix(0.7,1.0, smoothstep(1.25,0.4,length((uv-0.5)*vec2(1.1,1.0))));

		o=vec4(col,1.0);
	}`;

	onMount(() => {
		const REDUCE = matchMedia('(prefers-reduced-motion: reduce)').matches;
		const gl = canvas.getContext('webgl2');

		// no webgl2 — leave the canvas transparent, the CSS gradient underneath stands in
		if (!gl) return;

		const sh = (type, src) => {
			const x = gl.createShader(type);
			gl.shaderSource(x, src);
			gl.compileShader(x);
			if (!gl.getShaderParameter(x, gl.COMPILE_STATUS)) throw gl.getShaderInfoLog(x);
			return x;
		};

		const prog = gl.createProgram();
		gl.attachShader(prog, sh(gl.VERTEX_SHADER, VERT));
		gl.attachShader(prog, sh(gl.FRAGMENT_SHADER, FRAG));
		gl.bindAttribLocation(prog, 0, 'p');
		gl.linkProgram(prog);
		gl.useProgram(prog);

		const vao = gl.createVertexArray();
		gl.bindVertexArray(vao);
		const vbo = gl.createBuffer();
		gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
		gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1, -1, 3, -1, -1, 3]), gl.STATIC_DRAW);
		gl.enableVertexAttribArray(0);
		gl.vertexAttribPointer(0, 2, gl.FLOAT, false, 0, 0);

		const uT = gl.getUniformLocation(prog, 'uT');
		const uA = gl.getUniformLocation(prog, 'uA');
		const uM = gl.getUniformLocation(prog, 'uM');

		let W, H;
		const resize = () => {
			const dpr = Math.min(devicePixelRatio || 1, 2);
			W = innerWidth;
			H = innerHeight;
			canvas.width = W * dpr;
			canvas.height = H * dpr;
			gl.viewport(0, 0, canvas.width, canvas.height);
		};
		resize();

		let mx = 0.5,
			my = 0.5,
			tmx = 0.5,
			tmy = 0.5;
		const onPointerMove = (e) => {
			tmx = e.clientX / innerWidth;
			tmy = 1 - e.clientY / innerHeight;
		};

		window.addEventListener('resize', resize);
		window.addEventListener('pointermove', onPointerMove, { passive: true });

		const t0 = performance.now();
		let raf;
		const frame = (now) => {
			mx += (tmx - mx) * 0.05;
			my += (tmy - my) * 0.05;
			gl.uniform1f(uT, REDUCE ? 12.0 : (now - t0) / 1000);
			gl.uniform1f(uA, W / H);
			gl.uniform2f(uM, mx, my);
			gl.drawArrays(gl.TRIANGLES, 0, 3);
			if (!ready) ready = true;
			if (!REDUCE) raf = requestAnimationFrame(frame);
		};
		raf = requestAnimationFrame(frame);

		return () => {
			cancelAnimationFrame(raf);
			window.removeEventListener('resize', resize);
			window.removeEventListener('pointermove', onPointerMove);
		};
	});
</script>

<div class="base"></div>
<canvas class="gradient" class:ready bind:this={canvas}></canvas>
<div class="grain"></div>

<style>
	/* static stand-in for the shader — visible until the first frame lands */
	.base {
		position: fixed;
		inset: 0;
		z-index: 0;
		background-color: #080a12;
		background-image: radial-gradient(
			120% 90% at 15% 90%,
			#d5622a 0%,
			#3a2440 30%,
			#0a1230 60%,
			#080a12 100%
		);
	}
	.gradient {
		position: fixed;
		inset: 0;
		width: 100%;
		height: 100%;
		display: block;
		z-index: 0;
		opacity: 0;
		transition: opacity 0.7s ease;
	}
	.gradient.ready {
		opacity: 1;
	}

	@media (prefers-reduced-motion: reduce) {
		.gradient {
			transition: none;
		}
	}
	.grain {
		position: fixed;
		inset: 0;
		z-index: 1;
		pointer-events: none;
		mix-blend-mode: overlay;
		opacity: 0.06;
		background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='140' height='140'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2'/></filter><rect width='100%25' height='100%25' filter='url(%23n)'/></svg>");
	}
</style>
