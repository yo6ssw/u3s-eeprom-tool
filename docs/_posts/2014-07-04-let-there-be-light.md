---
layout: post
title: And God said "Let there be light!"
tags:
    - demoscene
excerpt: "Congratulations! You accomplished the previous task and now you proudly look at them wires rotating around in style. But something's bothering you; they seem all a bit ... thin, shall we say? Your next mission is to get them dressed up in simple clothes and turn the light on. In common tongue this means that each cube should be flat shaded and have a distinct colour while performing the same mesmerizing dance as before.
And to add a bit of style, have the light modulate its energy from none to full twice in the same 5 second interval."
---
### Target

Congratulations! You accomplished the previous task and now you proudly look at them wires rotating around in style. But something's bothering you; they seem all a bit ... thin, shall we say? Your next mission is to get them dressed up in simple clothes and turn the light on. In common tongue this means that each cube should be flat shaded and have a distinct colour while performing the same mesmerizing dance as before.
And to add a bit of style, have the light modulate its energy from none to full twice in the same 5 second interval.

### Design

Ok, ok, so where do we start on this one? Let's see where we left off last time. We have a `CubeScene` that can `update()` and `render()`. For this challenge our main loop will remain as before:

{% highlight c++ %}
while (timer.elapsedSeconds() < 5) {
    scene.update(timer.lap());
    scene.render();
}
{% endhighlight %}

What is new however is that each cube now has its own color. A `Colour` is simply an RGB triplet, holding values in the range of [0...1].

{% highlight c++ %}
struct Colour {
    double r, g, b;
}
{% endhighlight %}

Since each cube has its own colour then we can simply add `Colour` as a `Mesh` property.

{% highlight c++ %}
class Mesh {
    ...
    Colour colour;
}
{% endhighlight %}

Next, in order to make shading have a meaning, we need a light source. We thus create a `Light` entity. But what properties does this `Light` have? We have identified two so far based on our requirements:

1. a colour
2. a position in space

This type of light is called a point light as it radiates light in every direction like a light bulb. We happen to know [there are other type of lights](http://en.wikipedia.org/wiki/Shading#Lighting) so we choose to rename the `Light` to `PointLight` to be more accurate in our naming.

{% highlight c++ %}
struct PointLight {
    Vertex position;
    Colour colour;
}
{% endhighlight %}

----

#### But what is this vertex though?

Time for some small refactoring. Something smells for quite a while. We use `Vertex` for both mesh position and mesh geometry, but what our `Vertex` contains is nothing more than a 3 dimensional _vector_. 

A 3 dimensional vector can be thought to contain a direction and a magnitude. Let's extract a `Vector3` class which we can use for both mesh and vertex positions. Also, the `Vertex` can contain many more attributes besides position, things like texture coordinate, colour, normal, so we'll refactor the `Vertex` to contain a `Vector3` _position_.

{% highlight c++ %}
struct Vector {
    double x, y, z;
}

struct Vertex {
    Vector3 position;
}
{% endhighlight %}

----

#### This rendering thing still does not look right...

So what we've got now is a `CubeScene` which contains some `Mesh`es and a `Light` and that should be able to update and render itself.
The update should obviously handle setting the appropriate state of the components in the scene which is composed by the cubes' orientation and light's intensity.

{% highlight c++ %}
void CubeScene::update(double secondsSinceLastUpdate) {
    for (auto& mesh : meshes)
        mesh.rotationAngle += 30 * secondsSinceLastUpdate;

    elapsedTimeInSeconds += secondsSinceLastUpdate;
    light.intensity = lightIntensityForCurrentTime(elapsedTimeInSeconds);
}
{% endhighlight %}

We've hidden the implementation details of the light intensity calculation in a meaningful named method whose job will be to make sure the intensity varies from 0 to 1 and back to 0 twice in 5 seconds.

How about rendering? In the previous objective implementation we simply taught meshes to draw themselves:

{% highlight c++ %}
void CubeScene::render() {
    ...
    for (auto& mesh : meshes) {
        glPushMatrix();
        mesh.applyTransformation();
        mesh.renderWireframe();
        glPopMatrix();
    }
}
{% endhighlight %}

I don't really like this anymore. The `Mesh` knows too much instead of being a dummy data holder. It now knows for instance how to draw itself in wireframe but now we need to also teach them to draw themselves in solid coloured polygons. 

Who should decide how a cube be rendered? Clearly not the humble mesh since its role should be that of a simple actor, dressed with certain clothing and told to pose and smile for the camera.

Ah, the clothing we mentioned above I feel is a good analogy for introducing a new component called `Material` whose reponsibility is to group all characteristics that define __how__ a `Mesh` should be rendered. Just as every person _owns_ some clothes, every `Mesh` refers to a `Material` which specifies how it should be rendered.

{% highlight c++ %}
class enum RenderType {
    Wireframe,
    Solid
}

struct Material {
    Colour colour;
    RenderType renderType;
}

struct Mesh {
    Material material;
}
{% endhighlight %}

The astute reader will notice that the `Colour` attribute has moved from its previous holder, the `Mesh` to a more meaningful parent, the `Material` and all in the scope of the current blog post. This is normal for evolutionary design which grows organicly with our needs.
