<script>
    import * as THREE from "three"
    import {onMount} from "svelte";
    import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"

    let scene, etoiles = null, etoilesGeom = null, etoilesMat = null

    const parameters = {
        nb: 10000,
        taille: 0.02,
        branche: 3,
        rayon: 5,
        spin: 1,
        aleatoire: 0.02,
        puissance: 4,
        couleur1: '#ff6030',
        couleur2: '#1b3984',
    }



    const clearGalaxy = () => {
        etoilesGeom.dispose()
        etoilesMat.dispose()
        scene.remove(etoiles)
    }

    const generateGalaxy = () => {
        if(etoiles !== null){
            clearGalaxy()
        }
        etoilesGeom = new THREE.BufferGeometry()
        const positions = new Float32Array(parameters.nb * 3)
        const colors = new Float32Array(parameters.nb * 3)

        const color1 = new THREE.Color(parameters.couleur1)
        const color2 = new THREE.Color(parameters.couleur2)


        for (let i = 0; i < parameters.nb; i++ ){
            let i3 = i * 3

            const radius = Math.random() * parameters.rayon
            const angleBranches = ((i % parameters.branche) / parameters.branche) * Math.PI * 2 + (radius * parameters.spin)

            //((Math.random() - 0.5) * parameters.aleatoire) * Math.log10(radius)

            const randomX = Math.pow(Math.random(), parameters.puissance) * (Math.random() < 0.5 ? -1 : 1)
            const randomY = Math.pow(Math.random(), parameters.puissance) * (Math.random() < 0.5 ? -1 : 1)
            const randomZ = Math.pow(Math.random(), parameters.puissance) * (Math.random() < 0.5 ? -1 : 1)

            positions[i3] = Math.cos(angleBranches) * radius + randomX
            positions[i3 + 1] = Math.random() * 0.1 + randomY
            positions[i3 + 2] = Math.sin(angleBranches) * radius + randomZ

            const mixed = color1.clone()
            mixed.lerp(color2, radius/parameters.rayon)

            colors[i3] = mixed.r
            colors[i3 + 1] = mixed.g
            colors[i3 + 2] = mixed.b

        }

        etoilesGeom.setAttribute('position', new THREE.BufferAttribute(positions, 3))
        etoilesGeom.setAttribute('color', new THREE.BufferAttribute(colors, 3))

        etoilesMat = new THREE.PointsMaterial({
            size: parameters.taille,
            sizeAttenuation: true,
            depthWrite: false,
            blending: THREE.AdditiveBlending,
            vertexColors: true
        })

        etoiles = new THREE.Points(etoilesGeom, etoilesMat)
        scene.add(etoiles)

    }

    onMount( () => {
        const canvas = document.querySelector('canvas.webgl')

        scene = new THREE.Scene()

        const textureLoader = new THREE.TextureLoader()

        const sizes = {
            width: window.innerWidth,
            height: window.innerHeight/2
        }

        window.addEventListener('resize', () =>
        {
            sizes.width = window.innerWidth
            sizes.height = window.innerHeight/2

            camera.aspect = sizes.width / sizes.height
            camera.updateProjectionMatrix()

            renderer.setSize(sizes.width, sizes.height)
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
        })

        const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
        camera.position.x = 4
        camera.position.y = 2
        camera.position.z = 5
        scene.add(camera)

        const controls = new OrbitControls(camera, canvas)
        controls.enableDamping = true

        const renderer = new THREE.WebGLRenderer({
            canvas: canvas,

        })
        renderer.setSize(sizes.width - 20, sizes.height - 100)
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

        renderer.shadowMap.enabled = true
        renderer.shadowMap.type = THREE.PCFSoftShadowMap

        const clock = new THREE.Clock()

        const tick = () =>
        {
            const elapsedTime = clock.getElapsedTime()
            controls.update()
            renderer.render(scene, camera)
            window.requestAnimationFrame(tick)
        }

        tick()
        generateGalaxy()
    });
</script>


<div class="w-full flex flex-col justify-between ">
    <canvas class="webgl"></canvas>
    <div class="gui bg-white rounded-md p-10 flex flex-col w-full ">
        <h2 class="text-2xl">Générateur de galaxie</h2>
        <label>Nombre d'étoiles (={parameters.nb})</label>
        <input type="range" min="100" max="100000" step="100" bind:value={parameters.nb} on:mouseup={generateGalaxy} class="range range-accent">
        <label>Taille des étoiles  (={parameters.taille})</label>
        <input type="range" min="0.01" max="0.1" step="0.01"  bind:value={parameters.taille} on:mouseup={generateGalaxy} class="range range-accent">
        <label>Rayon  (={parameters.rayon})</label>
        <input type="range" min="0.01" max="20" step="0.01"  bind:value={parameters.rayon} on:mouseup={generateGalaxy} class="range range-accent">
        <label>Nombre de branches  (={parameters.branche})</label>
        <input type="range" min="1" max="10" step="1"  bind:value={parameters.branche} on:mouseup={generateGalaxy} class="range range-accent">
        <label>Spin  (={parameters.spin})</label>
        <input type="range" min="-5" max="5" step="0.01"  bind:value={parameters.spin} on:mouseup={generateGalaxy} class="range range-accent">
        <label>Aléatoirisation  (={parameters.aleatoire})</label>
        <input type="range" min="0" max="2" step="0.001"  bind:value={parameters.aleatoire} on:change={generateGalaxy} class="range range-accent">
        <label>Puissance  (={parameters.puissance})</label>
        <input type="range" min="1" max="10" step="0.001"  bind:value={parameters.puissance} on:change={generateGalaxy} class="range range-accent">
        <label>Couleur intérieur </label>
        <input type="color" bind:value={parameters.couleur1} on:change={generateGalaxy} >
        <label>Couleur extérieur </label>
        <input type="color" bind:value={parameters.couleur2} on:change={generateGalaxy} >
    </div>
</div>
