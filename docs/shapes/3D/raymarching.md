---
layout: page
title:  "raymarching"
date:   2022-11-22 15:08:55 +0100
categories: jekyll update
parent: "3D"
grand_parent: "Shapes"
nav_order: 1
child_nav_order: asc
has_children: true
---
<!-- Mathjax Support -->
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# Ray Marching / Sphere Marching

## Ray marching
### Definition
Ray marching is a rendering method, where a ray is cast from a point of origin in a direction, and is subdivided into smaller segments until an intersection with a surface is detected. 
The segmentation is often done using spheres, hence the other name for ray marching: sphere marching. It can also be achieved using cubes, but that won't be covered here.
The surface that the ray intersects with is defined by [signed distance functions](https://en.wikipedia.org/wiki/Signed_distance_function).

---
### How it works
Ray marching is very litteraly named. We are trying to move along a ray to find a surface. The movement along the ray is done in steps. For each step we check the distance from our current position to the closest surface, add that distance to the total distance traveled along the ray, and update our current position for the next step.


In the following stunning GIF, the steps are represented by the orange circles, the current position is the red circles, and the green line is the distance to the closest surface.

![A beautiful GIF should show up](./illustrations/ray_marching_steps.gif)

 We repeat that process until one of three things happen:

1. We exceed the set maximum number of steps.

2. The distance we measure is smaller than the minimum distance to a surface. In other words: we are close enough.

3. The distance traveled along the ray exceeds the set maximum distance.

Once one of those conditions is met, we return the distance traveled along the ray, which corresponds to the distance from the camera to the surface. Then we move on to another ray, and repeat the whole process.

---

### Implementation
1. Casting Rays

The first thing we have to do is to cast rays that we can march along. This is the equivalent of a camera, it is placed in the world and points at something, or in other terms is has a position and a direction.
```glsl
    vec3 ray_origin = vec3(0,1,0);
    vec3 ray_direction = normalize(vec3(st.s, st.t, 1.));
```
The _ray_origin_ vector is quite simple, it's at the center of the world, slightly elevated above the scene.
The _ray_direction_ vector is a normalised, or unit vector that defines where the camera points. The first two parameters, _st.s_ and _st.t_ are the canvas coordinates, which means that we cast a ray for each fragment. The last parameter gives a "forward" direction in the z axis. And we use the normalize function to get a vector with the same direction but of length 1. See [Math](..\..\math\math.md) for more info on normalised vectors.

And with these two variables we now have our camera, or rays.

2. Marching along the rays

The next step is to define the ray marching function. 

First we define the three variables we'll use as break conditions in our ray marching loop like we mentioned above.
-  MAX_STEPS, the maximum number of steps to take along the ray. It will greatly determine how precise the ray marching will be. Can give cool morphing effects when set low.
- MAX_DIST the maximum distance we want to travel along our ray.
- SURFACE_DIST, the minimum value for the distance between the current position and the closest surface. In other words: it's precision. The smaller the number the closer to the surface our ray marching will go. Raymarching is not exact, it's an estimate of where a surface is, but we can decide to make it more or less approximate.

``` c
#define MAX_STEPS 100
#define MAX_DIST 100.
#define SURFACE_DIST 0.01
```

The we define the ray marching function, that we creatively call "raymarch".

``` glsl
float raymarch(vec3 ro, vec3 rd){
    float distance_traveled = 0.;
    for(int i = 0; i<MAX_STEPS; i++){
        vec3 current_pos = ro +distance_traveled*rd;
        float distance_closest = sdfSphere(current_pos);
        distance_traveled += distance_closest;
        if(distance_closest<SURFACE_DIST || distance_traveled>MAX_DIST){break;}
    }

    return distance_traveled;
}
```

Let's break this function down and start by the parameters.
The raymarch function takes in the two ray variables we created earlier: the ray's origin _ro_ and the ray's direction _rd_.

We set the total distance traveled along the ray to 0, because we have just begun marching.

Then we make a for loop that will run for the maximum number of steps. Inside this loop, we have our step code.
~~~ glsl
    for(int i = 0; i<MAX_STEPS; i++){
        ...
    }
~~~
 First we set our current position to be how far we've traveled along the ray so far. So for our first step, where we haven't marched along a ray yet, that would be our camera's position _ro_. 
```glsl
vec3 current_pos = ro +distance_traveled*rd;
```

Then we get the distance from our position to the closest surface by calling sdfSphere(), which will be explained in the __Signed Distance Functions__ chapter later. The distance is added to the total distance traveled, which will in turn change the current position of the next step.

```glsl
float distance_closest = sdfSphere(current_pos);
distance_traveled += distance_closest;
```
And lastly, we check for break conditions that we talked about earlier.
```glsl
    if(distance_closest<SURFACE_DIST || distance_traveled>MAX_DIST){break;}
    }
```
---
## Signed Distance Functions
In the ray marching implementation we saw a function called sdfSphere(), _sdf_ stands for "signed distance function", and without it we wouldn't have anything to display when we ray march.

Because of course the point of ray marching is to detect surfaces in our scene, so if we have no surfaces, we have nothing to detect. And we're not loading any 3D meshes or anything fancy like this, so we have to get our surfaces from somewhere else.

This is where _signed distance functions_ come in. There are many of them, and they are used to draw basic, and sometimes not so basic, shapes using raymarching. These functions take in our current position, and various other parameters (like a radius for a sphere), they apply a shape specific formula and give us the distance from our point to the shape surface. I know it seems really abstract said like this, but that's basically it.


## Basic shapes
Inogo Quilez has a a lot of them listed on his [website](https://iquilezles.org/articles/distfunctions/). However he doesn't give explanations for them, so below are explanations on how to come up with some of these functions. A lot of these I understood from the wonderful [The Art of Code](https://www.youtube.com/@TheArtofCodeIsCool/featured) channel who gives theoritical explanations of these functions and then implements them in ShaderToy. I highly recommend his videos.

I'll be repeating a lot of what The Art of Code covers, but I'll try to break down the math a bit more, for the math impaired like me. You can never have too many diagrams.

### Sphere
It is the simplest!

The function looks like this:

~~~glsl
float sdfSphere(vec3 p, vec3 sp, float r){
    return length(p-spherePos)-r;
}
~~~

It takes in the a vector3 _p_, which is our current position along the ray we're marching, a vector3 _sp_, which is the position of the sphere in our scene, and a float _r_, the radius.

In order to return the distance to the surface of the sphere, we can start by mesuring the distance between our position and the sphere position. To get the distance to the surface, we simply substract the radius from that distance.


<div style="vertical-align:middle; text-align:center">
    <img src="./illustrations/SDF_sphere.png"/>
</div>

For an explanation on the length function, see [here](..\..\math\math.md).
### Capsule
Next are capsules, which are essentially two spheres joined together by a vector going from the center of one sphere to the center of the other.

The idea is to get a scalar projection _s_ of the vector _ap_ onto the vector _ab_, clamp that scalar between 0 and 1, and use it to move from _a_ in the direction of _ab_ by the scalar amount, to get the point _c_ on _ab_. From there, we can measure the distance _d_ from _p_ to _c_ and substract the radius _r_.
1. Function Signature

First let's set the function to take in our current position _p_, two vec3 that will be our spheres, and a float _r_ that will be our radius
~~~glsl
float sdfCapsule(vec3 p, vec3 a, vec3 b, float r)
~~~

2. Getting the scalar projection

The scalar projection is basically the length of a vector projected onto another. So on the diagram below we're trying to get the length of the red arrow. Another way to think of this is that the scalar projection is the size of a projected vector on a scale of _ab_. 

<div style="vertical-align:center; text-align:center">
<img src="./illustrations/SDF_Capsule_00.png" width = "350" />
</div>

To do that we'll use a scalar projection of the vector _ap_ onto the vector _ab_.

To get the scalar projection of a vector onto another, the formula is:  $$\frac{\vec{ap} \cdot \vec{ab}}{\left \| \vec{ab} \right \|}$$

which gives us:

~~~glsl
float s = dot(ap, ab)/length(ab);
~~~
This will give us a float value that will be between 0 and 1 when _p_ is "above" _ab_, 0 being directly above _a_ and 1 being directly above _b_.

 <div style="vertical-align:center; text-align:center">
<img src="./illustrations/SDF_Capsule_00_scale.png" width = "350"/>
</div>
However if we take into consideration the case where _p_ is not above the capsule, our scalar projection will either go negative or go past the length of the vector _ab_, like it's the case on the diagram below.

<div style="vertical-align:center; text-align:center">
<img src="./illustrations/SDF_Capsule_00_negative.png" width = "350"/>
<img src="./illustrations/SDF_Capsule_00_beyond_ab.png" width = "350"/>
</div>

And since our goal is to get a point on the vector _ab_ we need to make sure that our scalar is limited to the length of that vector.

We can easily do this by clamping our scalar _s_ between 0 and 1.

~~~glsl
s = clamp(s, 0., 1.);
~~~


3. Getting a vec3

 Since our goal is to get a point on the vector _ab_, and our scalar is a float, we need to find a way use that float to get the vector _c_ in pink below.

<div style="vertical-align:center; text-align:center">
    <img src="./illustrations/SDF_Capsule.png" width = "350"/>
</div>

To do that we can use a similar logic that we used in the ray marching function.
We march from point _a_, in the direction of _ab_ by the amount of _s_.

~~~glsl
vec3 c = a + s * ab;
~~~
Easy.

4. Getting a distance

Now that we have a point, we can get the distance from our current position to that point. And to get the distance from our current position to the surface of the capsule, we can simply substract the radius.

~~~glsl
float d = length(p-c)-r;
~~~

and that's it, we have a signed distance function to draw capsules. Right now our function can only work with spheres that have the same radius, so let's bare that limitation in mind.

### Box

### Torus
---
# Credits
- [Ray Marching for Dummies](https://www.youtube.com/watch?v=PGtv-dBi2wE) by [The Art of Code](https://www.youtube.com/c/TheArtofCodeIsCool): explains the basic concepts simply and implements them in ShaderToy.
- [Ray Marching Primitives](https://www.shadertoy.com/view/wdf3zl) by [The Art of Code](https://www.youtube.com/c/TheArtofCodeIsCool): Explains the signed distance functions for some basic shapes, and does an implementation of them in ShaderToy.
- [Distance functions](https://iquilezles.org/articles/distfunctions/) by Inigo Quilez: A very useful reference that lists signed distance functions for basic primitive shapes.
- [Ray marching scene exemple](https://www.shadertoy.com/view/Xds3zN) by Inigo Quilez: A useful and complete exemple on how to use the distance functions, ray marching operators, camera transformations.