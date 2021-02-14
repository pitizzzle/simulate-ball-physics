## goal
+ The same as in level 3, but using spring collisions.
  - Simulate a ball that bounces off all the walls if it touches them.
  - The ball is initially stationary, and gravity is constantly acting on it downwards.

<br>



## new idea
+ Instead of computing the results of individual collisions deterministically, we can also simulate an elastic deformation process when a ball hits a wall.
  <div align="center"><img src="img/ball-deformation.gif" alt="ball-deformation" width="400" /><br>contribution: <a href="https://www.youtube.com/watch?v=JT0wx27J9xs" target="_blank">SmarterEveryDay</a></div><br>

+ The deformation process is modeled by a spring that sits between the two colliding objects.
  - Its initial length is 0 and when the two objects bump deeper into each other, the spring is extended.
  <div align="center"><img src="img/level-5-collision-diagram-(1).gif" alt="level-5-collision-diagram-(1)" width="400" /></div>

  - The extended spring then pulls the objects out of each other, because it wants to return to its initial length 0, by applying a force on both objects that is proportional to the distance of extension, according to [Hooke's Law](https://en.wikipedia.org/wiki/Hooke%27s_law).
  $$
  F = s \cdot k
  $$
  - The force of this spring acts equally on both objects, but in opposite directions, pulling them out of each other (or rather pushing them away from each other).
  - Naturally, we don't want the wall to react to that force. Physically speaking, it has an infinite mass.
  - The spring force first slows down the ball's velocity down to zero. And because the ball still overlaps with the wall, the spring then accelerates the ball in the other direction and out of the wall.
  - This models an elastic deformation process, because just like in an actual deformation process, the deeper an object bumps into another object, the more it is deformed, storing the energy like a spring in the the internal material tension. The only difference is, that we don't keep track of the energy stored in the spring, or something like this. It is totally sufficient to simulate the force the spring applies to a ball, based on the depth of penetration.

<br>



## equations

$$
\text{--------- constants ---------}
$$

$$
\begin{aligned}
\text{\hspace{50pt}}
m &= \text{const.}\text{\small\color{gray}\hspace{14pt}(mass of the ball)}\\
g &= \text{const.}\text{\small\color{gray}\hspace{14pt}(gravity constant)}\\
r&=\text{const.}\text{\small\color{gray}\hspace{14pt}(radius of the ball)}\\
w&=\text{const.}\text{\small\color{gray}\hspace{14pt}(width of the scene)}\\
h&=\text{const.}\text{\small\color{gray}\hspace{14pt}(height of the scene)}\\[8pt]
\end{aligned}
$$

$$
\text{--------- constants ---------}
$$

$$
\begin{aligned}
\text{\hspace{50pt}}
w &= \text{const.}\text{\small\color{gray}\hspace{14pt}(width of the scene)}\\
h &= \text{const.}\text{\small\color{gray}\hspace{14pt}(height of the scene)}\\
k &= \text{const.}\text{\small\color{gray}\hspace{14pt}(spring stiffness)}\\[14pt]
\end{aligned}
$$







$$
\text{--------- ball-wall collisions ---------}
$$

<div align="center"><img src="img/level-5-collision-diagram-(2).svg" alt="level-4-collision-diagram-(2)" width="700" /></div>

$$
\text{\small\color{gray} (left and right wall)}
$$

$$
\begin{aligned}
F'_{x} =
    \begin{cases}
        + (r-x) \cdot k, & \text{if}\quad 0 \lt r-x\\
        - (x+r-w) \cdot k,\quad & \text{if}\quad 0 \lt x+r-w\\
        0, & \text{otherwise}
    \end{cases}\\[24pt]
\end{aligned}
$$

$$
\text{\small\color{gray} (top and bottom wall)}
$$

$$
\begin{aligned}
F'_{i,y} =
    \begin{cases}
        + (r-y) \cdot k, &\text{if}\quad 0 \lt r-y\\
        - (y+r-h) \cdot k,\quad & \text{if}\quad 0 \lt y+r-h\\
        0, & \text{otherwise}
    \end{cases}\\[24pt]
\end{aligned}
$$

$$
\text{--------- time step ---------}
$$

$$
\text{\small\color{gray} (new force, which is constant during the time step)}
$$

$$
\begin{aligned}
F''_x &= F'_x\\[8pt]
F''_y &= F'_y + m \cdot g\\[8pt]
\end{aligned}
$$

$$
\text{\small\color{gray} (new acceleration, which is constant during the time step)}
$$

$$
\begin{aligned}
a'_x &= \frac{F''_x}{m}\\[8pt]
a'_y &= \frac{F''_y}{m}\\[8pt]
\end{aligned}
$$

$$
\text{\small\color{gray} (new velocity after the time step)}
$$

$$
\begin{aligned}
v'_x &= v_x + dv_x  &  &\leftarrow  &  dv_x &= dt \cdot a'_x\\[4pt]
v'_y &= v_y + dv_y  &  &\leftarrow  &  dv_y &= dt \cdot a'_y\\[8pt]
\end{aligned}
$$

$$
\text{\small\color{gray} (new position, using the new velocity)}
$$

$$
\begin{aligned}
x' &= x + dx  &  &\leftarrow  &  dx &= dt \cdot v'_x\\[4pt]
y' &= y + dy  &  &\leftarrow  &  dy &= dt \cdot v'_y\\[14pt]
\end{aligned}
$$

<br>



## code
```js
const ball = {
    x: 0.5 * canvas.w,  // horizontally centered
    y: 0.2 * canvas.h,  // at the top
    v_x: 0,  // initially stationary
    v_y: 0,  //
    m: 1,
    r: 15,
};

const g = 0.5;  // gravity constant
const k = 0.1;  // spring stiffness

function simulateOneStep(dt) {

    ball.F_x = 0;           // reset/initialize
    ball.F_y = ball.m * g;  //

    if (ball.x - ball.r < 0) {
        ball.F_x -= (ball.x - ball.r) * k;
    }
    if (ball.x + ball.r - w > 0) {
        ball.F_x -= (ball.x + ball.r - w) * k;
    }
    if (ball.y - ball.r < 0) {
        ball.F_y -= (ball.y - ball.r) * k;
    }
    if (ball.y + ball.r - h > 0) {
        ball.F_y -= (ball.y + ball.r - h) * k;
    }

    const a_x = ball.F_x / ball.m;
    const a_y = ball.F_y / ball.m;
    ball.v_x += dt * a_x;
    ball.v_y += dt * a_y;
    ball.x += dt * ball.vx;
    ball.y += dt * ball.vy;
}
```

<br>



## discussion of the time step equations
+ In this level we have utilized the most simple time step equations, like initially in level 2 and level 3.
  - We compute a new acceleration, with that compute the velocity after the time step, and then compute the new position, assuming that the new velocity is already constantly present during the time step itself.
+ However, unlike in level 2 or level 3, where using the average velocity to update the position is beneficial or even required, we must not use it for spring collisions.
  - Reason 1 (minor): We can't argue anymore that averaging the velocity would slightly increase accuracy. This is because acceleration is not strictly constant anymore (in collisions the acceleration changes dynamically). And with dynamically changing acceleration, assuming the acceleration to be constant during a time step is already such a big approximation that it would be hilarious to say that avering the velocity would significantly increase accuracy.
  - Reason 2 (important): Spring collisions are really sensitive to the time step equations (similar to the ball jumping lower and lower in level 3). The ball would actually jump higher and higher if we used the average velocity to update the ball's position.
  <div align="center">[GIF of ball jumping too high and out of bounds]</div><br>

+ Explanation for Reason 2 and the influence of delayed velocity:
  - If we update the position of the ball with the averaged velocity, the ball is always updated with a slightly older velocity than if we only took the new velocity. It seems like this slightly older velocity results in the ball falling deeper into the ground than it should, resulting in larger accelerations on the ball, resulting in the fact that ball jumps up again with a higher velocity than it approached with.
  - If we stick to the time step equations from above, we don't have a delayed velocity, because we assume the new velocity after time step to be already constantly present during the time step itself.
+ This theory of delayed velocity can be evidenced by two experiments:
  - (1) Delaying the the velocity used to update the ball's position even further, and obverserving that the ball gains even more speed on each collision.
    * The theory would explain this like that: "Because of the delayed velocity used to update the ball's position, the ball falls deeper into the ground and receives more upward acceleration."
    $$
    EQUATIONS
    $$
    <div align="center">[GIF of ball jumping too high and out of bounds (SUPER-CHARGED)]</div>
  - (2) Anti-delaying the velocity used to update the ball's position, and observing that the ball actually loses speed with every collision.
    * The theory would explain this like that: "Because of the anti-delayed velocity used to update ball's position, the ball quickly responds to the collision force and already leaves the ground before having gained full speed again."
    $$
    EQUATIONS
    $$
    <div align="center">[GIF of ball jumping lower and lower (SUB-CHARGED)]</div>

<br>



## working example <small>(for all variants)</small>

||||
| --- | --- | --- |
| [Code]() | [Code Live]() | [Code Fiddle]() |