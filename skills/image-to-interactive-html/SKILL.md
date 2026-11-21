---
name: image-to-interactive-html
description: Convert an uploaded UI image or image file path into a single-file interactive HTML prototype where every visual element can be freely dragged, edited inline (text), recolored (paint bucket), and annotated (pencil & eraser).
---

# Image to Interactive HTML Prototype (Interactive UI Studio)

Convert a reference UI image into a self-contained, single-file HTML prototype with full free-form drag-and-drop interactivity, inline text editing, adjustable paint-bucket color filling, and freehand canvas drawing (pencil & eraser) sharing a unified color palette.

## 1. Strict Input Validation (Guardrail)

- **Allowed Inputs:** An uploaded image attachment OR a valid local file path pointing to an image file (`.png`, `.jpg`, `.jpeg`, `.webp`, `.gif`).
- **Missing or Invalid Input:** If the user provides only text instructions without an image attachment or a valid image file path, **STOP IMMEDIATELY**. Do NOT hallucinate, guess, or invent a UI layout. Reply concisely asking the user to attach a reference image or provide a valid image file path first.

## 2. Visual Extraction & Fidelity

When a valid image is provided, inspect it carefully and replicate its visual design as closely as possible:
- **Layout & Container:** Match the viewport background color, card dimensions, padding, border-radius, and drop shadows.
- **Typography:** Extract all visible text verbatim (preserving exact language, casing, line breaks, and hierarchy). Load appropriate web fonts (e.g., Google Fonts `Prompt` or `Sarabun` for Thai, `Inter` for English) to match the visual weight and style.
- **Colors & Components:** Match exact hex/RGB colors for backgrounds, borders, badges, headings, body text, and buttons. Recreate icons using clean inline SVGs.

## 3. Interactive Multi-Tool Studio Mechanics

Implement smooth, dependency-free Vanilla JavaScript supporting **5 interactive studio modes** and a **Unified Color System**:
1. **Move (`🖐️ Move` - Drag & Drop):**
   - Mark every distinct UI element (cards, back buttons, tags/badges, headings, paragraphs, action buttons, icons, and images) with a `.draggable-item` class.
   - Ensure dragging a child element moves only that element without dragging its parent container (`e.stopPropagation()` on `pointerdown`).
   - Use `pointerdown`, `pointermove`, and `pointerup` with `setPointerCapture` and hardware-accelerated `translate3d(x, y, 0)`.
2. **Edit Text (`🔤 Text` - แก้ไขข้อความ):**
   - When active, clicking any text/UI element enables `contenteditable="true"` and focuses it so the user can rewrite headings, labels, badges, or paragraphs directly in place (also allow double-clicking in Move mode to quick-edit text).
3. **Paint Bucket (`🪣 Fill` - ลงถังสีปรับสีได้):**
   - Uses the **Shared Color System** (Quick Color Swatches + Custom RGB/Hex Color Picker Wheel).
   - When active, clicking any `.draggable-item` (or the page background) applies the selected color immediately (overriding both `background` gradients and `backgroundColor`).
   - Includes a `Bg / Text` target toggle (or `Shift + Click`) so the user can fill either **Background Color** or **Text/Icon Color**.
4. **Pencil (`✏️ Pencil` - วาดดินสอ):**
   - Shares the exact same active color from the **Shared Color System** as the Paint Bucket.
   - Activates a full-viewport transparent HTML5 `<canvas>` overlay above the UI so the user can sketch freehand lines, arrows, or notes with adjustable brush thickness.
5. **Eraser (`🧹 Eraser` - ยางลบ):**
   - Uses `globalCompositeOperation = 'destination-out'` on the annotation canvas overlay so dragging rubs out drawn pencil strokes cleanly.

## 4. Built-in Floating Studio Toolbar

