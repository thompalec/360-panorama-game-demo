# 360° Panoramic Game Demo - Project Writeup

## Project Overview

Build an interactive 3D game demo that displays a dual-viewport system: a standard 120° first-person view on top, and a full 360° horizontal panoramic view on the bottom created via cubemap rendering and cylindrical projection.

## Core Technologies

- **Three.js**: 3D rendering engine
- **WebGL**: Graphics API
- **Canvas**: Rendering target
- **Touch & Keyboard Input**: Player controls

## Architecture & Components

### 1. Scene Setup
- Create a Three.js scene with basic geometry (spheres, cubes) for testing
- Add lighting (ambient + directional) so objects are visible
- Implement a ground plane or simple environment for spatial reference
- Load textures if desired for visual variety

### 2. Player & Camera System
- Maintain player position (x, y, z) in the scene
- Track player rotation (yaw/pitch for viewing direction)
- Create two camera objects:
  - **Top Camera**: Standard perspective camera (75–80° FOV) for the main view
  - **Cubemap Camera**: Uses Three.js `CubeCamera` for omnidirectional rendering

### 3. Dual Viewport Rendering
- Divide the canvas into two equal halves (top and bottom)
- Set up scissor tests and viewport calls to render each half independently
- Top half: render scene with standard camera
- Bottom half: render scene with cubemap camera, then project cubemap onto cylinder

### 4. Cubemap Rendering Pipeline
- Create a `WebGLCubeRenderTarget` to store the six cubemap faces
- Each frame, render the scene six times using `CubeCamera` positioned at the player's location
- The cubemap captures 360° × 360° of the scene (all directions)

### 5. Cylindrical Projection & Display
- Create a cylinder mesh (just the curved surface, no caps/sides)
- Position it at the player's location
- Write a custom fragment shader that:
  - Takes cylinder UV coordinates (0–1 horizontally around the circumference, 0–1 vertically)
  - Converts horizontal UV to an angle (0 to 2π)
  - Creates a 3D direction vector pointing outward from the cylinder
  - Samples the cubemap texture using this direction
  - Returns the sampled color
- Render the cylinder to the bottom viewport using the cubemap as a texture

### 6. Input Handling
- **Keyboard (WASD)**:
  - W/A/S/D keys move the player forward/left/backward/right
  - Movement is relative to the player's viewing direction
  - Update player position each frame
- **Touch/Mouse Input**:
  - Track mouse/touch position and delta (change from previous frame)
  - Horizontal delta adjusts yaw (player rotation around Y axis)
  - Vertical delta adjusts pitch (camera up/down tilt)
  - Clamp pitch to prevent camera flip (e.g., ±80°)

### 7. Main Update Loop
- Poll input and update player position/rotation
- Update camera matrices based on player state
- Render cubemap (expensive operation, consider optimizing to less frequent updates)
- Render top viewport with standard camera
- Render bottom viewport with cylinder + cubemap projection
- Request next animation frame

### 8. Performance Optimization (Optional, but Recommended)
- Only update the cubemap when the player moves significantly or rotates, not every frame
- Render cubemap at reduced resolution (512×512 or 1024×1024) instead of full screen resolution
- Consider rendering the panorama at a lower frame rate than the main view

## File Structure

**Single HTML/JavaScript file** containing:
- Three.js library import (via CDN)
- Vertex shader for cylinder (simple passthrough with UV mapping)
- Fragment shader for cylindrical cubemap projection
- Scene initialization
- Input handling system
- Render loop
- All game logic

## Key Implementation Details

### Cylindrical Projection Shader
The fragment shader should:
1. Take `vUv` (UV coordinates from cylinder geometry) where x is horizontal (0–1) and y is vertical (0–1)
2. Convert `vUv.x * 2π` to get an angle around the Y axis
3. Create a direction vector: `vec3(cos(angle), verticalOffset, sin(angle))`
4. Normalize and sample the cubemap: `textureCube(cubemap, direction)`
5. Return the sampled color

### Movement & Rotation
- Store player position as `{x, y, z}` and rotation as `{yaw, pitch}`
- Create a forward vector from yaw/pitch
- WASD movement adds scaled forward/right vectors to position
- Mouse/touch delta updates yaw/pitch in radians

### Cubemap Camera Setup
- Position at player location
- Has no rotation (it captures all directions equally)
- Renders to WebGLCubeRenderTarget with appropriate resolution
- Use Three.js `CubeCamera.update(renderer, scene)` to trigger six render passes

## Potential Challenges & Solutions

**Seam Visibility**: Cubemaps handle seams naturally via texture filtering, but cylindrical projection may reveal them. Solution: Use higher resolution cubemaps or add subtle blurring in seams.

**Performance**: Rendering six views per frame is expensive. Solution: Update cubemap every N frames or only when player moves.

**Vertical Distortion**: If you sample the cubemap with full vertical freedom, poles will pinch. Solution: Clamp vertical sampling to a reasonable range (e.g., ±30°) to flatten the panorama.

**Touch Controls on Mobile**: Dragging should feel smooth and intuitive. Solution: Use normalized delta and sensitivity scaling; test extensively on actual devices.

## Success Criteria

- Two viewports render side-by-side on one canvas
- Top viewport shows standard first-person 3D view
- Bottom viewport shows 360° horizontal panorama
- WASD movement feels responsive and moves player through scene
- Mouse/touch input allows smooth camera rotation
- No obvious visual artifacts or seams in panoramic view
- Runs at reasonable frame rate (30+ FPS target)

## Testing Checklist

- Verify scene loads with geometry visible
- Test WASD movement in all directions
- Test mouse movement rotates view smoothly
- Test touch input on mobile devices
- Confirm panoramic view updates as player moves
- Check for visual seams or artifacts in panorama
- Monitor performance/frame rate
- Test edge cases (moving near scene objects, rotating rapidly, etc.)