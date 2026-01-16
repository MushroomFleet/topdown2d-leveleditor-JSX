# 2D Blocking Editor - Integration Plan

A comprehensive guide for integrating the 2D Blocking Editor into new or existing React-based games.

## Overview

The 2D Blocking Editor provides a visual tool for creating collision layouts on top of image-based level backgrounds. It supports three collision primitive types:

| Type | Use Case | Collision Shape |
|------|----------|-----------------|
| **Blocks** | Props, objects, furniture | Rotatable rectangles |
| **Walls** | Barriers, corridors, linear obstacles | Thick line segments |
| **Polygons** | Map boundaries, irregular shapes | Edge collision + optional inverted fill |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    EDITOR MODE                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │ Tool Panel  │  │   Canvas    │  │ Properties Panel│ │
│  │             │  │             │  │                 │ │
│  │ - Select    │  │ Background  │  │ - Position      │ │
│  │ - Place     │  │ + Overlays  │  │ - Size          │ │
│  │ - Wall      │  │ + Player    │  │ - Rotation      │ │
│  │ - Pen       │  │             │  │ - Thickness     │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼ Export JSON
┌─────────────────────────────────────────────────────────┐
│                    RUNTIME MODE                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Collision System                    │   │
│  │  - Load level JSON                              │   │
│  │  - Check player/entity collisions               │   │
│  │  - Optional: Debug visualization                │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Level Data Format

The editor exports/imports JSON with this structure:

```json
{
  "blocks": [
    {
      "id": 1234567890,
      "x": 100,
      "y": 150,
      "width": 50,
      "height": 50,
      "rotation": 45
    }
  ],
  "walls": [
    {
      "id": 1234567891,
      "x1": 0,
      "y1": 0,
      "x2": 200,
      "y2": 100,
      "thickness": 10
    }
  ],
  "polygons": [
    {
      "id": 1234567892,
      "points": [
        { "x": 50, "y": 50 },
        { "x": 200, "y": 50 },
        { "x": 200, "y": 300 },
        { "x": 50, "y": 300 }
      ],
      "inverted": true
    }
  ],
  "playerStart": {
    "x": 100,
    "y": 100
  },
  "backgroundImage": "data:image/png;base64,..."
}
```

---

## Integration Steps

### Step 1: Install Dependencies

The editor uses only React with no external dependencies:

```bash
# For new projects
npx create-react-app my-game
cd my-game

# Or add to existing project - no additional deps needed
```

### Step 2: Add the Editor Component

Copy `level-editor.jsx` to your project:

```
src/
├── components/
│   └── LevelEditor.jsx    # The editor component
├── levels/
│   └── level1.json        # Exported level data
└── game/
    └── CollisionSystem.js # Runtime collision detection
```

### Step 3: Create the Runtime Collision System

Extract the collision logic for use in your game:

