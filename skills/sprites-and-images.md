# Sprites and Images
> Creating and manipulating Sprite and Image game objects in Phaser 4 -- factory methods, texture/frame selection, the component mixin system, and common visual operations (position, scale, rotation, tint, flip, alpha, origin, depth).

**Key source paths:** `src/gameobjects/sprite/`, `src/gameobjects/image/`, `src/gameobjects/GameObject.js`, `src/gameobjects/components/`
**Related skills files:** loading-assets.md, animations.md, physics-arcade.md, game-object-components.md

## Quick Start

```js
// In a Scene's create() method:

// Static image (no animation support, slightly cheaper)
const bg = this.add.image(400, 300, 'background');

// Sprite (supports animations)
const player = this.add.sprite(100, 200, 'player', 'idle-0');

// Common operations -- all methods return `this` for chaining
player.setPosition(200, 300);
player.setScale(2);
player.setAngle(45);
player.setTint(0xff0000);
player.setAlpha(0.8);
player.setOrigin(0, 1);       // bottom-left
player.setDepth(10);
player.setFlip(true, false);  // flip horizontally
player.setVisible(false);

// Chained
this.add.sprite(100, 100, 'coin')
    .setScale(0.5)
    .setTint(0xffff00)
    .play('spin');
```

## Core Concepts

### Sprite vs Image

Both extend `GameObject` and share the **same set of component mixins**. The only difference is that Sprite includes an `AnimationState` instance (`sprite.anims`) and animation convenience methods (`play`, `stop`, `chain`, etc.).

| Feature | Image | Sprite |
|---|---|---|
| Static texture display | Yes | Yes |
| Tint, alpha, flip, scale, rotate | Yes | Yes |
| Physics body | Yes | Yes |
| Input / hit area | Yes | Yes |
| Animation (`play`, `stop`, `chain`) | **No** | **Yes** |
| `preUpdate` called each frame | No | Yes (updates animation) |
| Added to Scene `updateList` | No | Yes |

**Rule of thumb:** Use `Image` for anything that does not need frame-by-frame animation. It skips the per-frame `preUpdate` cost and has a smaller API surface. Use `Sprite` only when you need the Animation component.

### The Component Mixin System

Phaser builds Game Object classes by mixing component objects into the prototype. Both `Sprite` and `Image` share this identical Mixins array (sourced from `src/gameobjects/sprite/Sprite.js` and `src/gameobjects/image/Image.js`):

```
Mixins: [
    Components.Alpha,
    Components.BlendMode,
    Components.Depth,
    Components.Flip,
    Components.GetBounds,
    Components.Lighting,
    Components.Mask,
    Components.Origin,
    Components.RenderNodes,
    Components.ScrollFactor,
    Components.Size,
    Components.TextureCrop,
    Components.Tint,
    Components.Transform,
    Components.Visible,
    SpriteRender / ImageRender   // render-specific (differs per class)
]
```

The base `GameObject` class itself mixes in:
```
Mixins: [
    Components.Filters,
    Components.RenderSteps
]
```

Each component adds specific properties and methods to every instance. For example, `Components.Transform` adds `x`, `y`, `scale`, `rotation`, `setPosition()`, etc. The full list of available components is in `src/gameobjects/components/index.js`.

**Key point for agents:** When you see a method like `setAlpha()` on a Sprite, it comes from `Components.Alpha`, not from the Sprite class itself. The component source file is the authoritative reference for that method's signature and behavior.

### Texture and Frame

Both Sprite and Image use the `TextureCrop` component which provides:

- `texture` -- the `Phaser.Textures.Texture` instance
- `frame` -- the current `Phaser.Textures.Frame` instance
- `setTexture(key, frame)` -- change the texture (and optionally the frame)
- `setFrame(frame, updateSize, updateOrigin)` -- change only the frame
- `setCrop(x, y, width, height)` -- crop a rectangular region of the texture
- `isCropped` -- boolean, toggle cropping on/off after `setCrop`

The `texture` parameter in factory methods and constructors accepts either a string key (as registered in the Texture Manager) or a `Phaser.Textures.Texture` instance.

The `frame` parameter accepts a string name or numeric index into the texture's frame collection. If omitted, the base frame (frame 0 / `'__BASE'`) is used.

