---
name: image-to-interactive-html
description: Convert an uploaded UI image or image file path into a single-file interactive HTML prototype where every visual element can be freely dragged and repositioned on screen.
---

# Image to Interactive HTML Prototype

Convert a reference UI image into a self-contained, single-file HTML prototype with full free-form drag-and-drop interactivity for every visual element.

## 1. Strict Input Validation (Guardrail)

- **Allowed Inputs:** An uploaded image attachment OR a valid local file path pointing to an image file (`.png`, `.jpg`, `.jpeg`, `.webp`, `.gif`).
- **Missing or Invalid Input:** If the user provides only text instructions without an image attachment or a valid image file path, **STOP IMMEDIATELY**. Do NOT hallucinate, guess, or invent a UI layout. Reply concisely asking the user to attach a reference image or provide a valid image file path first.

## 2. Visual Extraction & Fidelity

When a valid image is provided, inspect it carefully and replicate its visual design as closely as possible:
- **Layout & Container:** Match the viewport background color, card dimensions, padding, border-radius, and drop shadows.
- **Typography:** Extract all visible text verbatim (preserving exact language, casing, line breaks, and hierarchy). Load appropriate web fonts (e.g., Google Fonts `Prompt` or `Sarabun` for Thai, `Inter` for English) to match the visual weight and style.
- **Colors & Components:** Match exact hex/RGB colors for backgrounds, borders, badges, headings, body text, and buttons. Recreate icons using clean inline SVGs.

## 3. Interactive Drag-and-Drop Mechanics

Implement smooth, dependency-free Vanilla JavaScript pointer/touch dragging so every individual UI element can be moved anywhere on the canvas:
- **Draggable Targets:** Mark every distinct UI element (cards, back buttons, tags/badges, headings, paragraphs, action buttons, icons, and images) with a `.draggable-item` class.
- **Independent Movement:** Ensure dragging a child element (e.g., a button inside a card) moves only that element without dragging its parent container (`e.stopPropagation()` on `pointerdown`), while dragging the parent container itself moves the container.
- **Pointer & Touch Support:** Use `pointerdown`, `pointermove`, and `pointerup` events with `setPointerCapture` so dragging works seamlessly on both desktop mice and touchscreens.
- **Visual Feedback:** Show `cursor: grab` on hover, `cursor: grabbing` with a subtle elevation shadow and elevated `z-index` while dragging, and prevent accidental text selection (`user-select: none`).

## 4. Built-in Floating Toolbar

Include a minimal, non-intrusive floating control bar in the top-right corner containing:
- **Live Coordinates:** Displays the active element's current `(X, Y)` offset while dragging.
- **Reset Layout Button:** Smoothly animates all elements back to their original `(0, 0)` positions.

## 5. Technical Constraints & Output

- **Single-File Architecture:** Output everything in a single, self-contained `.html` file using embedded `<style>` and `<script>` tags (Tailwind CSS via CDN is permitted, or pure modern CSS). No external build steps or local asset dependencies.
- **Clean Delivery:** Provide the complete HTML code (or write it directly to `index.html` if file system tools are available) followed by a brief summary of the extracted elements.

## Reference Implementation Pattern (Drag Engine & Floating Toolbar)

Use this battle-tested CSS + Vanilla JS structure inside the generated `.html` file to guarantee smooth nested dragging and coordinate tracking:

```html
<style>
  .draggable-item {
    cursor: grab;
    user-select: none;
    touch-action: none;
    position: relative;
    will-change: transform;
  }
  .draggable-item.is-dragging {
    cursor: grabbing;
    z-index: 9999 !important;
    box-shadow: 0 12px 28px rgba(0, 0, 0, 0.18), 0 0 0 2px rgba(59, 130, 246, 0.6);
    transition: none !important;
  }
  .draggable-item.is-resetting {
    transition: transform 0.35s cubic-bezier(0.22, 1, 0.36, 1) !important;
  }
</style>

<!-- Floating Toolbar -->
<div id="floating-toolbar" style="position:fixed;top:16px;right:16px;z-index:10000;display:flex;align-items:center;gap:12px;background:rgba(15,23,42,0.88);backdrop-filter:blur(8px);color:#f8fafc;padding:8px 14px;border-radius:9999px;font-family:monospace;font-size:13px;box-shadow:0 4px 16px rgba(0,0,0,0.2);">
  <span id="coord-readout">X: 0, Y: 0</span>
  <button id="reset-layout-btn" style="background:#3b82f6;color:#fff;border:none;padding:4px 12px;border-radius:9999px;cursor:pointer;font-family:inherit;font-size:12px;font-weight:600;">Reset Layout</button>
</div>

<script>
  (() => {
    const coordReadout = document.getElementById('coord-readout');
    const resetBtn = document.getElementById('reset-layout-btn');
    let highestZ = 100;

    document.querySelectorAll('.draggable-item').forEach((el) => {
      el.dataset.x = el.dataset.x || '0';
      el.dataset.y = el.dataset.y || '0';

      let startX = 0, startY = 0, initialX = 0, initialY = 0;

      el.addEventListener('pointerdown', (e) => {
        if (e.button !== 0 && e.pointerType === 'mouse') return;
        e.stopPropagation(); // Prevent parent .draggable-item from moving when dragging a child
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
        if (!el.hasPointerCapture(e.pointerId)) return;
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

    resetBtn.addEventListener('click', () => {
      document.querySelectorAll('.draggable-item').forEach((el) => {
        el.classList.add('is-resetting');
        el.dataset.x = '0';
        el.dataset.y = '0';
        el.style.transform = 'translate3d(0px, 0px, 0)';
        el.style.zIndex = '';
      });
      coordReadout.textContent = 'X: 0, Y: 0';
    });
  })();
</script>
```
