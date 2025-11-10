# Re-create the Jupyter notebook with corrected quoting

import nbformat, numpy as np, matplotlib.pyplot as plt

nb = nbformat.v4.new_notebook()
cells = []

intro_md = r"""
# Projectile Motion with Air Resistance (1st-Year Physics Simulation Project)

*Goal:* Build a simulation that compares ideal projectile motion (no drag) with motion under quadratic air resistance, explore how launch angle, speed, mass, and drag affect range and flight time, and present conclusions.

## Learning Outcomes
- Translate Newton’s 2nd law into coupled ODEs and solve them numerically.
- Compare analytical (no-drag) and numerical (drag) models.
- Generate and interpret trajectory plots and range–angle curves.
- Communicate results with clear figures and a short report.
"""
cells.append(nbformat.v4.new_markdown_cell(intro_md))

theory_md = r"""
## Theory

For a projectile of mass \(m\) launched from ground level with speed \(v_0\) and angle \(\theta\):

*No drag (analytic):*
\[
x(t)=v_0\cos\theta \, t,\quad
y(t)=v_0\sin\theta \, t-\tfrac{1}{2}gt^2.
\]
Time of flight \(T=\frac{2 v_0\sin\theta}{g}\). Range \(R=\frac{v_0^2\sin 2\theta}{g}\).

*With quadratic drag:*
Let \(\mathbf{v}=(v_x, v_y)\), \(v=\|\mathbf{v}\|\), gravitational acceleration \(g\) downward, and drag force
\[
\mathbf{F}_d = -\tfrac{1}{2}\rho C_d A\, v\, \mathbf{v} = -k\, v\, \mathbf{v},\quad k=\tfrac{1}{2}\rho C_d A.
\]
Equations of motion:
\[
\dot v_x = -\frac{k}{m} v v_x,\qquad
\dot v_y = -g -\frac{k}{m} v v_y.
\]

We will integrate these ODEs numerically (RK4 and Euler provided).
"""
cells.append(nbformat.v4.new_markdown_cell(theory_md))

setup_code = '''
import numpy as np
import matplotlib.pyplot as plt

g = 9.81  # m/s^2

def euler_projectile_drag(v0, theta_deg, m=0.145, k=0.05, dt=1e-3, y0=0.0):
    """
    Euler integration for projectile with quadratic drag.
    Parameters:
      v0         : launch speed (m/s)
      theta_deg  : launch angle in degrees
      m          : mass (kg)
      k          : drag parameter k = 0.5 * rho * Cd * A  (kg/m) 
      dt         : time step (s)
      y0         : initial height (m)
    Returns:
      t, x, y, vx, vy
    """
    theta = np.deg2rad(theta_deg)
    vx = v0*np.cos(theta)
    vy = v0*np.sin(theta)
    x = 0.0
    y = y0
    xs, ys, vxs, vys, ts = [x], [y], [vx], [vy], [0.0]
    t = 0.0
    while y >= 0.0 or t==0.0:
        v = np.hypot(vx, vy)
        ax = -(k/m) * v * vx
        ay = -g - (k/m) * v * vy
        # Euler update
        vx += ax*dt
        vy += ay*dt
        x += vx*dt
        y += vy*dt
        t += dt
        xs.append(x); ys.append(y); vxs.append(vx); vys.append(vy); ts.append(t)
        # Safety to avoid infinite loop
        if t > 60.0:
            break
    # Linear interpolate last point to ground (y=0)
    if ys[-1] < 0 and ys[-2] >= 0:
        # interpolate between last two points to find ground hit
        frac = ys[-2] / (ys[-2] - ys[-1])
        xg = xs[-2] + frac*(xs[-1]-xs[-2])
        tg = ts[-2] + frac*(ts[-1]-ts[-2])
        xs[-1] = xg; ys[-1] = 0.0; ts[-1] = tg
    return np.array(ts), np.array(xs), np.array(ys), np.array(vxs), np.array(vys)

def rk4_step(vx, vy, m, k, dt):
    def accel(vx, vy):
        v = np.hypot(vx, vy)
        ax = -(k/m) * v * vx
        ay = -g - (k/m) * v * vy
        return ax, ay

    k1x, k1y = accel(vx, vy)
    k2x, k2y = accel(vx + 0.5*dt*k1x, vy + 0.5*dt*k1y)
    k3x, k3y = accel(vx + 0.5*dt*k2x, vy + 0.5*dt*k2y)
    k4x, k4y = accel(vx + dt*k3x, vy + dt*k3y)

    vx_new = vx + (dt/6.0)*(k1x + 2*k2x + 2*k3x + k4x)
    vy_new = vy + (dt/6.0)*(k1y + 2*k2y + 2*k3y + k4y)
    return vx_new, vy_new

def rk4_projectile_drag(v0, theta_deg, m=0.145, k=0.05, dt=1e-3, y0=0.0):
    """
    RK4 integration for projectile with quadratic drag.
    Returns: t, x, y, vx, vy
    """
    theta = np.deg2rad(theta_deg)
    vx = v0*np.cos(theta)
    vy = v0*np.sin(theta)
    x = 0.0
    y = y0
    xs, ys, vxs, vys, ts = [x], [y], [vx], [vy], [0.0]
    t = 0.0
    while y >= 0.0 or t==0.0:
        vx, vy = rk4_step(vx, vy, m, k, dt)
        x += vx*dt
        y += vy*dt
        t += dt
        xs.append(x); ys.append(y); vxs.append(vx); vys.append(vy); ts.append(t)
        if t > 60.0:
            break
    # interpolate to ground
    if ys[-1] < 0 and ys[-2] >= 0:
        frac = ys[-2] / (ys[-2] - ys[-1])
        xg = xs[-2] + frac*(xs[-1]-xs[-2])
        tg = ts[-2] + frac*(ts[-1]-ts[-2])
        xs[-1] = xg; ys[-1] = 0.0; ts[-1] = tg
    return np.array(ts), np.array(xs), np.array(ys), np.array(vxs), np.array(vys)

def ideal_projectile(v0, theta_deg, dt=1e-3, y0=0.0):
    theta = np.deg2rad(theta_deg)
    vx = v0*np.cos(theta)
    vy0 = v0*np.sin(theta)
    if y0 > 0:
        t_flight = (vy0 + np.sqrt(vy0**2 + 2*9.81*y0)) / 9.81 * 2
    else:
        t_flight = max(0.0, 2*vy0/9.81)
    t = np.arange(0, max(t_flight, dt)+dt, dt)
    x = vx * t
    y = y0 + vy0*t - 0.5*9.81*t**2
    y = np.maximum(y, 0.0)
    return t, x, y

def simulate_and_plot(v0=40.0, theta_deg=35.0, m=0.145, k=0.05, dt=1e-3, method='rk4', y0=0.0):
    if method.lower()=='euler':
        t, x, y, vx, vy = euler_projectile_drag(v0, theta_deg, m, k, dt, y0)
    else:
        t, x, y, vx, vy = rk4_projectile_drag(v0, theta_deg, m, k, dt, y0)
    ti, xi, yi = ideal_projectile(v0, theta_deg, dt, y0)

    # Trajectory
    plt.figure()
    plt.plot(xi, yi, label='No drag (analytic)')
    plt.plot(x, y, label=f'Drag (k={k:.3f}, {method.upper()})')
    plt.xlabel('x (m)')
    plt.ylabel('y (m)')
    plt.title('Projectile Trajectory')
    plt.legend()
    plt.grid(True)
    plt.show()

    # Range–angle sweep
    angles = np.linspace(10, 80, 71)
    ranges_drag = []
    ranges_nodrag = []
    for ang in angles:
        _, x_d_t, y_d_t, *_ = rk4_projectile_drag(v0, ang, m=m, k=k, dt=dt, y0=y0)
        ranges_drag.append(x_d_t[-1])
        ti2, xi2, yi2 = ideal_projectile(v0, ang, dt, y0)
        ranges_nodrag.append(xi2[-1])
    angles = np.array(angles)
    ranges_drag = np.array(ranges_drag)
    ranges_nodrag = np.array(ranges_nodrag)

    plt.figure()
    plt.plot(angles, ranges_nodrag, label='No drag (analytic)')
    plt.plot(angles, ranges_drag, label='With drag (RK4)')
    plt.xlabel('Launch angle (deg)')
    plt.ylabel('Range (m)')
    plt.title('Range vs. Launch Angle')
    plt.legend()
    plt.grid(True)
    plt.show()

    return {'trajectory_range_drag': float(ranges_drag.max()), 'angle_for_max_drag': float(angles[ranges_drag.argmax()])}

print("Loaded helpers. Try: simulate_and_plot(v0=40, theta_deg=35, m=0.145, k=0.05, dt=0.002, method='rk4')")
'''
cells.append(nbformat.v4.new_code_cell(setup_code))

