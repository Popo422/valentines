# 3D Valentine's Sunflower Bouquet - Implementation Plan

## Overview
Create an interactive Valentine's webpage with a 3D sunflower bouquet using Three.js.

---

## CRITICAL: Sunflower Geometry (READ CAREFULLY)

### The Sunflower Structure
Think of a sunflower as a **flat 2D flower viewed from the front**:
- A circular brown disk (the center)
- Yellow petals radiating outward from the disk's edge
- **IMPORTANT**: Petals are like **WHEEL SPOKES** or **TIRE SPOKES** - they stick out from the circumference of the disk, all lying FLAT in the SAME PLANE as the disk face

### Visual Reference
```
        Top View (looking at flower face):

             petal
               |
         petal-+-petal
              /|\
             / | \
      petal-+--O--+-petal   <-- O is the brown disk center
             \ | /
              \|/
         petal-+-petal
               |
             petal

    All petals are FLAT, radiating outward like spokes
```

### Three.js Implementation for Sunflower Head

1. **Brown Center Disk**:
   - Use `CylinderGeometry(radius=1.2, height=0.3)`
   - Rotate it so the circular face points toward camera: `disk.rotation.x = Math.PI / 2`
   - This makes the disk face the +Z direction (toward viewer)

2. **Petals** (THIS IS THE TRICKY PART):
   - Create petal shape using `THREE.Shape()` with bezier curves (elongated leaf shape)
   - The shape is drawn in X-Y plane, pointing UP (+Y direction)
   - Use `ExtrudeGeometry` to give it slight thickness

   **For each petal around the circumference**:
   ```javascript
   for (let i = 0; i < 20; i++) {
       const angle = (i / 20) * Math.PI * 2;
       const petal = createPetal();

       // Position at the edge of disk (radius 1.3, slightly outside disk edge)
       petal.position.set(
           Math.cos(angle) * 1.3,  // X
           Math.sin(angle) * 1.3,  // Y
           0                        // Z (same plane as disk center)
       );

       // ONLY rotate around Z-axis to point outward
       // angle - PI/2 rotates the petal from pointing up to pointing outward
       petal.rotation.z = angle - Math.PI / 2;

       // NO rotation on X or Y! Keep it flat in the disk's plane
   }
   ```

3. **Put petals in a group and rotate to match disk**:
   ```javascript
   petalGroup.rotation.x = Math.PI / 2;  // Same as disk rotation
   ```

### Common Mistakes to AVOID:
- ❌ Adding `petal.rotation.x` or `petal.rotation.y` - this tilts petals out of the flat plane
- ❌ Forgetting to rotate the petalGroup to match disk orientation
- ❌ Positioning petals in wrong plane

---

## File Structure
```
Valentine/
└── index.html    # Single file with embedded CSS/JS
```

## HTML Structure
- Full-screen canvas container
- Dark background: `#1a1a2e`
- Overlay with question box: "Will You Be My Valentine?"
- Yes button (orange/red gradient) → triggers flower animation
- No button (gray) → dodges on hover, clickable after 15 attempts → sad message
- Final message: "I LOVE YOU" with hearts and reset button

## Bouquet Arrangement (10 Flowers)
| Type | Count | Height | Tilt | Scale |
|------|-------|--------|------|-------|
| Center | 1 | 10 | None | 1.0 |
| Medium ring | 4 | 6.5-8.5 | Slight outward (0.15 rad) | 0.88-0.95 |
| Outer ring | 5 | 4.8-5.8 | More tilt (0.25-0.3 rad) | 0.7-0.85 |

Each flower has:
- Green cylindrical stem
- Brown center disk
- 20 yellow petals (wheel spoke arrangement)

## Animation Sequence
Stagger each flower by 400ms delay:
1. **Stem grows** (0-2s): Scale Y from 0→1, position Y follows
2. **Disk appears** (2-3s): Scale from 0→1
3. **Petals bloom** (3-4s): Each petal scales in with 30ms stagger between petals
4. **Easing**: Use `easeOutCubic` for smooth deceleration

## Web Audio API Sounds
| Sound | When | How |
|-------|------|-----|
| Grow | Stem animation starts | Sine wave 80Hz→200Hz sweep |
| Bloom | Petals appear | Ascending chord C5-E5-G5-C6 |
| Cheer | "I LOVE YOU" appears | 4-chord fanfare |
| Dodge | No button escapes | Quick boop 600Hz→400Hz |
| Sad | 15 failed No attempts | Descending trombone notes |

## Interactions
- **Yes click**: Hide question, show final message, animate bouquet
- **No hover**: Dodge to random position (15 times max)
- **No click after 15**: Show sad message
- **Drag/touch**: Rotate bouquet on X and Y axes
- **Idle**: Gentle auto-rotation `bouquet.rotation.y += 0.002`
- **Reset**: Return to initial state

## Three.js Setup
- Scene with background `0x1a1a2e`
- PerspectiveCamera: FOV 60, position (0, 5, 20)
- WebGLRenderer with antialiasing
- Lights:
  - AmbientLight (0.6 intensity)
  - DirectionalLight from (5, 10, 7)
  - PointLight golden accent from (-5, 5, 5)

## Responsive Design
- Canvas fills viewport
- Question box max-width 90%
- Smaller fonts/padding on mobile (max-width: 600px)
- Touch events with `passive: false` to prevent scroll