```js
// Change texture at runtime
sprite.setTexture('enemies', 'goblin-walk-1');

// Change only the frame (must belong to current texture)
sprite.setFrame('goblin-walk-2');

// setFrame signature:
// setFrame(frame, updateSize=true, updateOrigin=true)
// Pass false to prevent automatic resize/origin recalculation
sprite.setFrame('small-frame', false, false);

// Crop to show only a 50x50 region starting at (10, 10)
sprite.setCrop(10, 10, 50, 50);
// Reset crop
sprite.setCrop();
```

## Common Patterns

### Creating and Positioning

**Transform component** (`src/gameobjects/components/Transform.js`):

```js
// Properties (all read/write)
sprite.x           // horizontal position (default: 0)
sprite.y           // vertical position (default: 0)
sprite.z           // z position (does NOT control render order -- use depth)
sprite.w           // w position

// Methods
sprite.setPosition(x, y, z, w)        // y defaults to x if omitted
sprite.setRandomPosition(x, y, w, h)  // random within area; defaults to game size
sprite.copyPosition(source)            // copy from any {x, y} object
```

**Size component** (`src/gameobjects/components/Size.js`):

```js
sprite.width            // native (un-scaled) width
sprite.height           // native (un-scaled) height
sprite.displayWidth      // scaled width (read/write -- setting adjusts scaleX)
sprite.displayHeight     // scaled height (read/write -- setting adjusts scaleY)

sprite.setSize(width, height)           // set internal size (not visual)
sprite.setDisplaySize(width, height)    // set visual size (adjusts scale)
sprite.setSizeToFrame(frame)            // reset size to match frame
```

### Scaling and Rotation

**Transform component** (continued):

```js
// Scale properties
sprite.scale       // uniform scale (getter returns average of scaleX+scaleY)
sprite.scaleX      // horizontal scale (default: 1)
sprite.scaleY      // vertical scale (default: 1)

// Scale methods
sprite.setScale(x, y)    // y defaults to x if omitted

// Rotation properties
sprite.rotation    // in radians (right-hand clockwise: 0=right, PI/2=down)
sprite.angle       // in degrees (0=right, 90=down, 180/-180=left, -90=up)

// Rotation methods
sprite.setRotation(radians)   // defaults to 0
sprite.setAngle(degrees)      // defaults to 0
```

### Tinting and Alpha

**Tint component** (`src/gameobjects/components/Tint.js`) -- WebGL only:

```js
// Properties
sprite.tint             // overall tint (getter returns tintTopLeft)
sprite.tintTopLeft      // default: 0xffffff
sprite.tintTopRight     // default: 0xffffff
sprite.tintBottomLeft   // default: 0xffffff
sprite.tintBottomRight  // default: 0xffffff
sprite.tintMode         // default: Phaser.TintModes.MULTIPLY
sprite.isTinted         // read-only boolean

// Methods
sprite.setTint(topLeft, topRight, bottomLeft, bottomRight)
// If only topLeft given, applies uniformly to all four corners
sprite.setTintMode(mode)   // Phaser.TintModes.MULTIPLY | FILL | ADD | SCREEN | OVERLAY | HARD_LIGHT
sprite.clearTint()         // resets to 0xffffff + MULTIPLY mode
```

**Phaser 4 change:** `setTintFill()` is removed. Use `setTint(color).setTintMode(Phaser.TintModes.FILL)` instead.

**Alpha component** (`src/gameobjects/components/Alpha.js`):

```js
// Properties
sprite.alpha             // global alpha 0-1 (default: 1)
sprite.alphaTopLeft      // per-corner alpha (WebGL only)
sprite.alphaTopRight
sprite.alphaBottomLeft
sprite.alphaBottomRight

// Methods
sprite.setAlpha(topLeft, topRight, bottomLeft, bottomRight)
// If only topLeft given, applies uniformly to whole object
sprite.clearAlpha()     // resets to 1 (fully opaque)
```

Setting `alpha` to 0 clears the render flag so the object is not drawn. Setting it back to any non-zero value restores it.

### Flipping

**Flip component** (`src/gameobjects/components/Flip.js`):

```js
// Properties
sprite.flipX    // boolean (default: false)
sprite.flipY    // boolean (default: false)

// Methods
sprite.setFlipX(value)        // set horizontal flip
sprite.setFlipY(value)        // set vertical flip
sprite.setFlip(x, y)          // set both at once
sprite.toggleFlipX()          // invert current horizontal flip
sprite.toggleFlipY()          // invert current vertical flip
sprite.resetFlip()            // set both to false
```

Flipping is a rendering toggle only. It does not affect physics bodies or hit areas. Flip occurs from the middle of the texture and does not change the scale value.

