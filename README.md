# 🎮 Top-Down 2D Level Editor (JSX)

A visual collision editor for creating top-down game levels with image-based backgrounds. Built with React, zero dependencies.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![React](https://img.shields.io/badge/React-18+-61DAFB?logo=react&logoColor=white)](https://reactjs.org/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🖼️ **Background Layers** | Upload PNG/JPG images as level backgrounds |
| 📦 **Block Tool** | Place, move, scale, and rotate rectangular collision boxes |
| 📏 **Wall Tool** | Create point-to-point line segments with adjustable thickness |
| ✏️ **Pen Tool** | Trace irregular map boundaries with straight-line polygons |
| 🔄 **Invert Polygons** | Block everything *outside* a traced area (perfect for map boundaries) |
| 🎮 **Play Test Mode** | Test your collision layout with WASD movement |
| 📤 **Export/Import** | Save and load levels as JSON files |
| 🧲 **Grid Snapping** | Optional snap-to-grid for precise placement |

---

## 🚀 Quick Start

### Option 1: Try the Demo (No Setup Required)

Download and open [`demo.html`](./demo.html) in your browser. That's it!

```bash
# Clone the repo
git clone https://github.com/MushroomFleet/topdown2d-leveleditor-JSX.git

# Open the demo
open demo.html
# or on Windows
start demo.html
```

### Option 2: Add to Your React Project

```bash
# Copy the component to your project
cp level-editor.jsx your-project/src/components/LevelEditor.jsx
```

```jsx
import LevelEditor from './components/LevelEditor';

function App() {
  return <LevelEditor />;
}
```

---

## 🎯 Use Cases

- **2D Top-Down Games** - RPGs, adventure games, shooters
- **Visual Novel Scenes** - Define walkable areas on backgrounds
- **Tile-less Level Design** - Use hand-drawn or AI-generated backgrounds
- **Rapid Prototyping** - Quickly test collision layouts before final art

---

## 🛠️ Tools Overview

### ◼ Block Tool
Rectangular collision boxes ideal for:
- Furniture and props
- Doors and obstacles
- Any axis-aligned or rotated rectangular object

### ╱ Wall Tool
Line segments with thickness, perfect for:
- Thin walls and barriers
- Corridors and pathways
- Angled obstacles

### ✏️ Pen Tool
Polygon tracing for:
- Irregular map boundaries
- Complex room shapes
- Natural terrain edges

> **Pro Tip:** After tracing a map outline with the Pen Tool, select it and enable **"Invert (Block Outside)"** to confine the player within the traced area.

---

## 📁 Project Structure

```
topdown2d-leveleditor-JSX/
├── README.md                           # This file
├── level-editor.jsx                    # Main React component
├── demo.html                           # Standalone browser demo
└── 2dblockingeditor-integration.md     # Integration guide
```

---

## 📋 Level Data Format

Exported levels use this JSON structure:

```json
{
  "blocks": [
    { "id": 1234, "x": 100, "y": 150, "width": 50, "height": 50, "rotation": 45 }
  ],
  "walls": [
    { "id": 1235, "x1": 0, "y1": 0, "x2": 200, "y2": 100, "thickness": 10 }
  ],
  "polygons": [
    { "id": 1236, "points": [{"x": 50, "y": 50}, ...], "inverted": true }
  ],
  "playerStart": { "x": 100, "y": 100 },
  "backgroundImage": "data:image/png;base64,..."
}
```

---

## 🔧 Integration Guide

For detailed integration into new or existing React games, see:

📖 **[2dblockingeditor-integration.md](./2dblockingeditor-integration.md)**

This guide includes:
- Complete `CollisionSystem.js` class (copy-paste ready)
- Game loop integration examples
- Debug visualization overlay
- Performance optimization tips
- Editor/Game mode switching patterns

---

## ⌨️ Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `W` `A` `S` `D` | Move player (Play mode) |
| `Delete` / `Backspace` | Remove selected object |
| `Escape` | Cancel current drawing |

---

## 🎨 Collision Types Comparison

| Type | Shape | Best For | Rotation |
|------|-------|----------|----------|
| **Block** | Rectangle | Props, furniture, doors | ✅ Yes |
| **Wall** | Thick line | Barriers, corridors | N/A |
| **Polygon** | Multi-point | Map boundaries, complex shapes | ❌ No |
| **Polygon (Inverted)** | Exclusion zone | Playable area bounds | ❌ No |

---

## 🖥️ Browser Support

| Browser | Support |
|---------|---------|
| Chrome | ✅ Full |
| Firefox | ✅ Full |
| Safari | ✅ Full |
| Edge | ✅ Full |

---

## 🤝 Contributing

Contributions are welcome! Feel free to:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🗺️ Roadmap

- [ ] Undo/Redo support
- [ ] Vertex editing for polygons
- [ ] Multiple layers with z-ordering
- [ ] Trigger zones (non-blocking event areas)
- [ ] Tilemap integration option
- [ ] Custom collision shapes (circles, ellipses)

---

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{topdown2d_leveleditor_jsx,
  title = {Top-Down 2D Level Editor: A visual collision editor for React-based games},
  author = {Drift Johnson},
  year = {2025},
  url = {https://github.com/MushroomFleet/topdown2d-leveleditor-JSX},
  version = {1.0.0}
}
```

### Donate

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)

---

<p align="center">
  Made with ❤️ for game developers
</p>
