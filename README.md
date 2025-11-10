import math
import argparse
import os
from dataclasses import dataclass
from typing import Tuple, List
import numpy as np
import matplotlib.pyplot as plt

# ------------------- Constants -------------------
g = 9.81  # m/s^2

# ------------------- Data structure -------------------
@dataclass
class Trajectory:
    t: np.ndarray
    x: np.ndarray
    y: np.ndarray
    vx: np.ndarray | None = None
    vy: np.ndarray | None = None

# ------------------- Core Physics Functions -------------------
def euler_projectile_drag(v0, theta_deg, m=0.145, k=0.05, dt=1e-3, y0=0.0):
    """Euler method for projectile with quadratic drag."""
    theta = math.radians(theta_deg)
    vx, vy = v0 * math.cos(theta), v0 * math.sin(theta)
    x, y = 0.0, y0
    xs, ys, vxs, vys, ts = [x], [y], [vx], [vy], [0.0]
    t = 0.0
    while y >= 0.0 or t == 0.0:
        v = math.hypot(vx, vy)
        ax = -(k/m) * v * vx
        ay = -g - (k/m) * v * vy
        vx += ax * dt
        vy += ay * dt
        x += vx * dt
        y += vy * dt
        t += dt
        xs.append(x); ys.append(y); ts.append(t)
        if t > 60.0:
            break
    if ys[-1] < 0 and ys[-2] >= 0:
        frac = ys[-2] / (ys[-2] - ys[-1])
        xs[-1] = xs[-2] + frac*(xs[-1]-xs[-2])
        ys[-1] = 0.0
        ts[-1] = ts[-2] + frac*(ts[-1]-ts[-2])
    return Trajectory(np.array(ts), np.array(xs), np.array(ys))

def rk4_step(vx, vy, m, k, dt):
    def accel(vx, vy):
        v = math.hypot(vx, vy)
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
    """RK4 method for projectile with quadratic drag."""
    theta = math.radians(theta_deg)
    vx, vy = v0 * math.cos(theta), v0 * math.sin(theta)
    x, y = 0.0, y0
    xs, ys, ts = [x], [y], [0.0]
    t = 0.0
    while y >= 0.0 or t == 0.0:
        vx, vy = rk4_step(vx, vy, m, k, dt)
        x += vx * dt
        y += vy * dt
        t += dt
        xs.append(x); ys.append(y); ts.append(t)
        if t > 60.0:
            break
    if ys[-1] < 0 and ys[-2] >= 0:
        frac = ys[-2] / (ys[-2] - ys[-1])
        xs[-1] = xs[-2] + frac*(xs[-1]-xs[-2])
        ys[-1] = 0.0
        ts[-1] = ts[-2] + frac*(ts[-1]-ts[-2])
    return Trajectory(np.array(ts), np.array(xs), np.array(ys))

def ideal_projectile(v0, theta_deg, dt=1e-3, y0=0.0):
    """Analytical no-drag trajectory."""
    theta = math.radians(theta_deg)
    vx, vy0 = v0 * math.cos(theta), v0 * math.sin(theta)
    t_flight = (vy0 + math.sqrt(max(vy0**2 + 2*g*y0, 0))) / g * 2 if y0 > 0 else 2 * vy0 / g
    t = np.arange(0, t_flight + dt, dt)
    x = vx * t
    y = y0 + vy0*t - 0.5*g*t**2
    y = np.maximum(y, 0)
    return Trajectory(t, x, y)

# ------------------- Plotting -------------------
def plot_trajectory(no_drag, drag, path):
    plt.figure()
    plt.plot(no_drag.x, no_drag.y, label="No drag (analytic)")
    plt.plot(drag.x, drag.y, label="With drag (RK4)")
    plt.xlabel("x (m)"); plt.ylabel("y (m)")
    plt.title("Projectile Trajectory")
    plt.legend(); plt.grid(True)
    plt.savefig(path, bbox_inches="tight"); plt.close()

def sweep_angle(v0, m, k, dt, y0=0.0):
    angles = np.linspace(10, 80, 71)
    range_no, range_drag = [], []
    for a in angles:
        nd = ideal_projectile(v0, a, dt, y0)
        dr = rk4_projectile_drag(v0, a, m, k, dt, y0)
        range_no.append(nd.x[-1])
        range_drag.append(dr.x[-1])
    return angles, np.array(range_no), np.array(range_drag)

def plot_range_angle(angles, range_no, range_drag, path):
    plt.figure()
    plt.plot(angles, range_no, label="No drag (analytic)")
    plt.plot(angles, range_drag, label="With drag (RK4)")
    plt.xlabel("Angle (deg)"); plt.ylabel("Range (m)")
    plt.title("Range vs Launch Angle")
    plt.legend(); plt.grid(True)
    plt.savefig(path, bbox_inches="tight"); plt.close()

# ------------------- Demo -------------------
def run_demo(v0=40.0, theta=35.0, m=0.2, k=0.06, dt=0.002, y0=0.0, outdir="out"):
    os.makedirs(outdir, exist_ok=True)
    no_drag = ideal_projectile(v0, theta, dt, y0)
    drag = rk4_projectile_drag(v0, theta, m, k, dt, y0)
    plot_trajectory(no_drag, drag, f"{outdir}/trajectory.png")
    angles, range_no, range_drag = sweep_angle(v0, m, k, dt, y0)
    plot_range_angle(angles, range_no, range_drag, f"{outdir}/range_angle.png")
    print(f"Saved plots to {outdir}/")

# ------------------- CLI -------------------
def main():
    parser = argparse.ArgumentParser(description="Projectile motion with quadratic drag.")
    parser.add_argument("--mode", default="demo", choices=["demo", "sweep-angle", "traj"])
    parser.add_argument("--v0", type=float, default=40.0)
    parser.add_argument("--theta", type=float, default=35.0)
    parser.add_argument("--m", type=float, default=0.2)
    parser.add_argument("--k", type=float, default=0.06)
    parser.add_argument("--dt", type=float, default=0.002)
    parser.add_argument("--y0", type=float, default=0.0)
    parser.add_argument("--outdir", type=str, default="out")
    args = parser.parse_args()
    os.makedirs(args.outdir, exist_ok=True)

    if args.mode == "demo":
        run_demo(args.v0, args.theta, args.m, args.k, args.dt, args.y0, args.outdir)
    elif args.mode == "traj":
        nd = ideal_projectile(args.v0, args.theta, args.dt, args.y0)
        dr = rk4_projectile_drag(args.v0, args.theta, args.m, args.k, args.dt, args.y0)
        plot_trajectory(nd, dr, f"{args.outdir}/trajectory.png")
    elif args.mode == "sweep-angle":
        angles, range_no, range_drag = sweep_angle(args.v0, args.m, args.k, args.dt, args.y0)
        plot_range_angle(angles, range_no, range_drag, f"{args.outdir}/range_angle.png")

if _name_ == "_main_":
    main()