### Origin and Depth

**Origin component** (`src/gameobjects/components/Origin.js`):

```js
// Properties (normalized 0-1)
sprite.originX          // default: 0.5 (center)
sprite.originY          // default: 0.5 (center)
sprite.displayOriginX   // pixel value (originX * width)
sprite.displayOriginY   // pixel value (originY * height)

// Methods
sprite.setOrigin(x, y)           // y defaults to x; defaults to 0.5
sprite.setDisplayOrigin(x, y)    // set origin in pixels; y defaults to x
sprite.setOriginFromFrame()      // use Frame pivot if set, else 0.5
sprite.updateDisplayOrigin()     // recalculate pixel origin from normalized
```

**Depth component** (`src/gameobjects/components/Depth.js`):

```js
// Properties
sprite.depth    // default: 0 (higher = renders on top)

// Methods
sprite.setDepth(value)
sprite.setToTop()              // move to top of display list (no depth change)
sprite.setToBack()             // move to back of display list (no depth change)
sprite.setAbove(gameObject)    // position above another object in the list
sprite.setBelow(gameObject)    // position below another object in the list
```

Setting `depth` queues a depth sort in the Scene. The `setToTop`/`setToBack`/`setAbove`/`setBelow` methods change display list position without modifying the `depth` value.

### Destroying Game Objects

```js
sprite.destroy();           // removes from display list, update list, and cleans up
sprite.ignoreDestroy = true; // prevent destruction (you manage lifecycle manually)
sprite.isDestroyed           // boolean, true after destroy() has been called
```

For Sprites, `destroy()` also calls `this.anims.destroy()` to clean up the AnimationState.

## Additional Components

**Visible** (`src/gameobjects/components/Visible.js`):
```js
sprite.visible              // boolean (default: true)
sprite.setVisible(value)    // invisible objects skip rendering but still update
```

**ScrollFactor** (`src/gameobjects/components/ScrollFactor.js`):
```js
sprite.scrollFactorX        // default: 1 (moves with camera)
sprite.scrollFactorY        // default: 1
sprite.setScrollFactor(x, y)  // 0 = fixed to camera; y defaults to x
```

**BlendMode** (`src/gameobjects/components/BlendMode.js`):
```js
sprite.blendMode            // default: Phaser.BlendModes.NORMAL
sprite.setBlendMode(value)  // Phaser.BlendModes.ADD, MULTIPLY, SCREEN, ERASE, etc.
```

## Factory Methods

### this.add.sprite

Source: `src/gameobjects/sprite/SpriteFactory.js`

```js
// Signature
this.add.sprite(x, y, texture, frame);
```

| Param | Type | Required | Description |
|---|---|---|---|
| `x` | `number` | Yes | Horizontal position in world coordinates |
| `y` | `number` | Yes | Vertical position in world coordinates |
| `texture` | `string \| Phaser.Textures.Texture` | Yes | Texture key or Texture instance |
| `frame` | `string \| number` | No | Frame name or index within the texture |

Returns: `Phaser.GameObjects.Sprite`

Internally does: `this.displayList.add(new Sprite(this.scene, x, y, texture, frame))`

The Sprite constructor calls: `setTexture` -> `setPosition` -> `setSizeToFrame` -> `setOriginFromFrame` -> `initRenderNodes`.

### this.add.image

Source: `src/gameobjects/image/ImageFactory.js`

```js
// Signature
this.add.image(x, y, texture, frame);
```

| Param | Type | Required | Description |
|---|---|---|---|
| `x` | `number` | Yes | Horizontal position in world coordinates |
| `y` | `number` | Yes | Vertical position in world coordinates |
| `texture` | `string \| Phaser.Textures.Texture` | Yes | Texture key or Texture instance |
| `frame` | `string \| number` | No | Frame name or index within the texture |

Returns: `Phaser.GameObjects.Image`

Internally does: `this.displayList.add(new Image(this.scene, x, y, texture, frame))`

The Image constructor calls the same init sequence as Sprite: `setTexture` -> `setPosition` -> `setSizeToFrame` -> `setOriginFromFrame` -> `initRenderNodes`.

## API Quick Reference

Most-used properties and methods across all mixed-in components. All setter methods return `this` for chaining.

