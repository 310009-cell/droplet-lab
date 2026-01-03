# droplet-lab
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>液滴彈跳實驗室 - 在線模擬與數據分析</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { background-color: #f8fafc; font-family: 'PingFang TC', sans-serif; }
        canvas#simCanvas { background: #ffffff; cursor: crosshair; }
        .panel-shadow { box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
    </style>
</head>
<body class="min-h-screen flex flex-col">

    <header class="bg-slate-800 text-white p-4 shadow-lg">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <h1 class="text-xl font-bold flex items-center gap-2">
                <span class="bg-blue-500 p-1 rounded">🧪</span>
                液滴彈跳實驗室 (數據分析版)
            </h1>
            <div class="text-[10px] text-slate-400 text-right hidden sm:block">
                物理參數：萊頓弗羅斯特效應分析<br>
                橫軸：時間(s) / 縱軸：彈跳次數
            </div>
        </div>
    </header>

    <main class="flex-grow flex flex-col lg:flex-row p-4 gap-4 max-w-7xl mx-auto w-full">
        <!-- 模擬區 -->
        <div class="w-full lg:w-2/3 flex flex-col gap-4">
            <div class="relative bg-white rounded-xl border border-slate-200 overflow-hidden panel-shadow">
                <canvas id="simCanvas" class="w-full h-[400px]"></canvas>
                
                <div class="absolute top-4 left-4 flex flex-wrap gap-2">
                    <div class="bg-white/90 backdrop-blur px-3 py-2 rounded-lg border border-slate-200 shadow-sm">
                        <p class="text-[10px] text-slate-500 uppercase font-bold">彈跳次數</p>
                        <p id="bounceCount" class="text-xl font-mono font-bold text-blue-600">0</p>
                    </div>
                    <div class="bg-white/90 backdrop-blur px-3 py-2 rounded-lg border border-slate-200 shadow-sm">
                        <p class="text-[10px] text-slate-500 uppercase font-bold">高度 (cm)</p>
                        <p id="currentHeightDisplay" class="text-xl font-mono font-bold text-emerald-600">0.0</p>
                    </div>
                    <div class="bg-white/90 backdrop-blur px-3 py-2 rounded-lg border border-slate-200 shadow-sm">
                        <p class="text-[10px] text-slate-500 uppercase font-bold">時間 (s)</p>
                        <p id="timeDisplay" class="text-xl font-mono font-bold text-orange-600">0.00</p>
                    </div>
                </div>

                <div id="hint" class="absolute inset-0 flex items-center justify-center pointer-events-none text-slate-300">
                    <p class="bg-white/50 px-4 py-2 rounded-full border border-dashed border-slate-300">點擊畫面或按下方按鈕釋放液滴</p>
                </div>
            </div>

            <!-- 控制面板 -->
            <div class="bg-white p-6 rounded-xl border border-slate-200 panel-shadow grid grid-cols-1 md:grid-cols-3 gap-6">
                <div class="space-y-4">
                    <label class="block text-sm font-bold text-slate-700 border-l-4 border-blue-500 pl-2">液體預設</label>
                    <select id="liquidPreset" class="w-full p-2 bg-slate-50 border rounded-lg text-sm">
                        <option value="water">純水 (預設)</option>
                        <option value="ethanol">酒精 (低張力)</option>
                        <option value="oil">食用油 (高黏度)</option>
                        <option value="custom">自定義</option>
                    </select>
                    <div>
                        <div class="flex justify-between mb-1 text-xs"><span>溫度 (°C)</span><span id="v-temp">200</span></div>
                        <input type="range" id="temperature" min="20" max="300" value="200" class="w-full">
                    </div>
                </div>

                <div class="space-y-4 text-sm">
                    <label class="block text-sm font-bold text-slate-700 border-l-4 border-emerald-500 pl-2">物理變數</label>
                    <div>
                        <div class="flex justify-between mb-1 text-xs"><span>表面張力 (N/m)</span><span id="v-tension">0.072</span></div>
                        <input type="range" id="surfaceTension" min="0.01" max="0.1" step="0.001" value="0.072" class="w-full">
                    </div>
                    <div>
                        <div class="flex justify-between mb-1 text-xs"><span>黏度係數</span><span id="v-visc">1.0</span></div>
                        <input type="range" id="viscosity" min="0.1" max="5.0" step="0.1" value="1.0" class="w-full">
                    </div>
                </div>

                <div class="flex flex-col justify-end gap-2 text-sm">
                    <div>
                        <div class="flex justify-between mb-1 text-xs"><span>釋放高度 (cm)</span><span id="v-height">10</span></div>
                        <input type="range" id="dropHeight" min="2" max="25" value="10" class="w-full">
                    </div>
                    <div class="flex gap-2 pt-2">
                        <button id="clearChartBtn" class="flex-1 bg-slate-100 hover:bg-slate-200 py-2 rounded-lg transition font-bold text-slate-600">清空圖表</button>
                        <button id="dropBtn" class="flex-[2] bg-blue-600 hover:bg-blue-700 text-white py-2 rounded-lg shadow-md transition font-bold">釋放並記錄</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- 圖表分析區 -->
        <div class="w-full lg:w-1/3 flex flex-col gap-4">
            <div class="bg-white p-4 rounded-xl border border-slate-200 panel-shadow flex-grow flex flex-col">
                <h3 class="font-bold text-slate-800 mb-4 flex items-center justify-between">
                    <span>彈跳次數分析 (折線圖)</span>
                </h3>
                <div class="flex-grow relative min-h-[350px]">
                    <canvas id="analysisChart"></canvas>
                </div>
                <div class="mt-4 p-3 bg-blue-50 rounded-lg text-[11px] text-blue-800">
                    <strong>分析提示：</strong><br>
                    橫軸為模擬經過時間，縱軸為彈跳次數。<br>
                    不同顏色代表不同次實驗，變淡的線為歷史紀錄。
                </div>
            </div>
        </div>
    </main>

    <script>
        const canvas = document.getElementById('simCanvas');
        const ctx = canvas.getContext('2d');
        const g = 980; 
        let droplet = null;
        let surfaceY = 0;
        let chart = null;
        let experimentCount = 0;

        const presets = {
            water: { tension: 0.072, viscosity: 1.0, temp: 200 },
            ethanol: { tension: 0.022, viscosity: 0.8, temp: 200 },
            oil: { tension: 0.032, viscosity: 3.5, temp: 200 }
        };
        const colors = ['#3b82f6', '#ef4444', '#10b981', '#f59e0b', '#8b5cf6'];

        function initChart() {
            const ctxChart = document.getElementById('analysisChart').getContext('2d');
            chart = new Chart(ctxChart, {
                type: 'line',
                data: { datasets: [] },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: { type: 'linear', position: 'bottom', title: { display: true, text: '時間 (秒)' }, min: 0, max: 5 },
                        y: { min: 0, suggestedMax: 5, ticks: { stepSize: 1 }, title: { display: true, text: '累計彈跳次數' } }
                    },
                    animation: false,
                    plugins: { legend: { position: 'bottom', labels: { boxWidth: 8, font: { size: 10 } } } }
                }
            });
        }

        function resize() {
            canvas.width = canvas.offsetWidth;
            canvas.height = canvas.offsetHeight;
            surfaceY = canvas.height * 0.85;
        }

        window.addEventListener('resize', resize);
        resize();
        initChart();

        // UI 連動
        document.querySelectorAll('input[type=range]').forEach(input => {
            input.addEventListener('input', (e) => {
                const map = { surfaceTension:'tension', viscosity:'visc', dropHeight:'height', temperature:'temp' };
                document.getElementById('v-' + map[e.target.id]).innerText = e.target.value;
            });
        });

        document.getElementById('liquidPreset').addEventListener('change', (e) => {
            const p = presets[e.target.value];
            if(p) {
                document.getElementById('surfaceTension').value = p.tension;
                document.getElementById('viscosity').value = p.viscosity;
                document.getElementById('temperature').value = p.temp;
                document.getElementById('v-tension').innerText = p.tension;
                document.getElementById('v-visc').innerText = p.viscosity;
                document.getElementById('v-temp').innerText = p.temp;
            }
        });

        class Droplet {
            constructor(x, y) {
                this.x = x; this.y = y;
                this.radius = 8; this.vy = 0;
                this.active = true; this.bounces = 0; this.elapsedTime = 0;
                this.liquidName = document.getElementById('liquidPreset').options[document.getElementById('liquidPreset').selectedIndex].text;
                
                chart.data.datasets.forEach(ds => {
                    ds.borderColor = ds.borderColor.replace('1)', '0.2)');
                    ds.borderWidth = 1;
                    ds.pointRadius = 0;
                });

                experimentCount++;
                const color = colors[(experimentCount - 1) % colors.length];
                chart.data.datasets.push({
                    label: `#${experimentCount} ${this.liquidName}`,
                    data: [{x: 0, y: 0}],
                    borderColor: color,
                    borderWidth: 2,
                    pointRadius: 3,
                    showLine: true,
                    tension: 0.1
                });
            }

            update(dt) {
                if (!this.active) return;
                const tension = parseFloat(document.getElementById('surfaceTension').value);
                const temp = parseFloat(document.getElementById('temperature').value);
                const visc = parseFloat(document.getElementById('viscosity').value);

                this.vy += g * dt;
                this.y += this.vy * dt;
                this.elapsedTime += dt;

                let h = Math.max(0, (surfaceY - this.y - this.radius) / 20);
                document.getElementById('currentHeightDisplay').innerText = h.toFixed(1);
                document.getElementById('timeDisplay').innerText = this.elapsedTime.toFixed(2);

                if (this.y + this.radius >= surfaceY) {
                    this.y = surfaceY - this.radius;
                    let restitution = (tension / 0.072) * 0.6 * (1 / Math.sqrt(visc));
                    if (temp > 180) restitution += 0.28; 

                    if (Math.abs(this.vy) > 40 && restitution > 0.1) {
                        this.vy = -this.vy * restitution;
                        this.bounces++;
                        document.getElementById('bounceCount').innerText = this.bounces;
                        chart.data.datasets[chart.data.datasets.length-1].data.push({ x: this.elapsedTime, y: this.bounces });
                    } else {
                        this.active = false;
                        chart.data.datasets[chart.data.datasets.length-1].data.push({ x: this.elapsedTime, y: this.bounces });
                    }
                }
                if (this.elapsedTime > chart.options.scales.x.max) chart.options.scales.x.max = Math.ceil(this.elapsedTime + 1);
                chart.update();
            }

            draw() {
                if (!this.active) return;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                const grad = ctx.createRadialGradient(this.x-2, this.y-2, 1, this.x, this.y, this.radius);
                grad.addColorStop(0, '#ffffff'); grad.addColorStop(1, '#3b82f6');
                ctx.fillStyle = grad; ctx.fill();
                ctx.strokeStyle = '#1d4ed8'; ctx.stroke();
            }
        }

        document.getElementById('dropBtn').addEventListener('click', () => {
            const h = parseFloat(document.getElementById('dropHeight').value) * 20;
            document.getElementById('hint').classList.add('hidden');
            droplet = new Droplet(canvas.width / 2, surfaceY - h);
        });

        canvas.addEventListener('mousedown', (e) => {
            const rect = canvas.getBoundingClientRect();
            document.getElementById('hint').classList.add('hidden');
            droplet = new Droplet(e.clientX - rect.left, e.clientY - rect.top);
        });

        document.getElementById('clearChartBtn').addEventListener('click', () => {
            chart.data.datasets = []; experimentCount = 0; chart.options.scales.x.max = 5; chart.update();
        });

        function loop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#cbd5e1'; ctx.fillRect(0, surfaceY, canvas.width, 2);
            if (droplet) { droplet.update(0.016); droplet.draw(); }
            requestAnimationFrame(loop);
        }
        loop();
    </script>
</body>
</html>
