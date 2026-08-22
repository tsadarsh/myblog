---
title: 'Getting the simulation right'
description: 'Notes from tuning a simple inverted pendulum simulation in Mujoco and fixing jitter caused by micro-collisions.'
pubDate: 'Aug 21 2026'
heroImage: '../../assets/blog-placeholder-4.jpg'
---

I wanted to understand and learn PID control, so I designed a simple inverted pendulum setup in Onshape and simulated the controller in Mujoco. In this post, I want to note down the main takeaways for getting the simulation right. After exporting the model from Onshape and loading it in Mujoco, I noticed that the pendulum was jittery and the simulation was sluggish.

The first fix was to make the dimensions realistic. The model I designed in Onshape used millimeters, so when I created a pendulum of length 1, it was actually 1 mm rather than 1 meter. I scaled the model up, and the simulation immediately looked much more realistic. I also made sure the masses and materials were set correctly.

But the simulation was still jittery.
![jittery-sim](../../assets/jittery-sim.gif)

It took a few days to finally pinpoint the cause: micro-collisions. The hinge was constantly colliding with the pin holding the pendulum upright. To fix this, I set the `contype` and `conaffinity` parameters for each link correctly. I had seen these terms years earlier while working with Gazebo and URDFs, but I never took the time to understand what they were for.

It turns out they are a neat way to decide which objects collide with each other. Both `contype` and `conaffinity` are 32-bit masks that tell the simulator whether one geom should collide with another. The simulator decides this by computing: `(contype_A & conaffinity_B) OR (conaffinity_A & contype_B)`. If the result of this bit operation is greater than 0, then geomA and geomB can collide. If it is equal to 0, they behave as if they are ghosts to each other.

So in my simulation, I wanted the pendulum and the holder to not collide with each other. But both of them should be able to collide with the floor. So I set the following for the three collision bodies:

| Body | Contype | Conaffinity |
| ---  | ----   | ---- | 
| Floor | 1 | 1 | 
| Holder | 2 | 1 |
| Pendulum | 4 | 1 |

This way, the pendulum and holder do not collide with each other as:
> (2 & 1) OR (4 & 1) = 0

But the floor and holder collide (similarly, the floor and pendulum also collide):
> (1 & 1) OR (2 & 1) > 0

Here is the end result: no more jitter!
![fixed-no-jitter](../../assets/fixed-no-jitter.gif)
