---
name: image-to-interactive-html
description: Convert an uploaded UI image or image file path into a single-file interactive HTML prototype where every visual element can be freely dragged, edited inline (text), recolored (paint bucket), and annotated (pencil & eraser).
---

# Image to Interactive HTML Prototype (Interactive UI Studio)

Convert a reference UI image into a self-contained, single-file HTML prototype with full free-form drag-and-drop interactivity, inline text editing, paint-bucket color filling, and freehand canvas drawing (pencil & eraser).

## 1. Strict Input Validation (Guardrail)

- **Allowed Inputs:** An uploaded image attachment OR a valid local file path pointing to an image file (`.png`, `.jpg`, `.jpeg`, `.webp`, `.gif`).
- **Missing or Invalid Input:** If the user provides only text instructions without an image attachment or a valid image file path, **STOP IMMEDIATELY**. Do NOT hallucinate, guess, or invent a UI layout. Reply concisely asking the user to attach a reference image or provide a valid image file path first.

## 2. Visual Extraction & Fidelity

When a valid image is provided, inspect it carefully and replicate its visual design as closely as possible:
- **Layout & Container:** Match the viewport background color, card dimensions, padding, border-radius, and drop shadows.
- **Typography:** Extract all visible text verbatim (preserving exact language, casing, line breaks, and hierarchy). Load appropriate web fonts (e.g., Google Fonts `Prompt` or `Sarabun` for Thai, `Inter` for English) to match the visual weight and style.
- **Colors & Components:** Match exact hex/RGB colors for backgrounds, borders, badges, headings, body text, and buttons. Recreate icons using clean inline SVGs.

## 3. Interactive Multi-Tool Studio Mechanics

Implement smooth, dependency-free Vanilla JavaScript supporting **5 interactive studio modes**:
1. **Move (`🖐️ Move` - Drag & Drop):**
   - Mark every distinct UI element (cards, back buttons, tags/badges, headings, paragraphs, action buttons, icons, and images) with a `.draggable-item` class.
   - Ensure dragging a child element moves only that element without dragging its parent container (`e.stopPropagation()` on `pointerdown`).
   - Use `pointerdown`, `pointermove`, and `pointerup` with `setPointerCapture` and hardware-accelerated `translate3d(x, y, 0)`.
2. **Edit Text (`🔤 Text` - แก้ไขข้อความ):**
   - When active, clicking any text/UI element enables `contenteditable="true"` and focuses it so the user can rewrite headings, labels, badges, or paragraphs directly in place (also allow double-clicking in Move mode to quick-edit text).
3. **Paint Bucket (`🪣 Fill` - ลงถังสี):**
   - When active, clicking any `.draggable-item` (or the page background) fills its `background` / `backgroundColor` with the active color from the toolbar's color picker. Holding `Shift` while clicking changes the element's text `color` instead.
4. **Pencil (`✏️ Pencil` - วาดดินสอ):**
   - Activates a full-viewport transparent HTML5 `<canvas>` overlay above the UI so the user can sketch freehand lines, arrows, or notes using the selected color and brush thickness.
5. **Eraser (`🧹 Eraser` - ยางลบ):**
   - Uses `globalCompositeOperation = 'destination-out'` on the annotation canvas overlay so dragging rubs out drawn pencil strokes cleanly.

## 4. Built-in Floating Studio Toolbar

Include a minimal, non-intrusive floating control bar in the top-right corner containing:
- **Tool Selector Buttons:** `Move`, `Text`, `Fill`, `Pencil`, `Eraser` with clear active state styling.
- **Color Picker & Brush Size Slider:** Shared color picker (`<input type="color">`) for Paint Bucket & Pencil, plus a stroke size slider (`<input type="range">`) for Pencil & Eraser.
- **Live Coordinates:** Displays the active element's current `(X, Y)` offset while dragging.
- **Reset Layout Button:** Smoothly animates all elements back to `(0, 0)`, restores original colors and original text content, and clears the drawing canvas.

## 5. Technical Constraints & Output

- **Single-File Architecture:** Output everything in a single, self-contained `.html` file using embedded `<style>` and `<script>` tags (Tailwind CSS via CDN is permitted, or pure modern CSS). No external build steps or local asset dependencies.
- **Clean Delivery:** Provide the complete HTML code (or write it directly to `index.html` if file system tools are available) followed by a brief summary of the extracted elements and interactive tools.

