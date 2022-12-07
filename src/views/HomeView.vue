<template>
  <div v-loading="loading" style="position: relative">
    <div
      style="position:absolute;left:0;top:0;opacity:0.5;height: 35px;width: 100%;z-index: 9999;display: flex;justify-content: flex-start;align-items: center;padding: 0 20px">
      <button size="mini" @click="changeName('union')">公园</button>
    </div>
    <div ref="environment"></div>
  </div>
</template>
 
<script setup>
import { computed, ref, onMounted } from 'vue'
import {
  Scene,
  PerspectiveCamera,
  WebGL1Renderer,
  AmbientLight,
  CubeTextureLoader,
  Mesh,
  PlaneGeometry,
  MeshLambertMaterial,
  DoubleSide,
  TextureLoader
} from 'three'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
const loading = ref(true)
let scene = new Scene()
const environment = ref()
/**
 * 默认场景
 */
const nameDefault = ref('indoor')
/**
 * 切换场景事件
 */
const changeName = (environmentName) => {
  if (nameDefault.value === environmentName) {
    return
  }
  if (environmentName === '') {
    environmentName = nameDefault.value
  }
  loading.value = true
  scene.background = new CubeTextureLoader()
    .setPath(`/src/ambient/${environmentName}/`)
    .load(['posx.jpg', 'negx.jpg', 'posy.jpg', 'negy.jpg', 'posz.jpg', 'negz.jpg'], () => {
      loading.value = false
      nameDefault.value = environmentName
    })
}

/**
 * 初始化
 */
const init = () => {
  loading.value = true
  /**
   * 相机 PerspectiveCamera(视野大小, 视图的长宽比, 近景， 远景)
   */
  const camera = new PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 1000)
  camera.position.set(0, 0, 100)
  camera.lookAt(scene.position)

  /**
   * antialias消除锯齿
   */
  const renderer = new WebGL1Renderer({ antialias: true })
  // 背景颜色
  renderer.setClearColor(0xffffff)
  // 设置设备像素比
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.setSize(window.innerWidth, window.innerHeight)

  /**
   * 添加环境灯光
   */
  scene.add(new AmbientLight(0xf8f8f8, 2.1))

  environment.value?.appendChild(renderer.domElement)
  const planeTexture = new TextureLoader().load('/src/assets/ud_scImg.png')
  const planeGeometry = new PlaneGeometry(188 * 1.2, 165 * 1.2)
  // 材质 MeshBasicMaterial和MeshLambertMaterial的区别 MeshLambertMaterial它不会自己发光,而是需要一个光源照射
  const planeMaterial = new MeshLambertMaterial({ color: 0xabababab, side: DoubleSide, map: planeTexture })
  // 用来定位音源的网格模型
  const audioMesh = new Mesh(planeGeometry, planeMaterial);
  // 设置网格模型的位置，相当于设置音源的位置
  audioMesh.position.set(0, 0, 300);
  scene.add(audioMesh);

  window.addEventListener('resize', () => onWindowResize())

  /**
   * 轨道控制器 也就是鼠标转动等操作
   */
  let orbitControls = new OrbitControls(camera, renderer.domElement)
  orbitControls.autoRotateSpeed = 1
  orbitControls.minDistance = 500

  renderScene()
  function renderScene() {
    requestAnimationFrame(renderScene)
    renderer.render(scene, camera)
  }

  const onWindowResize = () => {
    renderer.setSize(window.innerWidth, window.innerHeight)
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
  }
}

onMounted(() => {
  init()
  changeName('')
})

</script>