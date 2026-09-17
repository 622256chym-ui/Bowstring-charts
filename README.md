html<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MCB Interactive Bowstring Chart Builder</title>
    <style>
        :root {
            --bg-main: #1e1e24;
            --bg-card: #2a2a35;
            --border-color: #3f3f50;
            --text-main: #f3f4f6;
            --text-muted: #9ca3af;
            --accent-blue: #2563eb;
            --accent-green: #10b981;
            --accent-red: #ef4444;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }
        body { background-color: var(--bg-main); color: var(--text-main); padding: 20px; line-height: 1.5; }
        header { margin-bottom: 25px; border-bottom: 2px solid var(--border-color); padding-bottom: 15px; display: flex; justify-content: space-between; align-items: center; }
        h1 { font-size: 1.5rem; letter-spacing: 0.5px; }
        
        .layout-grid { display: grid; grid-template-columns: 320px 1fr; gap: 20px; align-items: start; }
        .panel { background-color: var(--bg-card); border: 1px solid var(--border-color); border-radius: 8px; padding: 20px; }
        
        .form-group { margin-bottom: 15px; }
        label { display: block; font-size: 0.85rem; color: var(--text-muted); margin-bottom: 5px; font-weight: 600; text-transform: uppercase; }
        select, input, textarea { width: 100%; background-color: var(--bg-main); border: 1px solid var(--border-color); color: var(--text-main); padding: 8px 12px; border-radius: 6px; font-size: 0.95rem; }
        select:focus, input:focus { border-color: var(--accent-blue); outline: none; }
        
        .btn-group { display: flex; gap: 10px; margin-top: 15px; }
        button { flex: 1; background-color: var(--accent-blue); border: none; color: white; padding: 10px; border-radius: 6px; font-weight: 600; cursor: pointer; transition: opacity 0.2s; font-size: 0.9rem; }
        button:hover { opacity: 0.9; }
        button.secondary { background-color: #4b5563; }
        button.success { background-color: var(--accent-green); }

        /* Blueprint Canvas Area */
        .canvas-container { display: flex; flex-direction: column; gap: 20px; }
        .blueprint { background-color: #fafafa; color: #111827; border-radius: 8px; padding: 25px; box-shadow: inset 0 0 10px rgba(0,0,0,0.05); border: 2px solid #d1d5db; position: relative; }
        .blueprint-title { font-size: 1.2rem; font-weight: 700; color: #1e3a8a; border-bottom: 2px solid #1e3a8a; padding-bottom: 5px; margin-bottom: 25px; text-transform: uppercase; text-align: center; }
        
        /* String Component Renderers */
        .string-track { position: relative; margin: 45px 0; border: 1px dashed #9ca3af; padding: 10px 0; background: #f3f4f6; border-radius: 4px; }
        .track-label { position: absolute; top: -22px; left: 5px; font-size: 0.75rem; font-weight: bold; text-transform: uppercase; color: #4b5563; }
        .base-line { height: 4px; background-color: #111827; width: 100%; position: relative; display: flex; align-items: center; }
        .loop-end { width: 12px; height: 12px; border: 3px solid #111827; border-radius: 50%; background: #fafafa; position: absolute; top: -4px; }
        .loop-left { left: -6px; } .loop-right { right: -6px; }
        
        /* Serving Segments overlays */
        .serving-segment { position: absolute; height: 12px; top: -4px; border-radius: 2px; opacity: 0.85; display: flex; align-items: center; justify-content: center; }
        .serving-segment.green { background-color: #059669; border: 1px solid #047857; }
        .serving-segment.red { background-color: #dc2626; border: 1px solid #b91c1c; }
        .serving-segment.blue { background-color: #2563eb; border: 1px solid #1d4ed8; }

        /* Measurement Callout Rectangles */
        .marker { position: absolute; top: -35px; transform: translateX(-50%); display: flex; flex-direction: column; align-items: center; }
        .marker-box { background: #3b82f6; color: white; font-size: 0.7rem; padding: 2px 6px; border-radius: 4px; font-weight: bold; white-space: nowrap; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .marker-line { width: 1px; height: 15px; background: #3b82f6; margin-top: 2px; }

        /* Specs Grid Tables */
        .specs-tables-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-top: 15px; }
        .spec-table { width: 100%; border-collapse: collapse; background: white; font-size: 0.85rem; border: 1px solid #e5e7eb; }
        .spec-table th, .spec-table td { border: 1px solid #e5e7eb; padding: 6px 10px; text-align: left; }
        .spec-table th { background: #f3f4f6; font-weight: 700; color: #374151; text-transform: uppercase; font-size: 0.75rem; }

        .hidden-upload { display: none; }
        .footer-note { font-size: 0.8rem; color: var(--text-muted); text-align: center; margin-top: 20px; grid-column: span 2; }
    </style>
</head>
<body>

    <header>
        <h1>Murph's Custom Bowstrings — Workshop Blueprint Manager</h1>
        <div style="display: flex; gap: 10px;">
            <input type="file" id="uploadConfig" class="hidden-upload" accept=".json" onchange="loadLibraryFile(event)">
            <button class="secondary" style="width: auto; padding: 8px 15px;" onclick="triggerUpload()">Import Library File</button>
            <button class="success" style="width: auto; padding: 8px 15px;" onclick="exportLibraryFile()">Export Master Database</button>
        </div>
    </header>

    <main class="layout-grid">
        <!-- Input Dashboard Panel -->
        <div class="panel">
            <div class="form-group">
                <label for="brandSelect">1. Select Bow Make</label>
                <select id="brandSelect" onchange="populateYearDropdown()">
                    <option value="">-- Choose Brand --</option>
                </select>
            </div>

            <div class="form-group">
                <label for="yearSelect">2. Model Production Year</label>
                <select id="yearSelect" onchange="populateModelDropdown()" disabled>
                    <option value="">-- Choose Year --</option>
                </select>
            </div>

            <div class="form-group">
                <label for="modelSelect">3. Specific Model Blueprint</label>
                <select id="modelSelect" onchange="loadTargetModel()" disabled>
                    <option value="">-- Choose Model --</option>
                </select>
            </div>

            <hr style="border-color: var(--border-color); margin: 20px 0;">
            <h3 style="font-size: 0.9rem; text-transform: uppercase; margin-bottom: 10px; color: var(--text-muted);">Add / Edit Custom Model Specs</h3>
            
            <div class="form-group"><label>Brand Name</label><input type="text" id="customMake" placeholder="e.g., Mathews"></div>
            <div class="form-group"><label>Model Year</label><input type="text" id="customYear" placeholder="e.g., 2026"></div>
            <div class="form-group"><label>Model Name</label><input type="text" id="customName" placeholder="e.g., Phase4 29"></div>
            <div class="form-group"><label>String Total Length</label><input type="text" id="customStrLen" placeholder="e.g., 61 1/2"></div>
            <div class="form-group"><label>Cable Length</label><input type="text" id="customCabLen" placeholder="e.g., 34 1/4"></div>
            <div class="form-group"><label>Yoke Length (If Applicable)</label><input type="text" id="customYokeLen" placeholder="e.g., 12 3/4"></div>
            <div class="form-group"><label>Material Profile</label><input type="text" id="customMat" placeholder="e.g., BCY 452X"></div>
            <div class="form-group"><label>Base Strand Count</label><input type="number" id="customStrands" placeholder="e.g., 24"></div>

            <button onclick="addNewBowToDatabase()">Save Layout to Library</button>
        </div>

        <!-- Render Blueprint Workspace Canvas -->
        <div class="canvas-container">
            <div class="blueprint" id="blueprintPrintArea">
                <div class="blueprint-title" id="canvasTitle">MATHEWS Z7 XTREME BLUEPRINT</div>
                
                <!-- Main Cam String -->
                <div class="string-track">
                    <span class="track-label">Main Cam String (Top End &rarr; Bottom End)</span>
                    <div class="base-line">
                        <div class="loop-end loop-left"></div>
                        <!-- Serving Overlays Mapped Out -->
                        <div class="serving-segment green" style="left: 0; width: 18%;"></div>
                        <div class="serving-segment red" style="left: 45%; width: 12%;"></div>
                        <div class="serving-segment green" style="right: 0; width: 22%;"></div>
                        <div class="loop-end loop-right"></div>
                    </div>
                    <!-- Markers -->
                    <div class="marker" style="left: 18%;"><div class="marker-box">5"</div><div class="marker-line"></div></div>
                    <div class="marker" style="left: 45%;"><div class="marker-box">41 1/4" (Peep)</div><div class="marker-line"></div></div>
                    <div class="marker" style="right: 22%; transform: translateX(50%);"><div class="marker-box">57"</div><div class="marker-line"></div></div>
                </div>

                <!-- Cable 1 Track -->
                <div class="string-track">
                    <span class="track-label">Cable / Yoke Configuration Track</span>
                    <div class="base-line">
                        <div class="loop-end loop-left"></div>
                        <div class="s