## Reference Implementation Pattern (Multi-Tool Engine & Floating Studio Toolbar)

Use this battle-tested CSS + HTML + Vanilla JS structure inside the generated `.html` file:

```html
<style>
  .draggable-item {
    cursor: grab;
    user-select: none;
    touch-action: none;
    position: relative;
    will-change: transform;
  }
  body[data-tool="move"] .draggable-item:hover {
    outline: 1px dashed rgba(56, 189, 248, 0.5);
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
    outline: 2px dashed #f59e0b;
    outline-offset: 3px;
  }
  .draggable-item.is-dragging {
    cursor: grabbing !important;
    z-index: 9999 !important;
    box-shadow: 0 12px 28px rgba(0, 0, 0, 0.22), 0 0 0 2px rgba(59, 130, 246, 0.7);
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
</style>

<!-- Drawing Overlay Canvas -->
<canvas id="annotation-canvas"></canvas>

<!-- Floating Studio Toolbar -->
<div id="floating-toolbar" style="position:fixed;top:16px;right:16px;z-index:10000;display:flex;flex-wrap:wrap;align-items:center;gap:8px;background:rgba(15,23,42,0.92);backdrop-filter:blur(10px);color:#f8fafc;padding:8px 14px;border-radius:9999px;font-family:system-ui,-apple-system,sans-serif;font-size:12px;border:1px solid rgba(255,255,255,0.14);box-shadow:0 8px 24px rgba(0,0,0,0.35);">
  <button class="studio-btn active" data-set-tool="move" title="Move / Drag UI Elements">🖐️ Move</button>
  <button class="studio-btn" data-set-tool="text" title="Click any text to edit inline">🔤 Text</button>
  <button class="studio-btn" data-set-tool="fill" title="Click element to fill background (Shift+Click for text color)">🪣 Fill</button>
  <button class="studio-btn" data-set-tool="pencil" title="Freehand pencil drawing">✏️ Pencil</button>
  <button class="studio-btn" data-set-tool="eraser" title="Erase pencil drawings">🧹 Eraser</button>

  <span style="width:1px;height:18px;background:rgba(255,255,255,0.15);margin:0 2px;"></span>

  <input type="color" id="studio-color" value="#3b82f6" title="Color for Fill & Pencil" style="width:26px;height:26px;border:none;border-radius:50%;background:transparent;cursor:pointer;padding:0;" />
  <input type="range" id="studio-size" min="2" max="36" value="4" title="Pencil / Eraser Size" style="width:64px;accent-color:#3b82f6;cursor:pointer;" />

  <span style="width:1px;height:18px;background:rgba(255,255,255,0.15);margin:0 2px;"></span>

  <span id="coord-readout" style="font-family:monospace;min-width:82px;text-align:center;">X: 0, Y: 0</span>
  <button id="reset-layout-btn" style="background:#ef4444;color:#fff;border:none;padding:5px 12px;border-radius:9999px;cursor:pointer;font-size:12px;font-weight:600;">Reset</button>
</div>

<script>
  (() => {
    const coordReadout = document.getElementById('coord-readout');
    const resetBtn = document.getElementById('reset-layout-btn');
    const colorInput = document.getElementById('studio-color');
    const sizeInput = document.getElementById('studio-size');
    const toolButtons = document.querySelectorAll('[data-set-tool]');
    const canvas = document.getElementById('annotation-canvas');
    const ctx = canvas.getContext('2d');

    let currentTool = 'move';
    let highestZ = 100;
    const origBg = document.body.style.background;
    const origBgColor = document.body.style.backgroundColor;

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

      // Double-click in Move mode opens quick inline text edit
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

        // Paint Bucket Mode
        if (currentTool === 'fill') {
          e.stopPropagation();
          e.preventDefault();
          if (e.shiftKey) {
            el.style.color = colorInput.value;
          } else {
            el.style.background = colorInput.value;
            el.style.backgroundColor = colorInput.value;
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
        document.body.style.background = colorInput.value;
        document.body.style.backgroundColor = colorInput.value;
      }
    });

    // Freehand Pencil & Eraser on Canvas Overlay
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
        ctx.fillStyle = colorInput.value;
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
        ctx.strokeStyle = colorInput.value;
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

    // Reset All (Positions, Colors, Edited Text, and Canvas Drawings)
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
