<template>
  <div ref="container" class="container">
  </div>
</template>

<script setup>
import { 
	onMounted, 
	ref, 
	reactive, 
	toRaw
} from 'vue'

import {
	Scene,
	AxesHelper,
	PerspectiveCamera,
	WebGLRenderer,
	AmbientLight,
	PointLight,
	Color,
	BufferGeometry,
	BufferAttribute, 
	PointsMaterial,
	TextureLoader,
	Points,
} from 'three'

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { OBJLoader } from 'three/addons/loaders/OBJLoader.js'
import { MTLLoader } from 'three/addons/loaders/MTLLoader.js'

const container = ref(null)

const g = reactive({
	publicPath: process.env.BASE_URL,
	w: null,
	h: null,
	mouseX: null, 
	mouseY: null,
	fov: 50,
	scene: null, 
	axesHelper: null, 
	perspectiveCamera: null, 
	webGLRenderer: null, 
	OBJLoader: null,  	
	ambientLight: null,
	pointLight: null,
	textureLoader: null,
	orbitControls: null,
	TextureLoader: null,
	starsT1: null,
	starsT2: null,
	geometrys: null,
	materials: null,
	banana: null,
	MTLLoader: null
})

function init(){
	g.w = container.value.offsetWidth
	g.h = container.value.offsetHeight	
	
	g.scene = new Scene()

	g.axesHelper = new AxesHelper(10)
	g.axesHelper.visible = false
	g.scene.add(g.axesHelper)

	g.perspectiveCamera = new PerspectiveCamera(g.fov, g.w / g.h, 0.1, 1000)
	g.perspectiveCamera.translateX(-40)

	g.webGLRenderer = new WebGLRenderer()
	g.webGLRenderer.setSize(g.w, g.h)
	g.webGLRenderer.setClearColor(new Color("#1c1624"))
	g.webGLRenderer.domElement.id = "canvas"

	g.orbitControls = new OrbitControls(g.perspectiveCamera, g.webGLRenderer.domElement)
	g.orbitControls.autoRotateSpeed = 3
	g.orbitControls.autoRotate = false
	g.orbitControls.enableRotate = false
	g.orbitControls.enableZoom = false
	g.orbitControls.enablePan = false

	container.value.appendChild(g.webGLRenderer.domElement)

	g.ambientLight = new AmbientLight( 0x404040, 1 )
	g.scene.add( g.ambientLight )

	g.pointLight = new PointLight( 0x404040, 1 )
	g.perspectiveCamera.add( g.pointLight )
	g.scene.add( g.perspectiveCamera )

	g.geometrys = [new BufferGeometry(), new BufferGeometry()]
	g.geometrys[0].setAttribute("position", new BufferAttribute(getRandomParticelPos(1200), 3))
  g.geometrys[1].setAttribute("position", new BufferAttribute(getRandomParticelPos(1200), 3))

	g.textureLoader = new TextureLoader()
	g.materials = [ 
		new PointsMaterial({
      size: 0.1,
      map: g.textureLoader.load(`${g.publicPath}assets/star/sp1.png`),
      transparent: true,
    }),
    new PointsMaterial({
      size: 0.75,
      map: g.textureLoader.load(`${g.publicPath}assets/star/sp2.png`),
      transparent: true,
    })
  ]

	g.starsT1 = new Points(g.geometrys[0], g.materials[0])
	g.starsT2 = new Points(g.geometrys[1], g.materials[1])
  g.scene.add(g.starsT1)
  g.scene.add(g.starsT2)

	g.MTLLoader = new MTLLoader().setMaterialOptions()

	g.MTLLoader.load(`${g.publicPath}assets/banana/Banana.mtl`, (material) => {
		material.preload()
		g.OBJLoader = new OBJLoader()
		g.OBJLoader.setMaterials(material)
		g.OBJLoader.load(`${g.publicPath}assets/banana/Banana.obj`, (obj) => {	
			g.banana = obj
			g.banana.rotation.z += 90
			g.scene.add(g.banana)
		})
	})
}

function getRandomParticelPos (particleCount) {
  const arr = new Float32Array(particleCount * 3)
  for (let i = 0; i < particleCount; i++) {
    arr[i] = (Math.random() - 0.5) * 1000
  }
  return arr
}

function render(){
	g.orbitControls.update()
	g.webGLRenderer.render(toRaw(g.scene), g.perspectiveCamera)
}

function animate(){
	starsAnimate()
	bananaAnimate()
	requestAnimationFrame(animate)
	render()
}

function onWindowResize(){
	g.w = container.value.offsetWidth
	g.h = container.value.offsetHeight
	g.perspectiveCamera.aspect = g.w / g.h
	g.perspectiveCamera.updateProjectionMatrix()
	g.webGLRenderer.setSize(g.w, g.h)
}

function bananaAnimate(){
	if(g.banana) g.banana.rotation.y += 0.01
}

function starsAnimate(){
  g.starsT1.position.x = g.mouseY * 0.05
  g.starsT2.position.x = g.mouseY * 0.05
}

function onMouseEvent(e){
	g.mouseX = e.clientX
	g.mouseY = e.clientY
}

onMounted(()=> {
	init()
	animate()
	window.addEventListener('resize', onWindowResize)
	document.addEventListener("mousemove", onMouseEvent)
})

</script>

<style scoped>
.container{
	width: 100vw;
	height: 100vh;
}
	
</style>