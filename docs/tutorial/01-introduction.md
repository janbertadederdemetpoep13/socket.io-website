---
title: Tutorial - Introduction
sidebar_label: Introduction
slug: introduction
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Getting started

Welcome to the Socket.IO tutorial!

In this tutorial we'll create a basic chat application. It requires almost no basic prior knowledge of Node.JS or Socket.IO, so it’s ideal for users of all knowledge levels.

## Introduction

Writing a chat application with popular web applications stacks like LAMP (PHP) has normally been very hard. It involves polling the server for changes, keeping track of timestamps, and it’s a lot slower than it should be.

Sockets have traditionally been the solution around which most real-time chat systems are architected, providing a bi-directional communication channel between a client and a server.

This means that the server can *push* messages to clients. Whenever you write a chat message, the idea is that the server will get it and push it to all other connected clients.

## How to use this tutorial

### Tooling

Any text editor (from a basic text editor to a complete IDE such as [VS Code](https://code.visualstudio.com/)) should be sufficient to complete this tutorial.

Additionally, at the end of each step you will find a link to some online platforms ([CodeSandbox](https://codesandbox.io) and [StackBlitz](https://stackblitz.com), namely), allowing you to run the code directly from your browser:

![Screenshot of the CodeSandbox platform](/images/codesandbox.png)

### Syntax settings

In the Node.js world, there are two ways to import modules:

- the standard way: ECMAScript modules (or ESM)

```js
import { Server } from "socket.io";
```

Reference: https://nodejs.org/api/esm.html

- the legacy way: CommonJS

```js
const { Server } = require("socket.io");
```

Reference: https://nodejs.org/api/modules.html

Socket.IO supports both syntax. 

:::tip

We recommend using the ESM syntax in your project, though this might not always be feasible due to some packages not supporting this syntax.

:::

For your convenience, throughout the tutorial, each code block allows you to select your preferred syntax:

<Tabs groupId="lang">
  <TabItem value="cjs" label="CommonJS" default>

```js
const { Server } = require("socket.io");
```

  </TabItem>
  <TabItem value="mjs" label="ES modules">

```js
import { Server } from "socket.io";
```

  </TabItem>
</Tabs>const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

let players = {};

app.get("/", (req, res) => {
res.send(`
<!DOCTYPE html>
<html>
<head>
<title>3D GTA Mini</title>
<style>body { margin:0; overflow:hidden; }</style>
</head>
<body>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
<script src="/socket.io/socket.io.js"></script>

<script>
const socket = io();

// === THREE SETUP ===
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Licht
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(10,20,10);
scene.add(light);

// grond
const ground = new THREE.Mesh(
    new THREE.PlaneGeometry(2000,2000),
    new THREE.MeshStandardMaterial({color:0x228B22})
);
ground.rotation.x = -Math.PI/2;
scene.add(ground);

// spelers
let players = {};
let meshes = {};

// maak speler
function createPlayer(id){
    const geo = new THREE.BoxGeometry(2,4,2);
    const mat = new THREE.MeshStandardMaterial({
        color: id === socket.id ? 0xff0000 : 0x0000ff
    });
    const mesh = new THREE.Mesh(geo, mat);
    scene.add(mesh);
    meshes[id] = mesh;
}

// stad genereren
for(let i=0;i<200;i++){
    const building = new THREE.Mesh(
        new THREE.BoxGeometry(
            5 + Math.random()*10,
            10 + Math.random()*50,
            5 + Math.random()*10
        ),
        new THREE.MeshStandardMaterial({color:0x888888})
    );

    building.position.x = (Math.random()-0.5)*1000;
    building.position.z = (Math.random()-0.5)*1000;
    building.position.y = building.geometry.parameters.height/2;

    scene.add(building);
}

// sockets
socket.on("currentPlayers", data=>{
    players = data;
    for(let id in players){
        createPlayer(id);
    }
});

socket.on("newPlayer", data=>{
    players[data.id] = data.player;
    createPlayer(data.id);
});

socket.on("updatePlayers", data=>{
    players = data;
});

// movement
let keys = {};
document.addEventListener("keydown", e=> keys[e.key]=true);
document.addEventListener("keyup", e=> keys[e.key]=false);

function update(){
    let move = {x:0, z:0};

    if(keys["w"]) move.z -= 0.5;
    if(keys["s"]) move.z += 0.5;
    if(keys["a"]) move.x -= 0.5;
    if(keys["d"]) move.x += 0.5;

    socket.emit("move", move);
}

// render loop
function animate(){
    requestAnimationFrame(animate);

    update();

    for(let id in players){
        let p = players[id];
        if(meshes[id]){
            meshes[id].position.set(p.x,2,p.z);
        }
    }

    // camera volgt speler
    if(meshes[socket.id]){
        let me = meshes[socket.id];
        camera.position.x = me.position.x + 10;
        camera.position.y = me.position.y + 10;
        camera.position.z = me.position.z + 10;
        camera.lookAt(me.position);
    }

    renderer.render(scene, camera);
}
animate();
</script>
</body>
</html>
`);
});

// socket server
io.on("connection", socket=>{
    players[socket.id] = {
        x: Math.random()*50,
        z: Math.random()*50
    };

    socket.emit("currentPlayers", players);
    socket.broadcast.emit("newPlayer", {
        id: socket.id,
        player: players[socket.id]
    });

    socket.on("move", data=>{
        if(players[socket.id]){
            players[socket.id].x += data.x;
            players[socket.id].z += data.z;
        }
        io.emit("updatePlayers", players);
    });

    socket.on("disconnect", ()=>{
        delete players[socket.id];
        io.emit("updatePlayers", players);
    });
});

server.listen(3000, ()=>{
    console.log("👉 http://localhost:3000");
});

Ready? Click "Next" to g
et started.
