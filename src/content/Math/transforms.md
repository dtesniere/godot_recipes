---
title: "Transforms"
date: 2019-04-09T19:49:14-07:00
weight: 2
draft: false
---

Before reading this, make sure you have an understanding of vectors and how
they're used in game development. If you don't, I recommend you read this
introduction I wrote for the Godot documentation:
[Vector Math](https://docs.godotengine.org/en/latest/tutorials/math/vector_math.html).

## 2D Transforms

In 2D space, we use the familiar X-Y coordinate plane. Remember that in
Godot, as in most computer graphics applications, the **Y** axis points downward:

![alt](/godot_lessons/img/0_2d_coordinate_plane.png?width=250px)

To begin, let's consider this spaceship floating in space:

![alt](/godot_lessons/img/0_2d_rocket1.png?width=250px)

Currently, the ship is pointing in the same direction as the **X** axis. If we
wanted it to move forward, we could just add to its **X** coordinate and it
would move to the left: `position += Vector2(10, 0)`.

But what happens when the ship is rotated?

![alt](/godot_lessons/img/0_2d_rocket2.png?width=250px)

How do we move the ship forward now? If you remember Trigonometry from school,
you might be starting to think about angles, sin and cos and doing something
like `position += Vector2(10 * cos(angle), 10 * sin(angle))`. While this would
work, there's a much more convenient way: the _Transform_.

Let's look at the rotated ship again, but this time, let's also imagine that
the ship has its own **X** and **Y** axes that it carries with it, completely
independent of the global axes:

![alt](/godot_lessons/img/0_2d_rocket3.png?width=250px)

These "local" axes are a part of the transform called the `basis`.

Knowing this, we can move the ship forward by moving it along its own **X** axis and
we won't have to worry about angles and trig functions. To do this in Godot, we can use
the `transform` property, which is available to all [Node2D]([https://link](https://docs.godotengine.org/en/latest/classes/class_node2d.html)) derived nodes.

```gdscript
    position += transform.x * 10
```

This code says "Add the transform's x vector multiplied by 10." Let's break down
what that means. The `transform` contains `x` and `y` properties that represent
those local axes. They are _unit vectors_, which means their length is `1`.
Another term for unit vector is _direction vector_. They tell us the direction
the ship's **x** axis is pointing. We then multiply by `10` to scale it to a
longer distance.

{{% notice tip %}}
Just like `position` and `rotation`, which are included in the
transform, the `transform` property of a node is _relative_ to its parent
node. If you need to get the global value, it's available in
`global_transform`.
{{% /notice %}}

In addition to the `basis`, the transform also contains a component called the
`origin`. Just as the basis represents the body's rotation, the origin represents
the _translation_, or change in position.

In this picture, the blue vector is the `transform.origin`. It is equivalent
to the object's `position` vector.

![alt](/godot_lessons/img/0_2d_rocket4.png?width=250px)

### Operations on Transforms

#### Rotation

A given transform can be rotated with the `rotated()` method:

**IMAGE: Rotated transform (side-by-side)**

#### Translation

To move a transform, i.e. change its `origin` using global coordinates:

```gdscript
transform.origin += Vector2(2, 0)
```

Using local coordinates:

```gdscript
transform = transform.translated(Vector2(2, 0))
```

#### Scaling

If the _length_ of the basis vectors is not `1` (ie normalized), then the
the transform also encodes scale.

### Converting Between Local and Global Space

You can convert coordinates from global to local by applying
the transform. To apply a transform, use `xform()`:

**SNIPPET**

To convert from global to local coordinates, you can use the opposite (inverse)
of the transform with `xform_inv()`.

Let's use the example of an object in the 2D plane and convert mouse clicks
(global space) into coordinates relative to the object:

**IMAGE: ship w/grid**

```gdscript
extends Sprite

func _unhandled_input(event):
    if event is InputEventMouseButton and event.pressed:
        if event.button_index == BUTTON_LEFT:
            printt(event.position, transform.xform_inv(event.position))
```

For convenience, Node2D includes helper functions for this. Instead of using
`transform.xform_inv()`, you can call `to_local()`:

```gdscript
    printt(event.position, to_local(event.position))
```

See the [Transform2D docs](https://docs.godotengine.org/en/latest/classes/class_transform2d.html)
for all of the available properties and methods.

## 3D Transforms

In 3D space, the concept of transforms applies in the exact same way as in 2D.
In fact, it becomes even more necessary, as using angles in 3D can lead to
a variety of problems, as we'll see in a bit.

3D nodes inherit from the base node [Spatial]([https://link](https://docs.godotengine.org/en/latest/classes/class_spatial.html)),
which contains thevtransform information. However, the 3D transform requires more
information than the 2D version. Position is still held in the `origin` property,
but rotation is found in a property called `basis`, which contains three unit
vectors representing the body's local **X**, **Y**, and **Z** axes.

![alt](/godot_lessons/img/3d_intro_gizmo.png)

{{% notice note %}}
In the editor, you can see and manipulate the body's local orientation by
clicking the "Local Space Mode" button.
![alt](/godot_lessons/img/3d_intro_local_space.png)
{{% /notice %}}

Just as we did in 2D, we can use the local axes to move an object forward. In
Godot's 3D orientation (**Y-up**), this means that by default the body's **-Z**
axis is the forward direction:

```gdscript
    translation += -transform.basis.z
```

> **NOTE:** Godot has default vector values defined, for example: `Vector3.FORWARD = Vector3(0, 0, -1)`.