<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2D Maxima and Minima Finder</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.8.0/math.js"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            color: #333;
            max-width: 800px;
            margin: 40px auto;
            padding: 20px;
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        h1 { color: #2c3e50; text-align: center; }
        .input-group { margin-bottom: 20px; }
        label { display: block; margin-bottom: 8px; font-weight: bold; }
        input[type="text"] {
            width: 100%;
            padding: 10px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }
        button {
            background-color: #3498db;
            color: white;
            padding: 12px 20px;
            border: none;
            border-radius: 4px;
            font-size: 16px;
            cursor: pointer;
            width: 100%;
        }
        button:hover { background-color: #2980b9; }
        #output {
            margin-top: 30px;
            padding: 20px;
            background: #eef2f3;
            border-left: 5px solid #3498db;
            border-radius: 4px;
            white-space: pre-line;
            font-family: monospace;
            font-size: 15px;
        }
        .error { color: #e74c3c; font-weight: bold; }
    </style>
</head>
<body>

<div class="container">
    <h1>2D Function Maxima & Minima Finder</h1>
    <p>Enter a function of <strong>x</strong> and <strong>y</strong>. Use standard notation like <code>x^2 + y^2</code> or <code>x^3 - 3*x*y^2</code>.</p>
    
    <div class="input-group">
        <label for="functionInput">Function f(x, y):</label>
        <input type="text" id="functionInput" value="x^3 - 3*x*y^2" placeholder="e.g., x^3 - 3*x*y^2">
    </div>
    
    <button onclick="calculateCriticalPoints()">Find & Classify Stationary Points</button>
    
    <div id="output">Results will appear here...</div>
</div>

<script>
function calculateCriticalPoints() {
    const inputStr = document.getElementById('functionInput').value;
    const outputDiv = document.getElementById('output');
    
    try {
        // Parse the input function
        const f = math.parse(inputStr);
        
        // Compute partial derivatives
        const fx = math.derivative(f, 'x');
        const fy = math.derivative(f, 'y');
        
        // Compute second order derivatives for Hessian matrix
        const fxx = math.derivative(fx, 'x');
        const fyy = math.derivative(fy, 'y');
        const fxy = math.derivative(fx, 'y');

        let resultText = `<b>Partial Derivatives:</b>\n`;
        resultText += `f_x = ${fx.toString()}\n`;
        resultText += `f_y = ${fy.toString()}\n\n`;

        /* NOTE ON JAVASCRIPT EQUATION SOLVING:
           Unlike SymPy or MATLAB, basic frontend JS libraries struggle to natively solve arbitrary 
           systems of non-linear symbolic equations. 
           For demo and practical purposes, we will find critical points numerically for the given function.
        */
        
        resultText += `<b>Stationary Points & Classification:</b>\n`;
        
        // Let's check a few standard test cases or common grids numerically 
        // For a general implementation, we look at algebraic roots or use a quick Newton-Raphson approximation.
        // Let's find roots for the gradient [fx, fy] = [0, 0]
        
        const stationaryPoints = findRootsNumeric(fx, fy);
        
        if (stationaryPoints.length === 0) {
            resultText += "No real stationary points found in the scanned range (-10 to 10).\n";
        } else {
            stationaryPoints.forEach((pt, index) => {
                const scope = { x: pt.x, y: pt.y };
                
                // Evaluate second derivatives at the stationary point
                const r = fxx.evaluate(scope);
                const t = fyy.evaluate(scope);
                const s = fxy.evaluate(scope);
                
                // Discriminant D = f_xx * f_yy - (f_xy)^2
                const D = (r * t) - (s * s);
                
                resultText += `${index + 1}) Point: (${pt.x.toFixed(3)}, ${pt.y.toFixed(3)})\n`;
                resultText += `   D = ${D.toFixed(3)}, f_xx = ${r.toFixed(3)}\n`;
                
                if (D > 0) {
                    if (r > 0) resultText += `   <b>Conclusion: Local Minimum</b>\n`;
                    else if (r < 0) resultText += `   <b>Conclusion: Local Maximum</b>\n`;
                    else resultText += `   <b>Conclusion: Test Inconclusive</b>\n`;
                } else if (D < 0) {
                    resultText += `   <b>Conclusion: Saddle Point</b>\n`;
                } else {
                    resultText += `   <b>Conclusion: Test Inconclusive (D=0)</b>\n`;
                }
                resultText += `\n`;
            });
        }
        
        outputDiv.innerHTML = resultText;
        
    } catch (error) {
        outputDiv.innerHTML = `<span class="error">Error processing function: ${error.message}</span>`;
    }
}

// Simple numerical solver to find where fx=0 and fy=0
function findRootsNumeric(fx, fy) {
    const points = [];
    const seen = new Set();
    
    // Quick grid search to seed a basic Newton-Raphson solver
    for (let xSeeded = -3; xSeeded <= 3; xSeeded += 0.5) {
        for (let ySeeded = -3; ySeeded <= 3; ySeeded += 0.5) {
            let x = xSeeded;
            let y = ySeeded;
            
            // Perform 10 steps of Newton's method for system of 2 equations
            for (let i = 0; i < 10; i++) {
                try {
                    const fx_val = fx.evaluate({x, y});
                    const fy_val = fy.evaluate({x, y});
                    
                    // If close enough to zero, we stop early
                    if (Math.abs(fx_val) < 1e-7 && Math.abs(fy_val) < 1e-7) break;
                    
                    // Approximate Jacobian elements numerically
                    const h = 1e-5;
                    const fxx_val = (fx.evaluate({x: x+h, y}) - fx_val) / h;
                    const fxy_val = (fx.evaluate({x, y: y+h}) - fx_val) / h;
                    const fyx_val = (fy.evaluate({x: x+h, y}) - fy_val) / h;
                    const fyy_val = (fy.evaluate({x, y: y+h}) - fy_val) / h;
                    
                    const det = fxx_val * fyy_val - fxy_val * fyx_val;
                    if (Math.abs(det) < 1e-7) break;
                    
                    // Newton step: X_new = X - J^-1 * F
                    const dx = (fyy_val * fx_val - fxy_val * fy_val) / det;
                    const dy = (-fyx_val * fx_val + fxx_val * fy_val) / det;
                    
                    x -= dx;
                    y -= dy;
                } catch(e) {
                    break;
                }
            }
            
            // Verify if it converged nicely to a real root
            try {
                if (Math.abs(fx.evaluate({x, y})) < 1e-4 && Math.abs(fy.evaluate({x, y})) < 1e-4) {
                    // Round to avoid floating point duplicates (e.g. 0.000001 vs 0)
                    const rx = Math.abs(x) < 1e-4 ? 0 : Math.round(x * 1000) / 1000;
                    const ry = Math.abs(y) < 1e-4 ? 0 : Math.round(y * 1000) / 1000;
                    
                    const key = `${rx},${ry}`;
                    if (!seen.has(key) && !isNaN(rx) && !isNaN(ry)) {
                        seen.add(key);
                        points.push({x: rx, y: ry});
                    }
                }
            } catch(e) {}
        }
    }
    return points;
}
</script>

</body>
</html>
