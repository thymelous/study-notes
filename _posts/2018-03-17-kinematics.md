---
layout: post
title: "Kinematics"
date: 2018-03-17
mathjax: true
---
_(Originally published on [Tumblr](https://infinitejoke.tumblr.com/post/171960236016/notes-on-kinematics))_

Driven at this point less by a genuine interest in the topic than by the fear of failing my exams,
I set out to fill the glaring holes in my understanding of physics, starting with mechanics.

Memorizing precise formulas in high school left me with no actual idea of what's really going on.
Instead, I'd like to dig a bit further, and see the relationships between quantities.

The following notes were assembled from various web sources.

## Translational motion

The basic vector quantities are **displacement** $s$ `[m]`, referring to the object's overall change in position,
**velocity** $v$ `[m/s]`, the rate at which the position is changing,
and **acceleration** $a$ `[m/s^2]`, the rate at which the velocity is changing.

Let's start with acceleration, the first derivative of velocity with respect to time.
According to [this informative site](https://physics.info/kinematics-calculus/),
we can use the definition to go back to velocity:

$$a = \frac{dv}{dt} \implies dv = adt$$

$$\int_{v_0}^v dv = \int_0^t a dt$$

$$v \Big|_{v_0}^v = at \Big|_0^t$$

$$v - v_0 = at$$

$$v = v_0 + at$$

Velocity itself is a derivative, that of position with respect to time.
Which means we can use the same process to find the position-time equation:

$$v =\frac{ds}{dt} \implies ds = vdt = (v_0 + at)dt$$

$$\int_{s_0}^s ds = \int_0^t (v_0 + at)dt$$

$$s \Big|_{s_0}^s = v_0t + \frac{at^2}{2}\Big|_0^t$$

$$s - s_0 = v_0t + \frac{at^2}{2}$$

$$s = s_0 + v_0t + \frac{at^2}{2}$$

## Rotational motion

Angles are measured in _radians_. This demonstration from Wikipedia is quite informative:
<p style="text-align: center;">
<img src="https://upload.wikimedia.org/wikipedia/commons/4/4e/Circle_radians.gif">
</p>

Translational motion quantities have their rotational motion counterparts:
**angular displacement** $\theta$ `[rad]`,
**angular velocity** $\omega$ `[rad/s]`,
and **angular acceleraion** $\varepsilon$ `[rad/s^2]`.

Equations can be derived the same way:

$$\varepsilon = \frac{d\omega}{dt},\ \int_{\omega_0}^\omega d\omega = \int_0^t \varepsilon dt $$

$$\omega = \omega_0 + \varepsilon t$$

$$\omega = \frac{d\theta}{dt},\ \int_{\theta_0}^\theta d\theta = \int_0^t (\omega_0 + \varepsilon) dt$$

$$\theta = \theta_0 + \omega_0 t + \frac{\varepsilon t}{2}$$

While angular velocity (as well as angular displacement and angular acceleration) stay the same
for every point, _linear_ quantities -- **arc length/distance travelled** $s$, **tangential velocity** $v$,
and **tangential acceleration** $a_t$ -- do change depending on how far the point is from the axis.

$$s = R\Delta\theta,\ v = R\omega,\ a_t = R\varepsilon$$

Even if the object maintains a constant speed, with $a_t = 0$, the direction of the velocity is still changing
due to **centripetal acceleration** $a_c$. Its direction is toward the center, and its magnitude is

$$a_c = \frac{v^2}{R} = \frac{(R\omega)^2}{R} = R\omega^2$$

While tangential velocity is tangent to the turning object, angular velocity points in a direction _perpendicular_
to the object, determined by the Right-Hand Rule (illustration from Wikipedia):

<p style="text-align: center;">
<img width="100" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/34/Right-hand_grip_rule.svg/330px-Right-hand_grip_rule.svg.png">
</p>

Angular acceleration points in the same direction as angular velocity if the rotation is speeding up,
otherwise, the vector points in the opposite direction.