demo_md = r"""
## Quick Demo
Run the next cell to generate two figures:
1. Trajectory comparison (no drag vs drag)
2. Range–angle curves (no drag vs drag)
"""
cells.append(nbformat.v4.new_markdown_cell(demo_md))

demo_code = r"""
result = simulate_and_plot(v0=40.0, theta_deg=35.0, m=0.2, k=0.06, dt=0.002, method='rk4', y0=0.0)
print("Max range with drag (m):", result['trajectory_range_drag'])
print("Angle for max range with drag (deg):", result['angle_for_max_drag'])
"""
cells.append(nbformat.v4.new_code_cell(demo_code))

tasks_md = r"""
## What You Need to Submit
1. *Code* (this notebook or a clean .py file) with clear comments.
2. *3–4 figures*:
   - Trajectory comparison for at least *three* angles.
   - Range–angle plot (10–80°).
   - A plot showing how range varies with *mass* \(m\) or *drag parameter* \(k\).
3. *Short report* (2–3 pages): objectives, method, parameters used, results, and 5–6 bullet conclusions.

## Suggested Experiments
- Vary \(k\) (e.g., mimic different balls) and show its effect on optimal launch angle.
- Vary mass \(m\) at fixed geometry (note: with quadratic drag, heavier projectiles go farther).
- Include a small *tailwind/headwind* by modifying the relative velocity \(\mathbf{v}-\mathbf{u}\).
- Start from a non-zero height \(y_0\) (e.g., table or cliff) and discuss.

## Rubric (20 marks)
- Physics formulation (4)
- Code correctness & comments (4)
- Figures quality & labels (4)
- Analysis & discussion (6)
- Presentation & citations (2)
"""
cells.append(nbformat.v4.new_markdown_cell(tasks_md))

readme_md = r"""
## Tips
- Keep time step small enough for stability; prefer RK4 for accuracy.
- Check your code by recovering the no-drag formulas when \(k\to 0\).
- Label axes with units and include legends.
- Cite any sources used for parameters (e.g., \(C_d\), \(\rho\), \(A\)).
"""
cells.append(nbformat.v4.new_markdown_cell(readme_md))

nb['cells'] = cells

path = "/mnt/data/Projectile_Simulation_Project.ipynb"
import os, json
with open(path, "w", encoding="utf-8") as f:
    nbformat.write(nb, f)

path
