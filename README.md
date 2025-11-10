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
      margin: 30px auto; width: 650px; padding: 20px;
      background: white; border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    #graph-canvas {
      width: 100%; height: 350px; border: 1px solid #ddd;
      border-radius: 6px; background: #fdfdfd;
    }
    .legend {
      margin-top: 10px; text-align: right; font-size: 14px; color: #555;
    }
    .legend span { margin-left: 15px; }
    .legend .ideal { color: #3498db; font-weight: bold; }
    .legend .real { color: #e74c3c; font-weight: bold; }
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
    <h3>Range vs Launch Angle</h3>
    <canvas id="graph-canvas"></canvas>
    <div class="legend">
      <span class="ideal">No Drag</span> |
      <span class="real">With Drag</span>
    </div>
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

    // Enhanced Graph with Labels, Grid, Scale
    function updateGraph() {
      let idealRange = balls[0].x / scale;
      let realRange = balls[1].x / scale;
      let angle = angleSlider.value();

      graphData.ideal.push({angle, range: idealRange});
      graphData.real.push({angle, range: realRange});

      graphData.ideal.sort((a,b) => a.angle - b.angle);
      graphData.real.sort((a,b) => a.angle - b.angle);

      drawGraph();
    }

    function drawGraph() {
      let ctx = select('#graph-canvas').elt.getContext('2d');
      let w = 650, h = 350, padding = 60;
      ctx.clearRect(0, 0, w, h);

      // Find max range for Y-scale
      let allRanges = [...graphData.ideal, ...graphData.real].map(d => d.range);
      let maxRange = Math.max(...allRanges, 100);
      let yScale = (h - 2*padding) / maxRange;
      let xScale = (w - 2*padding) / 90;

      // Background grid
      ctx.strokeStyle = '#eee'; ctx.lineWidth = 1;
      for (let i = 0; i <= 10; i++) {
        let y = padding + i * (h - 2*padding)/10;
        ctx.beginPath(); ctx.moveTo(padding, y); ctx.lineTo(w - padding, y); ctx.stroke();
      }
      for (let i = 0; i <= 9; i++) {
        let x = padding + i * (w - 2*padding)/9;
        ctx.beginPath(); ctx.moveTo(x, padding); ctx.lineTo(x, h - padding); ctx.stroke();
      }

      // Axes
      ctx.strokeStyle = '#333'; ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(padding, h - padding); ctx.lineTo(padding, padding);
      ctx.lineTo(w - padding, padding); ctx.stroke();

      // Axis Labels
      ctx.fillStyle = '#333'; ctx.font = '14px Arial';
      ctx.textAlign = 'center';
      ctx.fillText('Launch Angle (degrees)', w/2, h - 15);
      ctx.save(); ctx.translate(20, h/2); ctx.rotate(-Math.PI/2);
      ctx.fillText('Range (meters)', 0, 0); ctx.restore();

      // Tick marks and labels
      ctx.font = '12px Arial'; ctx.textAlign = 'center';
      for (let i = 0; i <= 9; i++) {
        let angle = i * 10;
        let x = padding + angle * xScale;
        ctx.fillText(angle, x, h - padding + 20);
      }
      ctx.textAlign = 'right';
      for (let i = 0; i <= 5; i++) {
        let range = (maxRange / 5) * i;
        let y = h - padding - range * yScale;
        ctx.fillText(Math.round(range), padding - 10, y + 5);
      }

      // Plot: No Drag (Blue)
      ctx.strokeStyle = '#3498db'; ctx.lineWidth = 3;
      ctx.beginPath();
      graphData.ideal.forEach((d, i) => {
        let x = padding + d.angle * xScale;
        let y = h - padding - d.range * yScale;
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
        // Dot at data point
        ctx.fillStyle = '#3498db';
        ctx.beginPath(); ctx.arc(x, y, 4, 0, Math.PI*2); ctx.fill();
      });
      ctx.stroke();

      // Plot: With Drag (Red)
      ctx.strokeStyle = '#e74c3c'; ctx.lineWidth = 3;
      ctx.beginPath();
      graphData.real.forEach((d, i) => {
        let x = padding + d.angle * xScale;
        let y = h - padding - d.range * yScale;
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
        ctx.fillStyle = '#e74c3c';
        ctx.beginPath(); ctx.arc(x, y, 4, 0, Math.PI*2); ctx.fill();
      });
      ctx.stroke();
    }

    function windowResized() { resizeCanvas(800, 400); }
  </script>
</body>
</html>
