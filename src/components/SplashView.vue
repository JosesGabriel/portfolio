<template>
  <section class="flex">
    <div class="canvas" ref="container"></div>
    <div class="flex canvas__text">
      <div class="splash__heading">
        <div class="header__text">Software engineer.</div>
        <br /><br />
        <a class="nuxify__link" href="https://nuxify.tech/" target="_blank"
          >Nuxify</a
        >
        Chief Operating Officer.
      </div>
    </div>
  </section>
</template>

<script>
import * as THREE from "three";
export default {
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      mesh: null,
      stars: null,
      starsGeo: null,
      starImg: require("../assets/images/star.png"),
    };
  },
  mounted() {
    this.initialize();
  },
  methods: {
    initialize() {
      let container = this.$refs["container"];
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0x000000);
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      this.camera.position.z = 2;
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      container.append(this.renderer.domElement);

      const vertexShader = `
          varying vec3 vNormal;
          uniform float uTime;
          uniform float uSpeed;
          uniform float uNoiseDensity;
          uniform float uNoiseStrength;

          vec3 mod289(vec3 x)
          {
            return x - floor(x * (1.0 / 289.0)) * 289.0;
          }

          vec4 mod289(vec4 x)
          {
            return x - floor(x * (1.0 / 289.0)) * 289.0;
          }

          vec4 permute(vec4 x)
          {
            return mod289(((x*34.0)+1.0)*x);
          }

          vec4 taylorInvSqrt(vec4 r)
          {
            return 1.79284291400159 - 0.85373472095314 * r;
          }

          vec3 fade(vec3 t) {
            return t*t*t*(t*(t*6.0-15.0)+10.0);
          }

          // Classic Perlin noise, periodic variant
          float pnoise(vec3 P, vec3 rep)
          {
            vec3 Pi0 = mod(floor(P), rep); // Integer part, modulo period
            vec3 Pi1 = mod(Pi0 + vec3(1.0), rep); // Integer part + 1, mod period
            Pi0 = mod289(Pi0);
            Pi1 = mod289(Pi1);
            vec3 Pf0 = fract(P); // Fractional part for interpolation
            vec3 Pf1 = Pf0 - vec3(1.0); // Fractional part - 1.0
            vec4 ix = vec4(Pi0.x, Pi1.x, Pi0.x, Pi1.x);
            vec4 iy = vec4(Pi0.yy, Pi1.yy);
            vec4 iz0 = Pi0.zzzz;
            vec4 iz1 = Pi1.zzzz;

            vec4 ixy = permute(permute(ix) + iy);
            vec4 ixy0 = permute(ixy + iz0);
            vec4 ixy1 = permute(ixy + iz1);

            vec4 gx0 = ixy0 * (1.0 / 7.0);
            vec4 gy0 = fract(floor(gx0) * (1.0 / 7.0)) - 0.5;
            gx0 = fract(gx0);
            vec4 gz0 = vec4(0.5) - abs(gx0) - abs(gy0);
            vec4 sz0 = step(gz0, vec4(0.0));
            gx0 -= sz0 * (step(0.0, gx0) - 0.5);
            gy0 -= sz0 * (step(0.0, gy0) - 0.5);

            vec4 gx1 = ixy1 * (1.0 / 7.0);
            vec4 gy1 = fract(floor(gx1) * (1.0 / 7.0)) - 0.5;
            gx1 = fract(gx1);
            vec4 gz1 = vec4(0.5) - abs(gx1) - abs(gy1);
            vec4 sz1 = step(gz1, vec4(0.0));
            gx1 -= sz1 * (step(0.0, gx1) - 0.5);
            gy1 -= sz1 * (step(0.0, gy1) - 0.5);

            vec3 g000 = vec3(gx0.x,gy0.x,gz0.x);
            vec3 g100 = vec3(gx0.y,gy0.y,gz0.y);
            vec3 g010 = vec3(gx0.z,gy0.z,gz0.z);
            vec3 g110 = vec3(gx0.w,gy0.w,gz0.w);
            vec3 g001 = vec3(gx1.x,gy1.x,gz1.x);
            vec3 g101 = vec3(gx1.y,gy1.y,gz1.y);
            vec3 g011 = vec3(gx1.z,gy1.z,gz1.z);
            vec3 g111 = vec3(gx1.w,gy1.w,gz1.w);

            vec4 norm0 = taylorInvSqrt(vec4(dot(g000, g000), dot(g010, g010), dot(g100, g100), dot(g110, g110)));
            g000 *= norm0.x;
            g010 *= norm0.y;
            g100 *= norm0.z;
            g110 *= norm0.w;
            vec4 norm1 = taylorInvSqrt(vec4(dot(g001, g001), dot(g011, g011), dot(g101, g101), dot(g111, g111)));
            g001 *= norm1.x;
            g011 *= norm1.y;
            g101 *= norm1.z;
            g111 *= norm1.w;

            float n000 = dot(g000, Pf0);
            float n100 = dot(g100, vec3(Pf1.x, Pf0.yz));
            float n010 = dot(g010, vec3(Pf0.x, Pf1.y, Pf0.z));
            float n110 = dot(g110, vec3(Pf1.xy, Pf0.z));
            float n001 = dot(g001, vec3(Pf0.xy, Pf1.z));
            float n101 = dot(g101, vec3(Pf1.x, Pf0.y, Pf1.z));
            float n011 = dot(g011, vec3(Pf0.x, Pf1.yz));
            float n111 = dot(g111, Pf1);

            vec3 fade_xyz = fade(Pf0);
            vec4 n_z = mix(vec4(n000, n100, n010, n110), vec4(n001, n101, n011, n111), fade_xyz.z);
            vec2 n_yz = mix(n_z.xy, n_z.zw, fade_xyz.y);
            float n_xyz = mix(n_yz.x, n_yz.y, fade_xyz.x);
            return 2.2 * n_xyz;
          }
          
          void main() {
            float t = uTime * uSpeed;
            float distortion = pnoise((normal + t) * uNoiseDensity, vec3(10.0)) * uNoiseStrength;

            vec3 pos = position + (normal * distortion);
            
            vNormal = normal;

            gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.);
          }`;
      const fragmentShader = `
        varying vec3 vNormal;
        uniform float uTime;
        void main() {
          vec3 color = vec3(1.0);
          gl_FragColor = vec4(vNormal, 1.0);
        }`;
      const geometry = new THREE.IcosahedronBufferGeometry(1, 32);
      const material = new THREE.ShaderMaterial({
        vertexShader,
        fragmentShader,
        uniforms: {
          uTime: { value: 0 },
          uSpeed: { value: 0.3 },
          uNoiseDensity: { value: 1.5 },
          uNoiseStrength: { value: 0.3 },
        },
        wireframe: false,
      });
      const mesh = new THREE.Mesh(geometry, material);
      mesh.rotateY(5.7);
      this.scene.add(mesh);
      const clock = new THREE.Clock();
      const tick = () => {
        const elapsedTime = clock.getElapsedTime();
        // Update Uniforms
        material.uniforms.uTime.value = elapsedTime;
        material.uniforms.uSpeed.value = 0.3;
        material.uniforms.uNoiseStrength.value = 0.3;
        material.uniforms.uNoiseDensity.value = 1.5;
        // Render
        this.renderer.render(this.scene, this.camera);
        // Call tick again on the next frame
        requestAnimationFrame(tick);
      };
      tick();
      window.addEventListener("resize", this.onWindowResize, false);
    },

    onWindowResize() {
      const sizes = {
        width: window.innerWidth,
        height: window.innerHeight,
      };
      // Update sizes
      this.sizes.width = window.innerWidth;
      this.sizes.height = window.innerHeight;

      // Update camera
      this.camera.aspect = sizes.width / sizes.height;
      this.camera.updateProjectionMatrix();

      // Update renderer
      this.renderer.setSize(sizes.width, sizes.height);
      this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    },
  },
};
</script>

<style scoped>
.header__text {
  font-family: poppins-regular;
  background-image: linear-gradient(to right, #dd62bca9, #d58b0a6f);
  border-radius: 5px;
  display: inline;
  padding: 5px;
}
.canvas {
  height: 100%;
  left: 0;
  position: absolute;
  top: 0;
  width: 100vw;
  z-index: 0;
}
.canvas__text {
  font-family: poppins-regular;
  height: 100vh;
  color: white;
  font-size: 2.4rem;
  font-weight: 900;
  width: 60vw;
  bottom: 3vh;
  z-index: 1;
}
.nuxify__link {
  color: white;
  text-decoration: underline;
  text-decoration-color: white;
  background-image: linear-gradient(to right, #dd62bca9, #d58b0a6f);
  border-radius: 5px;
  text-decoration: none;
  padding: 5px;
}
.nuxify__link:hover {
  background-color: rgb(219, 45, 190);
  -webkit-transition: background-color 200ms linear;
  -ms-transition: background-color 200ms linear;
  transition: background-color 200ms linear;
}
@media screen and (max-width: 768px) {
  .canvas__text {
    font-size: 2rem;
  }
}
@media screen and (max-width: 480px) {
  .canvas__text {
    font-size: 1.3rem;
  }
}
</style>
