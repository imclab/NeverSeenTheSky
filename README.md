Never Seen The Sky
===

Watch the demo here:
http://christmasexperiments.com/23/

Expect a more decent write up about this demo on acko.net soon. In the meantime, here's the Cliff's notes:

Use the included `debug.html` to examine the demo from the inside:  
[http://acko.net/files/never-seen-the-sky/git/debug.html](http://acko.net/files/never-seen-the-sky/git/debug.html)

Click and drag to look around. Type `freeze()` in the JS console to stop the script and move around freely with the WASD keys.

You can also alter the effects by changing the global exports object, e.g.:

```javascript
// Basic demo controls
exports.aurora.color1 = [.5, .3, .1];
exports.aurora.color2 = [.4, .4, .4];
exports.fade.opacity = .5;
exports.visualizer.preset = 4;

// Reset the script from the start (or set a new script!)
exports.director.live(Acko.Demo.Script)

// Even internal Three.js stuff is exposed if you dig around
// Tron mode:
exports.land.land.material.wireframe = true;
exports.moon.moon.material.wireframe = true;
exports.aurora.aurora.material.wireframe = true;
exports.earth.earth.material.wireframe = true;
```


Landscape
---

A heightmap is generated on the GPU by stacking layers of smoothly interpolated noise, similar to this:
http://www.iquilezles.org/www/articles/morenoise/morenoise.htm

A snow value is derived based on slope and some fudged formulas and a normal map is generated for per-pixel lighting. When rendering the landscape,
fake ambient occlusion is applied (just made up), small-scale noise is added to hide the low res areas,
and volumetric fog is added in the valleys.

Though the map is 2048x2048, the detail is still pretty low in practice, especially because the mesh is only 256x256,
so the map is pre-warped to concentrate pixel and vertex density in the middle.

Aurora - Fluid simulation
---

A 2D MacCormack-advection fluid solver on a 256x256 grid, similar to GPU Gems:
http://http.developer.nvidia.com/GPUGems3/gpugems3_ch30.html

To further speed it up, I don't reset the pressure field every frame, and instead iterate from
the previous frame's solution.

The advected quantities are velocity (x,y), aurora density, and 'charm', which is used to modulate color in the result.

Aurora - Volumetric rendering
---

The 2D fluid density and charm is turned into a 2.5D effect by raytracing through a box volume. But first, the 256x256 field
is upscaled to 512x512 and shimmering noise is added to make it more aurora-like. At this resolution, to look good, this would require about
32 raytracing steps per pixel, which is too much. So instead, the field is first radially blurred relative to where the
camera is (right above or below). This smears out the individual stacked layers exactly in the direction perspective would, and
gives a smooth result with only 6 raytracing steps.

The aurora has two primary colors at any given time, and the 'charm' value is used to interpolate between them (0..1)
and even extrapolate (outside that range) when the party really gets going.

When in space, the aurora is warped around the planet by applying a fudged formula of the form 1+r^2, approximating a paraboloid shell.

Earth
---

The earth shader involves a lot of twiddled numbers, but is basically just a diffuse lit sphere with atmospheric fog.
A ray-sphere intersection is used to measure the travel distance from the top of the atmosphere to the surface,
or to the far edge (for the outer atmosphere effect).

To increase detail, the texture only includes the northern hemisphere, and the cloud texture is tiled over it at a higher scale.


Effectively, it all looks like a bit crap everywhere the camera is not looking, which is why there is a lot of grain in the picture..


Credit
---

Moon by John French:
http://www.pa.msu.edu/people/frenchj/moon/index1.html

Earth at night by NASA:
http://visibleearth.nasa.gov/view.php?id=55167

Milkyway panorama by ESO:
http://en.wikipedia.org/wiki/File:ESO_-_The_Milky_Way_panorama_(by).jpg

Music is Cloudburn by Feed Me:
http://www.feedme.uk.com

* * *

Demo by Steven Wittens - http://acko.net/