Include a minimal, non-intrusive floating control bar in the top-right corner containing:
- **Tool Selector Buttons:** `Move`, `Text`, `Fill`, `Pencil`, `Eraser` with clear active state styling.
- **Unified Color Picker & Preset Swatches (Shared by Paint Bucket & Pencil):**
  - Quick color swatch buttons for instant 1-click color selection.
  - A custom color picker (`<input type="color">`) with a live color preview dot and hex readout (`#3B82F6`) that syncs with both the Paint Bucket (`🪣 Fill`) and Pencil (`✏️ Pencil`).
  - A Fill Target sub-toggle (`Bg` vs `Text`) so users can easily recolor backgrounds or text.
- **Brush Size Slider:** Stroke thickness slider (`<input type="range">`) for Pencil & Eraser.
- **Live Coordinates & Reset Button:** Displays `(X, Y)` offset while dragging, plus a `Reset` button that restores all positions to `(0, 0)`, original colors, original text, and clears the drawing canvas.

## 5. Technical Constraints & Output

- **Single-File Architecture:** Output everything in a single, self-contained `.html` file using embedded `<style>` and `<script>` tags (Tailwind CSS via CDN is permitted, or pure modern CSS). No external build steps or local asset dependencies.
- **Clean Delivery:** Write the complete HTML code directly to a `.html` file on disk (e.g., `index.html` or `interactive-ui.html`), then automatically launch it in the built-in Browser AI pane (Section 6) followed by a brief summary of the extracted elements and interactive tools.

## 6. Auto-Launch Live Preview in Built-in Browser AI (`browser` Subagent)

Immediately after writing the generated `.html` file to disk, **automatically pop open the built-in Browser AI pane** so the user can interact with the UI right away without opening files manually:

1. **Start a Background Local Server (`IsDaemon: true`):**
   - Run a lightweight local HTTP server in the directory containing the generated `.html` file using `run_command` with `IsDaemon: true` and `WaitMsBeforeAsync: 1000` (for example: `python -m http.server 8765` or `npx --yes serve -l 8765`).
2. **Invoke the Built-in `browser` Subagent (`invoke_subagent`):**
   - Immediately call `invoke_subagent` with:
     - `TypeName: "browser"`
     - `Role: "Interactive UI Previewer"`
     - `Model: "flash"`
     - `Prompt`: Instruct the browser subagent to navigate to `http://localhost:8765/<filename>.html`, verify that the UI card/layout and the top-right `#floating-toolbar` (`Move`, `Text`, `Fill`, `Pencil`, `Eraser`) are visible and rendered properly, and leave the tab open for the user to interact with.
3. **Fallback (Non-Antigravity Environments):**
   - Only if the `invoke_subagent` tool is unavailable in the host agent environment, open the generated `.html` file using the OS default browser.

## Reference Implementation Pattern (Multi-Tool Engine & Unified Color System)

Use this battle-tested CSS + HTML + Vanilla JS structure inside the generated `.html` file:

