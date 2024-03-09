<script>
  import { T, useLoader } from '@threlte/core'
  import Bulb from "./Bulb.svelte"
  import { OrbitControls } from "@threlte/extras"
  import { TextureLoader } from "three"
  import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';


  // const texture  = useLoader(TextureLoader).load("textures/tokyo-tower.jpeg")
  const {load} = useLoader(TextureLoader);
  // console.log(texture)
</script>

<T.PerspectiveCamera
  makeDefault
  position={[0.0,0.00,0.135]}
  on:create={({ref}) => {
    ref.lookAt(0,0,0)
  }}
    fov={50}>
    <OrbitControls enableDamping />
</T.PerspectiveCamera>

<!-- <T.Mesh>
  <T.BoxGeometry />
  <T.MeshStandardMaterial color="hotpink"/>
</T.Mesh> -->
<Bulb />

<!-- Plane with text -->
<T.Mesh  position={[0,-0.5,-3.15]}>
  <T.PlaneGeometry args={[5, 3.33]}/>
  {#await load("textures/transparentPlaneMap.png") then map }
  <T.MeshStandardMaterial {map} transparent={true} emissive={"white"}/>
  {/await}
</T.Mesh>
<!-- Plane for background -->
<T.Mesh  position={[0,-0.5,-3.35]}>
  <T.PlaneGeometry args={[10, 6.33]}/>
  <T.MeshStandardMaterial color={"#0D2818"} emissive={"#0D2818"}/>
</T.Mesh>



<T.DirectionalLight position={[1.5,0.15,0.75]} intensity={5} color={"white"}/>
<!-- <T.DirectionalLight position={[0,-0.5,-0.5]} intensity={5} color={"white"}/> -->
<!-- <T.DirectionalLight position={[0,0.0,0.5]} intensity={5} color={"white"}/> -->
<!-- <T.DirectionalLight position={[0.5,0,1.5]} intensity={1} color={"white"}/> -->
<!-- <T.DirectionalLight position={[0,2.5,2.5]} intensity={1} color={'white'}/> -->
<!-- <T.PointLight position={[0, -0.27, 0.1]} intensity={1} color={"#ffbf00"}
  power={2}
  decay={2}
/>
<T.PointLight position={[0, -0.27, -0.1]} intensity={1} color={"#ffbf00"}
  power={2}
  decay={2}
/> -->
<!-- <T.PointLight position={[-0.5, 0.5, 0.3]} intensity={5} color={"amber"} width={10} height={5}/> -->
<!-- <T.AmbientLight position={[0,0,1]} intensity={1} /> -->
<!-- <T.SpotLight position={[0, 0.5, 0.5]}  /> -->