### Transform
| API | Type | Description |
|---|---|---|
| `x`, `y` | `number` | World position |
| `z`, `w` | `number` | Extra position axes (z does NOT control render order) |
| `scale` | `number` | Uniform scale (read: avg of scaleX+scaleY; write: sets both) |
| `scaleX`, `scaleY` | `number` | Per-axis scale |
| `angle` | `number` | Rotation in degrees (clockwise, 0=right) |
| `rotation` | `number` | Rotation in radians |
| `setPosition(x, y, z, w)` | method | Set position |
| `setScale(x, y)` | method | Set scale (y defaults to x) |
| `setRotation(radians)` | method | Set rotation in radians |
| `setAngle(degrees)` | method | Set rotation in degrees |
| `setRandomPosition(x, y, w, h)` | method | Randomize position within area |
| `copyPosition(source)` | method | Copy x/y/z/w from another object |

### Alpha
| API | Type | Description |
|---|---|---|
| `alpha` | `number` | Global opacity 0-1 |
| `alphaTopLeft/TR/BL/BR` | `number` | Per-corner alpha (WebGL) |
| `setAlpha(tl, tr, bl, br)` | method | Set alpha (single value or per-corner) |
| `clearAlpha()` | method | Reset to fully opaque |

### Tint (WebGL only)
| API | Type | Description |
|---|---|---|
| `tint` | `number` | Overall tint color (hex) |
| `tintTopLeft/TR/BL/BR` | `number` | Per-corner tint |
| `tintMode` | `number` | Tint blend mode |
| `isTinted` | `boolean` | Read-only: is any tint applied? |
| `setTint(tl, tr, bl, br)` | method | Set tint (single or per-corner) |
| `setTintMode(mode)` | method | Set tint blend mode |
| `clearTint()` | method | Reset tint to white + MULTIPLY |

### Origin
| API | Type | Description |
|---|---|---|
| `originX`, `originY` | `number` | Normalized origin 0-1 |
| `displayOriginX`, `displayOriginY` | `number` | Pixel origin values |
| `setOrigin(x, y)` | method | Set normalized origin |
| `setDisplayOrigin(x, y)` | method | Set pixel origin |

### Depth
| API | Type | Description |
|---|---|---|
| `depth` | `number` | Z-index for rendering |
| `setDepth(value)` | method | Set depth |
| `setToTop()` | method | Move to top of display list |
| `setToBack()` | method | Move to back of display list |
| `setAbove(gameObject)` | method | Position above another object |
| `setBelow(gameObject)` | method | Position below another object |

### Flip
| API | Type | Description |
|---|---|---|
| `flipX`, `flipY` | `boolean` | Flip state |
| `setFlip(x, y)` | method | Set both flip values |
| `setFlipX(value)` | method | Set horizontal flip |
| `setFlipY(value)` | method | Set vertical flip |
| `toggleFlipX()`, `toggleFlipY()` | method | Toggle flip state |
| `resetFlip()` | method | Reset both to false |

### Size
| API | Type | Description |
|---|---|---|
| `width`, `height` | `number` | Native (un-scaled) dimensions |
| `displayWidth`, `displayHeight` | `number` | Scaled dimensions (setting adjusts scale) |
| `setSize(w, h)` | method | Set internal size |
| `setDisplaySize(w, h)` | method | Set visual size (adjusts scale) |
| `setSizeToFrame(frame)` | method | Match size to frame |

### Texture (TextureCrop)
| API | Type | Description |
|---|---|---|
| `texture` | `Texture` | Current texture reference |
| `frame` | `Frame` | Current frame reference |
| `setTexture(key, frame)` | method | Change texture and frame |
| `setFrame(frame, updateSize, updateOrigin)` | method | Change frame only |
| `setCrop(x, y, w, h)` | method | Crop visible region |
| `isCropped` | `boolean` | Toggle crop on/off |

### Other
| API | Type | Description |
|---|---|---|
| `visible` / `setVisible(bool)` | | Visibility toggle |
| `scrollFactorX`, `scrollFactorY` / `setScrollFactor(x, y)` | | Camera scroll influence |
| `blendMode` / `setBlendMode(mode)` | | Blend mode |
| `destroy()` | method | Remove and clean up |
| `active` | `boolean` | Update-list processing flag (from GameObject) |
| `name` | `string` | Developer-assigned name (from GameObject) |
| `state` | `number\|string` | Developer-assigned state (from GameObject) |
| `setInteractive()` | method | Enable input (from GameObject) |
| `setData(key, value)` | method | Store custom data (from GameObject) |

## Gotchas and Common Mistakes

