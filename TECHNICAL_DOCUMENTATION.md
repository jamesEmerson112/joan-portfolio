# How Joan Positions Objects and Adds Code - Documentation

## Object Positioning & Integration Workflow

This documentation focuses specifically on how Joan positions 3D objects in space and integrates new code into the portfolio project.

## Step-by-Step Process for Adding 3D Objects

### 1. **Create/Place the 3D Model File**
- Export model as `.glb` format (GLTF binary with compression)
- Place in `static/assets/models/` directory
- Example: `static/assets/models/newObject.glb`

### 2. **Register Asset in Manifest (`src/Experience/assets.js`)**
Add the model to the assets array:
```javascript
export default [
  {
    name: "base",
    data: {},
    items: [
      // Existing assets...
      { name: "newObject", source: "/assets/models/newObject.glb" },
      // More assets...
    ]
  }
];
```

### 3. **Define Position Constants (`src/Experience/constants.js`)**
Joan stores all positioning data in constants for easy adjustment:
```javascript
import { Vector3, Euler } from "three";

// New Object positioning
export const NEW_OBJECT_POSITION = new Vector3(1.5, 2.0, -3.0); // X, Y, Z coordinates
export const NEW_OBJECT_ROTATION = new Euler(0, Math.PI/4, 0);  // Rotation in radians
export const NEW_OBJECT_SCALE = 0.5; // Uniform scale factor
```

### 4. **Create Object Class (`src/Experience/NewObject.js`)**
Every object follows this standard pattern:
```javascript
import Experience from "./Experience.js";
import { NEW_OBJECT_POSITION, NEW_OBJECT_ROTATION, NEW_OBJECT_SCALE } from "./constants.js";

export default class NewObject {
  constructor() {
    // Get core system references
    this.experience = new Experience();
    this.scene = this.experience.scene;
    this.resources = this.experience.resources;

    // Reference to shared materials from baked lighting
    this.sharedMaterial = this.experience.world.baked.model.material;

    this.setModel();
  }

  setModel() {
    // Load the model from resources
    this.model = {};
    this.model.mesh = this.resources.items.newObject.scene;

    // Apply shared baked material to all meshes
    this.model.mesh.traverse((child) => {
      if (child.isMesh) {
        child.material = this.sharedMaterial;
      }
    });

    // Position the object using constants
    this.model.mesh.position.copy(NEW_OBJECT_POSITION);
    this.model.mesh.rotation.copy(NEW_OBJECT_ROTATION);
    this.model.mesh.scale.setScalar(NEW_OBJECT_SCALE);

    // Set name for identification
    this.model.mesh.name = "newObject";

    // Add to scene
    this.scene.add(this.model.mesh);
  }
}
```

### 5. **Register in World Manager (`src/Experience/World.js`)**
Import and initialize the new object:
```javascript
import NewObject from "./NewObject.js";

export default class World {
  constructor(_options) {
    // ... existing code ...

    this.resources.on("groupEnd", (_group) => {
      if (_group.name === "base") {
        // ... existing objects ...
        this.setNewObject(); // Add this line
      }
    });
  }

  // Add this method
  setNewObject() {
    this.newObject = new NewObject();
  }

  // ... rest of existing code ...
}
```

## Real Examples from Joan's Code

### Example 1: Arcade Machine Positioning
**Constants defined:**
```javascript
// constants.js
export const ARCADE_CSS_OBJECT_POSITION = new Vector3(3.24776, 2.7421, 2.3009);
export const ARCADE_CSS_OBJECT_ROTATION_X = -Math.PI / 7;
export const ARCADE_CSS_OBJECT_ROTATION_Y = -Math.PI / 2;
```

**Object implementation:**
```javascript
// ArcadeScreen.js
setModel = () => {
  this.model.arcadeMachineModel = this.resources.items.arcadeMachine.scene;
  this.model.arcadeMachineModel.traverse((child) => {
    if (child.isMesh) {
      child.material = this.arcadeMachineMaterial;
    }
  });
  this.model.arcadeMachineModel.name = "arcadeMachine";
  this.scene.add(this.model.arcadeMachineModel);
};
```

### Example 2: Whiteboard Positioning
**Constants:**
```javascript
// constants.js - Camera position for whiteboard view
export const WHITEBOARD_CAMERA_POSITION = new Vector3(-3.3927, 5.18774, 4.61366);
export const WHITEBOARD_CAMERA_TARGET = new Vector3(-3.3927, 3.18774, -4.61366);
```

**Canvas positioning in object:**
```javascript
// Whiteboard.js
const planeMesh = new Mesh(whiteboardGeom, this.whiteboardMaterial);
planeMesh.position.set(-3.3927, 3.18774, -4.61366); // Exact 3D coordinates
planeMesh.name = "whiteboardCanvas";
this.model.mesh.add(planeMesh);
```

### Example 3: Rubik's Cube Positioning
**Constants:**
```javascript
// constants.js
export const RUBIK_POSITION = new Vector3(-0.67868, 1.499, -3.92849);
export const RUBIK_SCALE = 0.021432;
export const RUBIK_ROTATION_Y = (-152.484 * Math.PI) / 180;
```

**Usage in object:**
```javascript
// RubiksCube.js constructor
this.rubiksCube = new RubiksCube(RUBIK_POSITION, RUBIK_SCALE);
```

## Positioning Coordinate System

Joan uses Three.js standard coordinate system:
- **X-axis**: Left (-) to Right (+)
- **Y-axis**: Down (-) to Up (+)
- **Z-axis**: Away (-) to Towards camera (+)

**Typical object positions in Joan's room:**
- **Arcade Machine**: `(3.24776, 2.7421, 2.3009)` - Right side, elevated
- **Whiteboard**: `(-3.3927, 3.18774, -4.61366)` - Left wall, mounted high
- **Rubik's Cube**: `(-0.67868, 1.499, -3.92849)` - On desk surface
- **Top Chair**: `(1.4027, 0.496728, -1.21048)` - Floor level, center-right

## Material Application Pattern

Joan uses shared baked materials for consistent lighting:
```javascript
// Get shared material reference
this.sharedMaterial = this.experience.world.baked.model.material;

// Apply to all mesh children
object.traverse((child) => {
  if (child.isMesh) {
    child.material = this.sharedMaterial;
  }
});
```

## Integration Checklist

When adding a new object, ensure:

✅ **Asset registered** in `assets.js`
✅ **Constants defined** in `constants.js`
✅ **Object class created** with positioning
✅ **World.js updated** to initialize object
✅ **Shared materials applied** for consistent lighting
✅ **Object named** for identification/raycasting

This workflow ensures consistent integration and easy maintenance of
