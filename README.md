# METRO-DYING-LIGHT-MIX-game
a game that has Dying light mixed with METRO Redux, only for PC
<!DOCTYPE html>
<html>
<head>
<title>Wasteland Runner</title>
<style>

body{
margin:0;
overflow:hidden;
background:black;
font-family:monospace;
}

#hud{
position:absolute;
top:10px;
left:10px;
color:white;
}

#crosshair{
position:absolute;
left:50%;
top:50%;
transform:translate(-50%,-50%);
color:white;
font-size:20px;
}

</style>
</head>

<body>

<div id="hud">
Health: <span id="health">100</span><br>
Stamina: <span id="stamina">100</span><br>
Ammo: <span id="ammo">30</span><br>
Battery: <span id="battery">100</span><br>
Radiation: <span id="rad">0</span>
</div>

<div id="crosshair">+</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>

let scene = new THREE.Scene()

let camera = new THREE.PerspectiveCamera(
75,
window.innerWidth/window.innerHeight,
0.1,
2000
)

let renderer = new THREE.WebGLRenderer()

renderer.setSize(window.innerWidth,window.innerHeight)
document.body.appendChild(renderer.domElement)

let light = new THREE.DirectionalLight(0xffffff,1)
light.position.set(10,20,10)
scene.add(light)

let ambient = new THREE.AmbientLight(0x404040)
scene.add(ambient)

let flashlight = new THREE.SpotLight(0xffffff,2,30,0.5)

flashlight.visible=false
scene.add(flashlight)
scene.add(flashlight.target)

let floorGeo = new THREE.PlaneGeometry(4000,4000)
let floorMat = new THREE.MeshStandardMaterial({color:0x222222})
let floor = new THREE.Mesh(floorGeo,floorMat)

floor.rotation.x = -Math.PI/2
scene.add(floor)

let buildings=[]

function createCity(){

for(let i=0;i<200;i++){

let geo = new THREE.BoxGeometry(
Math.random()*10+5,
Math.random()*40+10,
Math.random()*10+5
)

let mat = new THREE.MeshStandardMaterial({color:0x444444})

let b = new THREE.Mesh(geo,mat)

b.position.x=(Math.random()-0.5)*1500
b.position.z=(Math.random()-0.5)*1500
b.position.y=geo.parameters.height/2

scene.add(b)
buildings.push(b)

}

}

createCity()

let radiationZones=[]

for(let i=0;i<10;i++){

let geo = new THREE.CylinderGeometry(20,20,1,32)
let mat = new THREE.MeshBasicMaterial({color:0x00ff00,transparent:true,opacity:0.3})

let r = new THREE.Mesh(geo,mat)

r.position.x=(Math.random()-0.5)*800
r.position.z=(Math.random()-0.5)*800
r.position.y=0.5

scene.add(r)

radiationZones.push(r)

}

let zombies=[]

function spawnZombies(){

for(let i=0;i<40;i++){

let geo = new THREE.BoxGeometry(1,2,1)
let mat = new THREE.MeshStandardMaterial({color:0x00aa00})

let z = new THREE.Mesh(geo,mat)

z.position.x=(Math.random()-0.5)*600
z.position.z=(Math.random()-0.5)*600
z.position.y=1

scene.add(z)
zombies.push(z)

}

}

spawnZombies()

let player={
health:100,
stamina:100,
ammo:30,
battery:100,
rad:0,
vy:0,
onGround:true
}

camera.position.y=2

let velocity=new THREE.Vector3()
let direction=new THREE.Vector3()

let keys={}

document.addEventListener("keydown",e=>{

keys[e.key.toLowerCase()]=true

if(e.key==="f"){

flashlight.visible=!flashlight.visible

}

})

document.addEventListener("keyup",e=>keys[e.key.toLowerCase()]=false)

document.body.addEventListener("click",()=>{

document.body.requestPointerLock()

})

let yaw=0
let pitch=0

document.addEventListener("mousemove",e=>{

if(document.pointerLockElement===document.body){

yaw-=e.movementX*0.002
pitch-=e.movementY*0.002

pitch=Math.max(-Math.PI/2,Math.min(Math.PI/2,pitch))

camera.rotation.set(pitch,yaw,0)

}

})

document.addEventListener("mousedown",()=>{

if(player.ammo<=0) return

player.ammo--

let ray=new THREE.Raycaster()

ray.setFromCamera(new THREE.Vector2(0,0),camera)

let hit=ray.intersectObjects(zombies)

if(hit.length>0){

scene.remove(hit[0].object)
zombies.splice(zombies.indexOf(hit[0].object),1)

}

})

let clock=new THREE.Clock()

let day=0

function update(){

let delta=clock.getDelta()

let speed=10

if(keys["shift"] && player.stamina>0){

speed=20
player.stamina-=20*delta

}else{

player.stamina+=10*delta

}

player.stamina=Math.min(100,player.stamina)

direction.z=Number(keys["w"])-Number(keys["s"])
direction.x=Number(keys["d"])-Number(keys["a"])

direction.normalize()

velocity.x-=velocity.x*10*delta
velocity.z-=velocity.z*10*delta

velocity.x+=direction.x*speed*delta
velocity.z+=direction.z*speed*delta

camera.position.x+=velocity.x
camera.position.z+=velocity.z

if(keys[" "] && player.onGround){

player.vy=8
player.onGround=false

}

player.vy-=20*delta
camera.position.y+=player.vy*delta

if(camera.position.y<2){

camera.position.y=2
player.vy=0
player.onGround=true

}

zombies.forEach(z=>{

let dx=camera.position.x-z.position.x
let dz=camera.position.z-z.position.z

let dist=Math.sqrt(dx*dx+dz*dz)

if(dist<60){

z.position.x+=dx/dist*delta*4
z.position.z+=dz/dist*delta*4

}

if(dist<2){

player.health-=10*delta

}

})

radiationZones.forEach(r=>{

let dx=camera.position.x-r.position.x
let dz=camera.position.z-r.position.z

let dist=Math.sqrt(dx*dx+dz*dz)

if(dist<20){

player.rad+=20*delta
player.health-=5*delta

}

})

if(flashlight.visible){

player.battery-=5*delta

if(player.battery<=0){

flashlight.visible=false
player.battery=0

}

}

flashlight.position.copy(camera.position)
flashlight.target.position.set(
camera.position.x+Math.sin(yaw),
camera.position.y,
camera.position.z+Math.cos(yaw)
)

day+=delta*0.05

let brightness=(Math.sin(day)+1)/2

light.intensity=brightness+0.2

document.getElementById("health").innerText=Math.floor(player.health)
document.getElementById("stamina").innerText=Math.floor(player.stamina)
document.getElementById("ammo").innerText=player.ammo
document.getElementById("battery").innerText=Math.floor(player.battery)
document.getElementById("rad").innerText=Math.floor(player.rad)

saveGame()

}

function saveGame(){

localStorage.setItem("wasteSave",JSON.stringify(player))

}

function loadGame(){

let data=localStorage.getItem("wasteSave")

if(!data) return

let s=JSON.parse(data)

player.health=s.health
player.ammo=s.ammo
player.stamina=s.stamina

}

loadGame()

function animate(){

update()

renderer.render(scene,camera)

requestAnimationFrame(animate)

}

animate()

</script>

</body>
</html>
