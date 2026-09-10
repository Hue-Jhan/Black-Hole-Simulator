

# 🕳️ Gargantua | Black Hole Simulator

A real-time black hole renderer in WebGL. The bent light, glowing disk, and the warped background runs inside a single GLSL shader that traces curved light paths around a black hole. Vibecoded with Sonnet 5.

# 🖥️ Code

Run it by clicking the link on the right or with this:

    python3 -m http.server 8000

The `index.html` file uses Three.js simply to set up a flat, blank canvas on your screen. JavaScript only handles your mouse controls and updates the camera angle. 

The GPU does all the heavy lifting inside a single GLSL fragment shader, there are no actual 3D models, polygons, or textures in this project, instead, the code uses a technique called raymarching: for every single pixel on your monitor, the graphics card shoots a virtual ray of light into the scene, it calculates the math step by step, bending the ray through the black hole's gravity field until it either crashes into the glowing disk, gets swallowed by the event horizon, or escapes to hit a background star. 

Because GPUs are designed to solve thousands of math problems at the exact same time, they can calculate these millions of curved light paths 60 times a second directly in your browser.

# 🌠 The Physics

The math is scaled so the black hole's event horizon (the point of no return, `r_s`) has a radius of `1`. Everything else is measured relative to that.

### Bending Light
Instead of using insane 4D spacetime math, the code fakes the light bending perfectly in 3D by applying a "pull" to each light ray as it travels:

    a(pos) = -1.5 · r_s · |h|² / r⁵ · pos
    h = pos₀ × dir₀

Because this pull always points straight at the black hole, the light path stays totally flat, just like in real physics. The code traces the ray until it either falls into the black hole (goes black) or escapes into space (hits a star). This naturally creates that famous "Einstein ring" halo without us having to draw a circle manually.

### Accretion Disk
Matter can't orbit too close to a black hole without falling in, the closest safe distance is the ISCO:

    r_isco = 3 · r_s

Our glowing disk of gas starts exactly there, which leaves a realistic dark gap between the fire and the black hole. The gas moves much faster on the inside so we color it white, on the outside it's dark red.

### Relativistic Beaming (Doppler Effect)
Because the disk is spinning incredibly fast, the side coming towards you looks brighter and slightly bluer, while the side spinning away looks dimmer and redder. The shader calculates this Doppler effect using:

    D = 1 / (1 - v · cos(θ))
    I_obs = D³ · I_emit

### The Background Sky
The code builds a space environment filled with starfields, colored nebulas, random comets, and a few giant marker stars. Because the virtual light rays are bent by gravity before they reach this background, the stars/nebulas get stretched, creating optical illusions when you orbit the camera around the black hole.

---