1. **Using Sprite when Image suffices.** Sprites are added to the Scene `updateList` and run `preUpdate` every frame for animation ticking. If you do not need animation, use `Image` instead to avoid unnecessary overhead.

2. **`z` does not control render order.** The `z` property on Transform is a generic coordinate, not a z-index. Use `depth` or the display list ordering methods (`setToTop`, `setAbove`, etc.) to control what renders on top.

3. **`scale` getter returns the average.** Reading `sprite.scale` returns `(scaleX + scaleY) / 2`. If scaleX and scaleY differ, this may be misleading. Check `scaleX`/`scaleY` individually when non-uniform scaling is used.

4. **Setting scale to 0 hides the object.** Setting `scale`, `scaleX`, or `scaleY` to 0 clears a render flag. The object will not render until scale is set back to a non-zero value.

5. **Setting alpha to 0 hides the object.** Same render-flag behavior as scale. Alpha of exactly 0 prevents rendering. Use `setVisible(false)` if you want to explicitly hide without changing alpha.

6. **`setTintFill` is removed in Phaser 4.** Use `sprite.setTint(color).setTintMode(Phaser.TintModes.FILL)` instead. Calling `setTintFill()` will log an error to the console.

7. **Tint and tintMode are now separate in Phaser 4.** In Phaser 3 they were set together. In Phaser 4, `setTint()` only sets the color; use `setTintMode()` to change the blend operation.

8. **Flip does not affect physics bodies.** Flipping is a rendering toggle only. If your game logic depends on facing direction, track it separately from `flipX`/`flipY`.

9. **`setFrame` resets size and origin by default.** Calling `setFrame('new-frame')` will resize the object and recalculate origin. Pass `false` for `updateSize` and `updateOrigin` to prevent this: `setFrame('f', false, false)`.

10. **ScrollFactor and physics.** Scroll factor is a visual offset only. Physics collisions always use world position regardless of scroll factor. A scroll factor other than 1 can cause visual misalignment with physics bodies.

11. **Origin default is center (0.5, 0.5).** Position coordinates refer to the center of the object by default. Set `setOrigin(0, 0)` if you want position to refer to the top-left corner.

12. **Rotation direction.** Phaser uses right-hand clockwise rotation: 0 = right, 90 degrees (PI/2) = down, 180 = left, -90 (or 270) = up.

## Source File Map

| File | Purpose |
|---|---|
| `src/gameobjects/sprite/Sprite.js` | Sprite class definition and Mixins array |
| `src/gameobjects/sprite/SpriteFactory.js` | `this.add.sprite` factory registration |
| `src/gameobjects/sprite/SpriteRender.js` | Sprite render methods |
| `src/gameobjects/image/Image.js` | Image class definition and Mixins array |
| `src/gameobjects/image/ImageFactory.js` | `this.add.image` factory registration |
| `src/gameobjects/image/ImageRender.js` | Image render methods |
| `src/gameobjects/GameObject.js` | Base class (scene, active, name, state, data, destroy) |
| `src/gameobjects/GameObjectFactory.js` | Factory registry (`this.add`) |
| `src/gameobjects/components/index.js` | Component module exports |
| `src/gameobjects/components/Transform.js` | Position, scale, rotation |
| `src/gameobjects/components/Alpha.js` | Alpha / opacity |
| `src/gameobjects/components/Tint.js` | Tint color and mode |
| `src/gameobjects/components/Origin.js` | Origin / anchor point |
| `src/gameobjects/components/Depth.js` | Depth / z-ordering |
| `src/gameobjects/components/Flip.js` | Horizontal/vertical flip |
| `src/gameobjects/components/Visible.js` | Visibility toggle |
| `src/gameobjects/components/Size.js` | Width, height, display size |
| `src/gameobjects/components/ScrollFactor.js` | Camera scroll factor |
| `src/gameobjects/components/BlendMode.js` | Blend mode |
| `src/gameobjects/components/TextureCrop.js` | Texture, frame, and crop |
| `src/gameobjects/components/GetBounds.js` | Bounding box calculations |
| `src/gameobjects/components/Mask.js` | Bitmap and geometry masks |
| `src/gameobjects/components/Lighting.js` | Normal map lighting |
| `src/gameobjects/components/RenderNodes.js` | WebGL render node setup |
| `src/gameobjects/components/Filters.js` | Post-processing filters |
| `src/gameobjects/components/RenderSteps.js` | Render step pipeline |
| `src/animations/AnimationState.js` | Animation state machine (Sprite only) |