```javascript
// CollisionSystem.js

export class CollisionSystem {
  constructor(levelData) {
    this.blocks = levelData.blocks || [];
    this.walls = levelData.walls || [];
    this.polygons = levelData.polygons || [];
  }

  // Point to line segment distance
  pointToLineDistance(px, py, x1, y1, x2, y2) {
    const A = px - x1;
    const B = py - y1;
    const C = x2 - x1;
    const D = y2 - y1;
    
    const dot = A * C + B * D;
    const lenSq = C * C + D * D;
    let param = lenSq !== 0 ? dot / lenSq : -1;
    
    let xx, yy;
    if (param < 0) { xx = x1; yy = y1; }
    else if (param > 1) { xx = x2; yy = y2; }
    else { xx = x1 + param * C; yy = y1 + param * D; }
    
    return Math.sqrt((px - xx) ** 2 + (py - yy) ** 2);
  }

  // Point in polygon (ray casting)
  pointInPolygon(px, py, points) {
    let inside = false;
    for (let i = 0, j = points.length - 1; i < points.length; j = i++) {
      const xi = points[i].x, yi = points[i].y;
      const xj = points[j].x, yj = points[j].y;
      
      if (((yi > py) !== (yj > py)) && 
          (px < (xj - xi) * (py - yi) / (yj - yi) + xi)) {
        inside = !inside;
      }
    }
    return inside;
  }

  // Check circle vs rotated rectangle
  checkBlockCollision(cx, cy, radius, block) {
    const bx = block.x + block.width / 2;
    const by = block.y + block.height / 2;
    const angle = -block.rotation * Math.PI / 180;
    
    // Transform circle center to block's local space
    const localX = Math.cos(angle) * (cx - bx) - Math.sin(angle) * (cy - by) + block.width / 2;
    const localY = Math.sin(angle) * (cx - bx) + Math.cos(angle) * (cy - by) + block.height / 2;
    
    // Find closest point on rectangle
    const closestX = Math.max(0, Math.min(block.width, localX));
    const closestY = Math.max(0, Math.min(block.height, localY));
    
    const dist = Math.sqrt((localX - closestX) ** 2 + (localY - closestY) ** 2);
    return dist < radius;
  }

  // Check circle vs wall
  checkWallCollision(cx, cy, radius, wall) {
    const dist = this.pointToLineDistance(cx, cy, wall.x1, wall.y1, wall.x2, wall.y2);
    return dist < radius + wall.thickness / 2;
  }

  // Check circle vs polygon
  checkPolygonCollision(cx, cy, radius, polygon) {
    // Check if outside inverted polygon
    if (polygon.inverted && !this.pointInPolygon(cx, cy, polygon.points)) {
      return true;
    }
    
    // Check edge collisions
    const points = polygon.points;
    for (let i = 0; i < points.length; i++) {
      const j = (i + 1) % points.length;
      const dist = this.pointToLineDistance(
        cx, cy, 
        points[i].x, points[i].y, 
        points[j].x, points[j].y
      );
      if (dist < radius) return true;
    }
    return false;
  }

  // Main collision check
  checkCollision(x, y, radius) {
    for (const block of this.blocks) {
      if (this.checkBlockCollision(x, y, radius, block)) return true;
    }
    for (const wall of this.walls) {
      if (this.checkWallCollision(x, y, radius, wall)) return true;
    }
    for (const polygon of this.polygons) {
      if (this.checkPolygonCollision(x, y, radius, polygon)) return true;
    }
    return false;
  }

  // Movement with collision response
  tryMove(x, y, dx, dy, radius, bounds) {
    let newX = x + dx;
    let newY = y + dy;
    
    // Clamp to bounds
    newX = Math.max(radius, Math.min(bounds.width - radius, newX));
    newY = Math.max(radius, Math.min(bounds.height - radius, newY));
    
    // Try full movement
    if (!this.checkCollision(newX, newY, radius)) {
      return { x: newX, y: newY };
    }
    
    // Try sliding on X axis
    const canMoveX = !this.checkCollision(x + dx, y, radius);
    const canMoveY = !this.checkCollision(x, y + dy, radius);
    
    return {
      x: canMoveX ? Math.max(radius, Math.min(bounds.width - radius, x + dx)) : x,
      y: canMoveY ? Math.max(radius, Math.min(bounds.height - radius, y + dy)) : y
    };
  }
}
```

### Step 4: Game Integration Example

```jsx
// Game.jsx
import React, { useState, useEffect, useCallback } from 'react';
import { CollisionSystem } from './CollisionSystem';
import levelData from '../levels/level1.json';

const Game = () => {
  const [player, setPlayer] = useState(levelData.playerStart);
  const [collision] = useState(() => new CollisionSystem(levelData));
  const [keys, setKeys] = useState({});
  
  const SPEED = 4;
  const RADIUS = 15;
  const BOUNDS = { width: 800, height: 600 };

  useEffect(() => {
    const handleKeyDown = (e) => setKeys(k => ({ ...k, [e.key.toLowerCase()]: true }));
    const handleKeyUp = (e) => setKeys(k => ({ ...k, [e.key.toLowerCase()]: false }));
    
    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
    };
  }, []);

  useEffect(() => {
    const loop = setInterval(() => {
      let dx = 0, dy = 0;
      if (keys.w || keys.arrowup) dy -= SPEED;
      if (keys.s || keys.arrowdown) dy += SPEED;
      if (keys.a || keys.arrowleft) dx -= SPEED;
      if (keys.d || keys.arrowright) dx += SPEED;
      
      if (dx !== 0 || dy !== 0) {
        setPlayer(p => collision.tryMove(p.x, p.y, dx, dy, RADIUS, BOUNDS));
      }
    }, 16);
    
    return () => clearInterval(loop);
  }, [keys, collision]);

  return (
    <div 
      style={{
        width: BOUNDS.width,
        height: BOUNDS.height,
        backgroundImage: `url(${levelData.backgroundImage})`,
        backgroundSize: 'cover',
        position: 'relative'
      }}
    >
      {/* Player */}
      <div style={{
        position: 'absolute',
        left: player.x - RADIUS,
        top: player.y - RADIUS,
        width: RADIUS * 2,
        height: RADIUS * 2,
        borderRadius: '50%',
        backgroundColor: '#3b82f6',
        border: '2px solid white'
      }} />
    </div>
  );
};

export default Game;
```

---

## Editor/Game Mode Switching

For development builds, include a mode toggle:

```jsx
// App.jsx
import React, { useState } from 'react';
import LevelEditor from './components/LevelEditor';
import Game from './game/Game';

const App = () => {
  const [mode, setMode] = useState('editor'); // 'editor' | 'game'
  
  return (
    <div>
      <div style={{ padding: 10, background: '#1a1a2e' }}>
        <button onClick={() => setMode('editor')}>Editor</button>
        <button onClick={() => setMode('game')}>Game</button>
      </div>
      {mode === 'editor' ? <LevelEditor /> : <Game />}
    </div>
  );
};
```

---

## Advanced Integration Options

### Option A: Embedded Editor (Development Only)

```jsx
// Include editor in dev builds only
{process.env.NODE_ENV === 'development' && (
  <LevelEditor onExport={(data) => console.log(data)} />
)}
```

### Option B: Separate Editor Application

Build the editor as a standalone tool:

```bash
# Create production build of editor
npm run build

# Deploy to internal tools server
# Level designers export JSON files
# Import JSON into game assets
```

### Option C: Runtime Level Loading

```javascript
// Load levels dynamically
const loadLevel = async (levelName) => {
  const response = await fetch(`/levels/${levelName}.json`);
  const data = await response.json();
  return new CollisionSystem(data);
};
```

---

## Debug Visualization

Add optional collision visualization in your game:

```jsx
const DebugOverlay = ({ blocks, walls, polygons, visible }) => {
  if (!visible) return null;
  
  return (
    <svg style={{ position: 'absolute', inset: 0, pointerEvents: 'none' }}>
      {/* Blocks */}
      {blocks.map(b => (
        <rect
          key={b.id}
          x={b.x} y={b.y}
          width={b.width} height={b.height}
          transform={`rotate(${b.rotation} ${b.x + b.width/2} ${b.y + b.height/2})`}
          fill="rgba(255,0,0,0.3)"
          stroke="red"
        />
      ))}
      
      {/* Walls */}
      {walls.map(w => (
        <line
          key={w.id}
          x1={w.x1} y1={w.y1}
          x2={w.x2} y2={w.y2}
          stroke="orange"
          strokeWidth={w.thickness}
          strokeLinecap="round"
        />
      ))}
      
      {/* Polygons */}
      {polygons.map(p => (
        <polygon
          key={p.id}
          points={p.points.map(pt => `${pt.x},${pt.y}`).join(' ')}
          fill={p.inverted ? 'rgba(128,0,128,0.2)' : 'rgba(0,255,0,0.2)'}
          stroke="green"
        />
      ))}
    </svg>
  );
};
```

---

## Extending the Editor

### Adding Custom Collision Types

```javascript
// In LevelEditor.jsx, add to state:
const [customShapes, setCustomShapes] = useState([]);

// Add new tool type
// Implement collision detection for new shape
// Add to export format
```

### Adding Trigger Zones

Non-blocking areas that trigger events:

```javascript
{
  "triggers": [
    {
      "id": 123,
      "type": "zone",
      "shape": "rect",
      "x": 100, "y": 100,
      "width": 50, "height": 50,
      "event": "levelComplete"
    }
  ]
}
```

### Layer Support

For multi-layer collision (e.g., bridges):

```javascript
{
  "blocks": [
    { "id": 1, "layer": 0, ... },  // Ground level
    { "id": 2, "layer": 1, ... }   // Bridge level
  ]
}
```

---

## Performance Considerations

For large levels with many collision objects:

1. **Spatial Partitioning**: Implement a grid or quadtree
2. **Broad Phase**: Quick AABB checks before precise collision
3. **Object Pooling**: Reuse collision check results
4. **Static Batching**: Pre-compute static collision geometry

```javascript
// Simple grid-based spatial hash
class SpatialHash {
  constructor(cellSize = 100) {
    this.cellSize = cellSize;
    this.grid = new Map();
  }
  
  getKey(x, y) {
    return `${Math.floor(x / this.cellSize)},${Math.floor(y / this.cellSize)}`;
  }
  
  insert(obj, bounds) {
    // Add object to all cells it overlaps
  }
  
  query(x, y, radius) {
    // Return objects in nearby cells only
  }
}
```

---

## Checklist

- [ ] Copy `LevelEditor.jsx` to project
- [ ] Create `CollisionSystem.js` with collision logic
- [ ] Set up level JSON storage (static files or database)
- [ ] Implement game loop with collision checking
- [ ] Add debug visualization toggle
- [ ] Test all collision types (blocks, walls, polygons)
- [ ] Test inverted polygon boundaries
- [ ] Optimize for target level complexity

---

## File Reference

| File | Purpose |
|------|---------|
| `level-editor.jsx` | Visual editor component |
| `CollisionSystem.js` | Runtime collision detection |
| `level1.json` | Example exported level |
| `demo.html` | Standalone editor demo |