```html
<style>
  :root {
    --active-studio-color: #3b82f6;
  }
  .draggable-item {
    cursor: grab;
    user-select: none;
    touch-action: none;
    position: relative;
    will-change: transform;
  }
  body[data-tool="move"] .draggable-item:hover {
    outline: 1px dashed rgba(56, 189, 248, 0.55);
    outline-offset: 3px;
  }
  body[data-tool="text"] .draggable-item {
    cursor: text;
    user-select: text;
  }
  body[data-tool="text"] .draggable-item:hover,
  .draggable-item[contenteditable="true"]:focus {
    outline: 2px solid #a855f7;
    outline-offset: 3px;
  }
  body[data-tool="fill"] .draggable-item {
    cursor: crosshair;
  }
  body[data-tool="fill"] .draggable-item:hover {
    outline: 2px dashed var(--active-studio-color);
    outline-offset: 3px;
    box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.15);
  }
  .draggable-item.is-dragging {
    cursor: grabbing !important;
    z-index: 9999 !important;
    box-shadow: 0 12px 28px rgba(0, 0, 0, 0.25), 0 0 0 2px rgba(59, 130, 246, 0.7);
    transition: none !important;
  }
  .draggable-item.is-resetting {
    transition: transform 0.35s cubic-bezier(0.22, 1, 0.36, 1), background-color 0.25s, color 0.25s !important;
  }
  #annotation-canvas {
    position: fixed;
    inset: 0;
    width: 100vw;
    height: 100vh;
    z-index: 9990;
    pointer-events: none;
    touch-action: none;
  }
  body[data-tool="pencil"] #annotation-canvas,
  body[data-tool="eraser"] #annotation-canvas {
    pointer-events: auto;
    cursor: crosshair;
  }
  .studio-btn {
    background: rgba(255, 255, 255, 0.08);
    color: #cbd5e1;
    border: 1px solid rgba(255, 255, 255, 0.1);
    padding: 5px 10px;
    border-radius: 9999px;
    cursor: pointer;
    font-size: 12px;
    font-weight: 600;
    transition: all 0.15s ease;
    display: inline-flex;
    align-items: center;
    gap: 4px;
  }
  .studio-btn:hover {
    background: rgba(255, 255, 255, 0.16);
    color: #fff;
  }
  .studio-btn.active {
    background: #3b82f6;
    color: #fff;
    border-color: #60a5fa;
  }
  .color-swatch {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    border: 2px solid rgba(255, 255, 255, 0.25);
    cursor: pointer;
    padding: 0;
    transition: transform 0.12s ease, border-color 0.12s ease;
  }
  .color-swatch:hover {
    transform: scale(1.18);
  }
  .color-swatch.active {
    border-color: #ffffff;
    box-shadow: 0 0 0 2px #3b82f6;
    transform: scale(1.15);
  }
  .color-picker-wrap {
    position: relative;
    display: inline-flex;
    align-items: center;
    gap: 6px;
    background: rgba(255, 255, 255, 0.08);
    border: 1px solid rgba(255, 255, 255, 0.15);
    padding: 3px 8px 3px 4px;
    border-radius: 9999px;
    cursor: pointer;
  }
  .color-picker-wrap:hover {
    background: rgba(255, 255, 255, 0.14);
  }
  .color-preview-dot {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: var(--active-studio-color);
    border: 2px solid #fff;
    box-shadow: 0 1px 4px rgba(0,0,0,0.3);
  }
  #studio-color {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
    cursor: pointer;
  }
</style>

<!-- Drawing Overlay Canvas -->
<canvas id="annotation-canvas"></canvas>

<!-- Floating Studio Toolbar -->
<div id="floating-toolbar" style="position:fixed;top:16px;right:16px;z-index:10000;display:flex;flex-wrap:wrap;align-items:center;gap:6px;background:rgba(15,23,42,0.94);backdrop-filter:blur(12px);color:#f8fafc;padding:8px 14px;border-radius:9999px;font-family:system-ui,-apple-system,sans-serif;font-size:12px;border:1px solid rgba(255,255,255,0.14);box-shadow:0 10px 28px rgba(0,0,0,0.4);">
  <!-- 5 Core Tools -->
  <button class="studio-btn active" data-set-tool="move" title="Move / Drag UI Elements">🖐️ Move</button>
  <button class="studio-btn" data-set-tool="text" title="Click any text to edit inline">🔤 Text</button>
  <button class="studio-btn" data-set-tool="fill" title="Paint Bucket: Click any element to fill with the selected color">🪣 Fill</button>
  <button class="studio-btn" data-set-tool="pencil" title="Freehand Pencil: Draw with the selected color">✏️ Pencil</button>
  <button class="studio-btn" data-set-tool="eraser" title="Eraser: Rub out pencil strokes">🧹 Eraser</button>

  <!-- Fill Target Toggle (Background vs Text) -->
  <button id="fill-target-btn" class="studio-btn" title="Toggle Paint Bucket target: Background Color vs Text Color" style="background:rgba(59,130,246,0.18);border-color:rgba(96,165,250,0.4);color:#93c5fd;">🎯 เปลี่ยน: พื้นหลัง (Bg)</button>

  <span style="width:1px;height:18px;background:rgba(255,255,255,0.15);margin:0 2px;"></span>

  <!-- Shared Color System (Used by BOTH Paint Bucket & Pencil) -->
  <div style="display:inline-flex;align-items:center;gap:5px;" title="Shared Color Palette for Paint Bucket & Pencil">
    <button type="button" class="color-swatch active" data-color="#3b82f6" style="background:#3b82f6;" title="Blue"></button>
    <button type="button" class="color-swatch" data-color="#10b981" style="background:#10b981;" title="Emerald"></button>
    <button type="button" class="color-swatch" data-color="#f59e0b" style="background:#f59e0b;" title="Amber"></button>
    <button type="button" class="color-swatch" data-color="#ef4444" style="background:#ef4444;" title="Red"></button>
    <button type="button" class="color-swatch" data-color="#a855f7" style="background:#a855f7;" title="Purple"></button>
    <button type="button" class="color-swatch" data-color="#ec4899" style="background:#ec4899;" title="Pink"></button>
    <button type="button" class="color-swatch" data-color="#1e293b" style="background:#1e293b;" title="Dark Slate"></button>
    <button type="button" class="color-swatch" data-color="#ffffff" style="background:#ffffff;" title="White"></button>

    <label class="color-picker-wrap" title="Click to choose any custom color for Paint Bucket & Pencil">
      <span class="color-preview-dot" id="color-preview-dot"></span>
      <span id="color-hex-label" style="font-family:monospace;font-size:11px;color:#e2e8f0;">#3B82F6</span>
      <input type="color" id="studio-color" value="#3b82f6" />
    </label>
  </div>

  <!-- Brush Size Slider -->
  <input type="range" id="studio-size" min="2" max="36" value="4" title="Pencil / Eraser Thickness" style="width:58px;accent-color:#3b82f6;cursor:pointer;" />

  <span style="width:1px;height:18px;background:rgba(255,255,255,0.15);margin:0 2px;"></span>

  <span id="coord-readout" style="font-family:monospace;min-width:78px;text-align:center;">X: 0, Y: 0</span>
  <button id="reset-layout-btn" style="background:#ef4444;color:#fff;border:none;padding:5px 12px;border-radius:9999px;cursor:pointer;font-size:12px;font-weight:600;">Reset</button>
</div>

<script>
  (() => {
    const coordReadout = document.getElementById('coord-readout');
    const resetBtn = document.getElementById('reset-layout-btn');
    const colorInput = document.getElementById('studio-color');
    const colorPreviewDot = document.getElementById('color-preview-dot');
    const colorHexLabel = document.getElementById('color-hex-label');
    const swatches = document.querySelectorAll('.color-swatch');
    const fillTargetBtn = document.getElementById('fill-target-btn');
    const sizeInput = document.getElementById('studio-size');
    const toolButtons = document.querySelectorAll('[data-set-tool]');
    const canvas = document.getElementById('annotation-canvas');
    const ctx = canvas.getContext('2d');

    let currentTool = 'move';
    let currentColor = '#3b82f6';
    let fillTarget = 'bg'; // 'bg' or 'text'
    let highestZ = 100;
    const origBg = document.body.style.background;
    const origBgColor = document.body.style.backgroundColor;

    // Unified Color Updater (Shared between Paint Bucket & Pencil)
    const setSharedColor = (hex) => {
      currentColor = hex;
      colorInput.value = hex;
      document.documentElement.style.setProperty('--active-studio-color', hex);
      colorPreviewDot.style.background = hex;
      colorHexLabel.textContent = hex.toUpperCase();
      swatches.forEach((s) => s.classList.toggle('active', s.dataset.color.toLowerCase() === hex.toLowerCase()));
    };

    colorInput.addEventListener('input', (e) => setSharedColor(e.target.value));
    swatches.forEach((swatch) => {
      swatch.addEventListener('click', () => {
        setSharedColor(swatch.dataset.color);
        // Automatically switch to Fill or keep current drawing/fill tool
        if (currentTool !== 'fill' && currentTool !== 'pencil') {
          setTool('fill');
        }
      });
    });

    // Toggle Paint Bucket Target: Background vs Text Color
    fillTargetBtn.addEventListener('click', () => {
      fillTarget = fillTarget === 'bg' ? 'text' : 'bg';
      fillTargetBtn.textContent = fillTarget === 'bg' ? '🎯 เปลี่ยน: พื้นหลัง (Bg)' : '🎯 เปลี่ยน: ตัวอักษร (Text)';
      setTool('fill');
    });

    // Resize canvas while preserving drawing
    const resizeCanvas = () => {
      const temp = document.createElement('canvas');
      temp.width = canvas.width;
      temp.height = canvas.height;
      if (canvas.width && canvas.height) temp.getContext('2d').drawImage(canvas, 0, 0);
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      if (temp.width && temp.height) ctx.drawImage(temp, 0, 0);
    };
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // Tool switching
    const setTool = (tool) => {
      currentTool = tool;
      document.body.dataset.tool = tool;
      toolButtons.forEach((btn) => btn.classList.toggle('active', btn.dataset.setTool === tool));
      if (tool !== 'text') {
        document.querySelectorAll('.draggable-item[contenteditable="true"]').forEach((el) => {
          el.removeAttribute('contenteditable');
        });
      }
    };
    setTool('move');
    toolButtons.forEach((btn) => btn.addEventListener('click', () => setTool(btn.dataset.setTool)));

    // Element behaviors: Move, Text Edit, Paint Bucket
    document.querySelectorAll('.draggable-item').forEach((el) => {
      el.dataset.x = el.dataset.x || '0';
      el.dataset.y = el.dataset.y || '0';
      el.dataset.origHtml = el.innerHTML;
      el.dataset.origBg = el.style.background;
      el.dataset.origBgColor = el.style.backgroundColor;
      el.dataset.origColor = el.style.color;

      let startX = 0, startY = 0, initialX = 0, initialY = 0;

      el.addEventListener('dblclick', (e) => {
        if (currentTool === 'move') {
          e.stopPropagation();
          el.setAttribute('contenteditable', 'true');
          el.focus();
        }
      });

      el.addEventListener('blur', () => {
        if (currentTool !== 'text') el.removeAttribute('contenteditable');
      });

      el.addEventListener('pointerdown', (e) => {
        if (e.button !== 0 && e.pointerType === 'mouse') return;

        // Paint Bucket Mode (uses Shared Color)
        if (currentTool === 'fill') {
          e.stopPropagation();
          e.preventDefault();
          el.classList.remove('is-resetting');
          if (e.shiftKey || fillTarget === 'text') {
            el.style.setProperty('color', currentColor, 'important');
          } else {
            el.style.setProperty('background', currentColor, 'important');
            el.style.setProperty('background-color', currentColor, 'important');
          }
          return;
        }

        // Text Edit Mode
        if (currentTool === 'text') {
          e.stopPropagation();
          el.setAttribute('contenteditable', 'true');
          el.focus();
          return;
        }

        // Move Mode
        if (currentTool !== 'move') return;
        if (el.getAttribute('contenteditable') === 'true') return;

        e.stopPropagation();
        el.classList.remove('is-resetting');
        el.classList.add('is-dragging');
        el.style.zIndex = ++highestZ;

        initialX = parseFloat(el.dataset.x) || 0;
        initialY = parseFloat(el.dataset.y) || 0;
        startX = e.clientX;
        startY = e.clientY;

        el.setPointerCapture(e.pointerId);
        coordReadout.textContent = `X: ${Math.round(initialX)}, Y: ${Math.round(initialY)}`;
      });

      el.addEventListener('pointermove', (e) => {
        if (currentTool !== 'move' || !el.hasPointerCapture(e.pointerId)) return;
        const dx = e.clientX - startX;
        const dy = e.clientY - startY;
        const newX = initialX + dx;
        const newY = initialY + dy;

        el.dataset.x = newX;
        el.dataset.y = newY;
        el.style.transform = `translate3d(${newX}px, ${newY}px, 0)`;
        coordReadout.textContent = `X: ${Math.round(newX)}, Y: ${Math.round(newY)}`;
      });

      const endDrag = (e) => {
        if (!el.hasPointerCapture(e.pointerId)) return;
        el.releasePointerCapture(e.pointerId);
        el.classList.remove('is-dragging');
      };

      el.addEventListener('pointerup', endDrag);
      el.addEventListener('pointercancel', endDrag);
    });

    // Allow Paint Bucket on page background
    document.body.addEventListener('pointerdown', (e) => {
      if (currentTool === 'fill' && e.target === document.body) {
        document.body.style.setProperty('background', currentColor, 'important');
        document.body.style.setProperty('background-color', currentColor, 'important');
      }
    });

    // Freehand Pencil & Eraser on Canvas Overlay (uses Shared Color)
    let isDrawing = false;
    let lastX = 0, lastY = 0;

    canvas.addEventListener('pointerdown', (e) => {
      if (currentTool !== 'pencil' && currentTool !== 'eraser') return;
      isDrawing = true;
      lastX = e.clientX;
      lastY = e.clientY;
      canvas.setPointerCapture(e.pointerId);

      ctx.beginPath();
      ctx.arc(lastX, lastY, parseFloat(sizeInput.value) / 2, 0, Math.PI * 2);
      if (currentTool === 'eraser') {
        ctx.globalCompositeOperation = 'destination-out';
        ctx.fill();
      } else {
        ctx.globalCompositeOperation = 'source-over';
        ctx.fillStyle = currentColor;
        ctx.fill();
      }
    });

    canvas.addEventListener('pointermove', (e) => {
      if (!isDrawing) return;
      ctx.beginPath();
      ctx.moveTo(lastX, lastY);
      ctx.lineTo(e.clientX, e.clientY);
      ctx.lineCap = 'round';
      ctx.lineJoin = 'round';
      ctx.lineWidth = parseFloat(sizeInput.value);

      if (currentTool === 'eraser') {
        ctx.globalCompositeOperation = 'destination-out';
      } else {
        ctx.globalCompositeOperation = 'source-over';
        ctx.strokeStyle = currentColor;
      }
      ctx.stroke();
      lastX = e.clientX;
      lastY = e.clientY;
    });

    const stopDrawing = (e) => {
      if (!isDrawing) return;
      isDrawing = false;
      if (canvas.hasPointerCapture(e.pointerId)) canvas.releasePointerCapture(e.pointerId);
    };
    canvas.addEventListener('pointerup', stopDrawing);
    canvas.addEventListener('pointercancel', stopDrawing);

    // Reset All
    resetBtn.addEventListener('click', () => {
      document.querySelectorAll('.draggable-item').forEach((el) => {
        el.classList.add('is-resetting');
        el.dataset.x = '0';
        el.dataset.y = '0';
        el.style.transform = 'translate3d(0px, 0px, 0)';
        el.style.zIndex = '';
        el.style.background = el.dataset.origBg || '';
        el.style.backgroundColor = el.dataset.origBgColor || '';
        el.style.color = el.dataset.origColor || '';
        if (el.dataset.origHtml !== undefined) el.innerHTML = el.dataset.origHtml;
        el.removeAttribute('contenteditable');
      });
      document.body.style.background = origBg;
      document.body.style.backgroundColor = origBgColor;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      coordReadout.textContent = 'X: 0, Y: 0';
    });
  })();
</script>
```
