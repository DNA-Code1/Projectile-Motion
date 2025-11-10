<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Wave Interference Simulator</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/addons/p5.sound.min.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f4f7fa;
      color: #222;
      text-align: center;
      padding: 20px;
    }
    h1 { color: #2c3e50; margin-bottom: 10px; }
    p { margin-bottom: 20px; color: #555; }
    #canvas-container {
      width: 800px; height: 500px; margin: 20px auto;
      border: 2px solid #3498db; border-radius: 12px;
      background: #fff;
      box-shadow: 0 6px 20px rgba(0,0,0,0.1);
    }
    .controls {
      max-width: 700px; margin: 20px auto; padding: 18px;
      background: white; border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
      display: grid; grid-template-columns: 1fr 1fr; gap: 15px;
    }
    .control-group {
      display: flex; flex-direction: column; gap: 6px;
    }
    label { font-weight: 600; color: #2c3e50; text-align: left; }
    input[type="range"] { width: 100%; }
    button, select {
      padding: 10px; font-size: 15px; border-radius: 6px; border: none;
      background: #3498db; color: white; cursor: pointer;
    }
    button:hover { background: #2980b9; }
    .legend {
      margin-top: 15px; font-size: 14px; color: #444;
    }
    .legend span { margin: 0 12px; font-weight: bold; }
    .antinode { color: #e74c3c; }
    .node { color: #2c3e50; }
  </style>
</head>
<body>
  <h1>Wave Interference Simulator</h1>
  <p>Two point sources → Real-time interference → Sound or Light</p>

  <div id="canvas-container"></div>

  <div class="controls">
    <div class="control-group">
      <label>Source 1 Frequency (Hz): <span id="f1-val">440</span></label>
      <input type="range" id="f1" min="200" max="800" value="440" step="1"/>
    </div>
    <div class="control-group">
      <label>Source 2 Frequency (Hz): <span id="f2-val">440</span></label>
      <input type="range" id="f2" min="200" max="800" value="440" step="1"/>
    </div>
    <div class="control-group">
      <label>Wavelength (px): <span id="wl-val">80</span></label>
      <input type="range" id="wl" min="40" max="150" value="80" step="1"/>
    </div>
    <div class="control-group">
      <label>Phase Diff (°): <span id="phase-val">0</span></label>
      <input type="range" id="phase" min="0" max="360" value="0" step="1"/>
    </div>
    <div class="control-group">
      <label>Mode:</label>
      <select id="mode">
        <option value="light">Light (Visual)</option>
        <option value="sound">Sound (Audio)</option>
      </select>
    </div>
    <div class="control-group">
      <button id="reset-btn">Reset Sources</button>
    </div>
  </div>

  <div class="legend">
    <span class="antinode">Red Dots = Antinodes (Constructive)</span> |
    <span class="node">Dark Areas = Nodes (Destructive)</span>
  </div>

  <script>
    let osc1, osc2;
    let source1, source2;
    let waveField = [];
    let gridSize = 20;
    let mode = 'light';

    function setup() {
      let canvas = createCanvas(800, 500);
      canvas.parent('canvas-container');
      angleMode(DEGREES);

      // Initialize oscillators
      osc1 = new p5.Oscillator('sine');
      osc2 = new p5.Oscillator('sine');
      osc1.start(); osc2.start();
      osc1.amp(0); osc2.amp(0);

      // Sources
      source1 = createVector(width * 0.3, height / 2);
      source2 = createVector(width * 0.7, height / 2);

      initWaveField();
      setupUI();
    }

    function draw() {
      background(255);
      updateMode();
      updateWaveField();
      drawWaves();
      drawSources();
      drawInterferencePoints();
    }

    function initWaveField() {
      waveField = [];
      for (let x = 0; x < width; x += gridSize) {
        let col = [];
        for (let y = 0; y < height; y += gridSize) {
          col.push(0);
        }
        waveField.push(col);
      }
    }

    function updateWaveField() {
      let f1 = f1Slider.value(), f2 = f2Slider.value();
      let wl = wlSlider.value();
      let phaseDiff = phaseSlider.value();
      let t = frameCount * 0.05;

      for (let i = 0; i < waveField.length; i++) {
        for (let j = 0; j < waveField[i].length; j++) {
          let x = i * gridSize + gridSize / 2;
          let y = j * gridSize + gridSize / 2;

          let d1 = dist(x, y, source1.x, source1.y);
          let d2 = dist(x, y, source2.x, source2.y);

          let wave1 = sin((2 * PI * f1 * t) - (2 * PI * d1 / wl));
          let wave2 = sin((2 * PI * f2 * t) - (2 * PI * d2 / wl) + radians(phaseDiff));

          waveField[i][j] = wave1 + wave2;
        }
      }
    }

    function drawWaves() {
      noStroke();
      for (let i = 0; i < waveField.length; i++) {
        for (let j = 0; j < waveField[i].length; j++) {
          let amp = waveField[i][j];
          let brightness = map(abs(amp), 0, 2, 255, 0);
          let x = i * gridSize;
          let y = j * gridSize;

          if (mode === 'light') {
            fill(brightness);
          } else {
            let gray = map(amp, -2, 2, 0, 255);
            fill(gray);
          }
          rect(x, y, gridSize, gridSize);
        }
      }
    }

    function drawSources() {
      fill(52, 152, 219); stroke(0); strokeWeight(2);
      ellipse(source1.x, source1.y, 20, 20);
      ellipse(source2.x, source2.y, 20, 20);

      fill(255); noStroke(); textAlign(CENTER, CENTER);
      textSize(12); text("S1", source1.x, source1.y);
      text("S2", source2.x, source2.y);
    }

    function drawInterferencePoints() {
      strokeWeight(6);
      for (let i = 0; i < waveField.length; i += 2) {
        for (let j = 0; j < waveField[i].length; j += 2) {
          let amp = waveField[i][j];
          let x = i * gridSize + gridSize / 2;
          let y = j * gridSize + gridSize / 2;

          if (abs(amp) > 1.8) {
            stroke(231, 76, 60); // Red = Antinode
            point(x, y);
          }
        }
      }
    }

    function updateMode() {
      mode = modeSelect.value();
      if (mode === 'sound') {
        osc1.freq(f1Slider.value());
        osc2.freq(f2Slider.value());
        osc1.amp(0.2); osc2.amp(0.2);
      } else {
        osc1.amp(0); osc2.amp(0);
      }
    }

    // UI Elements
    let f1Slider, f2Slider, wlSlider, phaseSlider, modeSelect, resetBtn;

    function setupUI() {
      f1Slider = select('#f1'); f2Slider = select('#f2');
      wlSlider = select('#wl'); phaseSlider = select('#phase');
      modeSelect = select('#mode'); resetBtn = select('#reset-btn');

      f1Slider.input(updateLabels);
      f2Slider.input(updateLabels);
      wlSlider.input(updateLabels);
      phaseSlider.input(updateLabels);
      modeSelect.changed(updateMode);
      resetBtn.mousePressed(() => {
        source1.set(width * 0.3, height / 2);
        source2.set(width * 0.7, height / 2);
        initWaveField();
      });

      updateLabels();
    }

    function updateLabels() {
      select('#f1-val').html(f1Slider.value());
      select('#f2-val').html(f2Slider.value());
      select('#wl-val').html(wlSlider.value());
      select('#phase-val').html(phaseSlider.value());
    }

    // Allow dragging sources
    function mousePressed() {
      if (dist(mouseX, mouseY, source1.x, source1.y) < 20) {
        source1.dragging = true;
      } else if (dist(mouseX, mouseY, source2.x, source2.y) < 20) {
        source2.dragging = true;
      }
    }

    function mouseDragged() {
      if (source1.dragging) {
        source1.set(constrain(mouseX, 20, width - 20), constrain(mouseY, 20, height - 20));
      } else if (source2.dragging) {
        source2.set(constrain(mouseX, 20, width - 20), constrain(mouseY, 20, height - 20));
      }
    }

    function mouseReleased() {
      source1.dragging = false;
      source2.dragging = false;
    }

    function windowResized() {
      resizeCanvas(800, 500);
      initWaveField();
    }
  </script>
</body>
</html>
