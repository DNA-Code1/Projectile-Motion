<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Projectile Motion with Air Resistance</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f0f4f8;
      color: #333;
      padding: 20px;
      text-align: center;
    }
    h1 { margin-bottom: 10px; color: #2c3e50; }
    p { margin-bottom: 20px; color: #555; }
    #canvas-container {
      width: 800px; height: 400px; margin: 20px auto;
      border: 2px solid #3498db; border-radius: 10px;
      background: #ecf0f1; box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }
    .controls {
      max-width: 600px; margin: 20px auto; padding: 15px;
      background: white; border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      display: flex; flex-direction: column; gap: 12px;
    }
    .control-group {
      display: flex; justify-content: space-between; align-items: center;
      flex-wrap: wrap; gap: 10px;
    }
    label { font-weight: 600; color: #2c3e50; }
    input[type="range"] { flex: 1; min-width: 200px; }
    button {
      padding: 10px 20px; font-size: 16px; font-weight: bold;
      border: none; border-radius: 6px; cursor: pointer;
      transition: 0.2s;
    }
    #launch-btn { background: #27ae60; color: white; }
    #reset-btn { background: #e74c3c; color: white; }
    button:hover { transform: translateY(-2px); }
    .graph-container {
      margin: 30px auto; width: 600px; padding: 15px;
      background: white; border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    #graph-canvas {
      width: 100%; height: 300px; border: 1px solid #ddd;
      border-radius: 6px; background: #fdfdfd;
    }
  </style>
</head>
<body>
  <h1>Projectile Motion Simulator</h1>
  <p>Launch a cannonball and compare ideal vs real motion with air resistance.</p>

  <div id="canvas-container"></div>

  <div class="controls">
    <div class="control-group">
      <label>Speed (m/s): <span id="speed-val">30</span></label>
      <input type="range" id="speed" min="10" max="60" value="30" step="1"/>
    </div>
    <div class="control-group">
      <label>Angle (°): <span id="angle-val">45</span></label>
      <input type="range" id="angle" min="5" max="85" value="45" step="1"/>
    </div>
    <div class="control-group">
      <label>Air Resistance: 
        <input type="checkbox" id="drag-toggle" checked/>
        <span id="drag-status">ON</span>
      </label>
    </div>
    <button id="launch-btn">Launch!</button>
    <button id="reset-btn">Reset</button>
  </div>

  <div class="graph-container">
    <h3>Range vs Angle</h3>
    <canvas id="graph-canvas"></canvas>
  </div>

  <script>
    let balls = [], trails = [];
    let graphData = { ideal: [], real: [] };
    let launched = false;
    const g = 9.81, dt = 0.016, scale = 10, dragCoeff = 0.001;

    function setup() {
      let canvas = createCanvas(800, 400);
      canvas.parent('canvas-container');
      resetSimulation();
      setupUI();
    }

    function draw() {
      background(240, 248, 255);
      drawGround();
      drawCannon();
      updateBalls();
      if (launched) { launchBall(); launched = false; }
      if (balls.length === 2 && !balls[0].active && !balls[1].active && balls[0].landed) updateGraph();
    }

    function drawGround() {
      fill(34, 139, 34); rect(0, height - 50, width, 50);
    }

    function drawCannon() {
      push(); translate(80, height - 70);
      rotate(-radians(angleSlider.value()));
      fill(100, 100, 100); rect(0, -15, 80, 30);
      pop();
    }

    function launchBall() {
      resetSimulation();
      let v0 = speedSlider.value(), theta = radians(angleSlider.value());
      balls.push(new Ball(v0 * cos(theta), -v0 * sin(theta), false));
      balls.push(new Ball(v0 * cos(theta), -v0 * sin(theta), true));
      trails = [[], []];
    }

    function updateBalls() {
      for (let i = 0; i < balls.length; i++) {
        let b = balls[i];
        if (b.active) { b.update(); b.display(i); }
      }
    }

    class Ball {
      constructor(vx, vy, hasDrag) {
        this.x = 80; this.y = height - 85;
        this.vx = vx; this.vy = vy;
        this.hasDrag = hasDrag;
        this.active = true; this.landed = false;
        this.trail = [];
      }
      update() {
        if (!this.active) return;
        let ax = 0, ay = g;
        if (this.hasDrag) {
          let speed = sqrt(this.vx**2 + this.vy**2);
          let drag = dragCoeff * speed * speed;
          let angle = atan2(this.vy, this.vx);
          ax -= drag * cos(angle); ay += drag * sin(angle);
        }
        this.vx += ax * dt; this.vy += ay * dt;
        this.x += this.vx * dt * scale; this.y += this.vy * dt * scale;
        this.trail.push({x: this.x, y: this.y});
        if (this.trail.length > 300) this.trail.shift();
        if (this.y > height - 70) { this.y = height - 70; this.active = false; this.landed = true; }
      }
      display(i) {
        let col = this.hasDrag ? color(231, 76, 60) : color(52, 152, 219);
        fill(col); noStroke(); ellipse(this.x, this.y, 12, 12);
        stroke(col); strokeWeight(2); noFill();
        beginShape();
        for (let p of this.trail) vertex(p.x, p.y);
        endShape();
      }
    }

    function resetSimulation() {
      balls = []; trails = [[], []];
    }

    // UI
    let speedSlider, angleSlider, dragToggle, launchBtn, resetBtn;
    function setupUI() {
      speedSlider = select('#speed'); angleSlider = select('#angle');
      dragToggle = select('#drag-toggle'); launchBtn = select('#launch-btn');
      resetBtn = select('#reset-btn');
      speedSlider.input(() => select('#speed-val').html(speedSlider.value()));
      angleSlider.input(() => select('#angle-val').html(angleSlider.value()));
      dragToggle.changed(() => select('#drag-status').html(dragToggle.checked ? "ON" : "OFF"));
      launchBtn.mousePressed(() => launched = true);
      resetBtn.mousePressed(resetSimulation);
      select('#speed-val').html(30); select('#angle-val').html(45);
    }

    // Graph
    function updateGraph() {
      let idealRange = balls[0].x / scale, realRange = balls[1].x / scale;
      let angle = angleSlider.value();
      graphData.ideal.push({angle, range: idealRange});
      graphData.real.push({angle, range: realRange});
      graphData.ideal.sort((a,b) => a.angle - b.angle);
      graphData.real.sort((a,b) => a.angle - b.angle);
      drawGraph();
    }

    function drawGraph() {
      let ctx = select('#graph-canvas').elt.getContext('2d');
      let w = 600, h = 300; ctx.clearRect(0,0,w,h);
      let maxRange = Math.max(...graphData.ideal.map(d=>d.range), 100);
      ctx.strokeStyle = '#333'; ctx.lineWidth = 2;
      ctx.beginPath(); ctx.moveTo(50, h-50); ctx.lineTo(50,50); ctx.lineTo(w-50,50); ctx.stroke();
      ctx.fillStyle = '#333'; ctx.font = '12px Arial';
      ctx.fillText('Range (m)', w/2-30, h-10);
      ctx.save(); ctx.translate(15,h/2); ctx.rotate(-Math.PI/2);
      ctx.fillText('Angle (°)', -30,0); ctx.restore();

      // Ideal
      ctx.strokeStyle = '#3498db'; ctx.beginPath();
      graphData.ideal.forEach((d,i) => {
        let x = 50 + (d.angle/90)*(w-100);
        let y = h-50 - (d.range/maxRange)*(h-100);
        i===0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
      }); ctx.stroke();

      // Real
      ctx.strokeStyle = '#e74c3c'; ctx.beginPath();
      graphData.real.forEach((d,i) => {
        let x = 50 + (d.angle/90)*(w-100);
        let y = h-50 - (d.range/maxRange)*(h-100);
        i===0 ? ctx.moveTo(x,y) : ctx.lineTo(x,y);
      }); ctx.stroke();

      // Legend
      ctx.fillStyle = '#3498db'; ctx.fillRect(w-150,20,15,15);
      ctx.fillStyle = '#e74c3c'; ctx.fillRect(w-150,45,15,15);
      ctx.fillStyle = '#333'; ctx.font = '14px Arial';
      ctx.fillText('No Drag', w-130,33); ctx.fillText('With Drag', w-130,58);
    }

    function windowResized() { resizeCanvas(800, 400); }
  </script>
</body>
</html>
