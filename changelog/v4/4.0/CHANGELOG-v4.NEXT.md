# Phaser 4 Changelog

## Version 4.NEXT

### New Features

- `RenderConfig#mipmapRegeneration` option allows certain framebuffer-based objects to use mipmaps if the game is configured to use mipmaps. This has a cost because mipmaps must be recreated after every change. Currently it only applies to DynamicTextures; Filters cannot render mipmaps. Thanks @Flow!

### Fixes

- Fix reversions in rounded rectangle handling. Thanks @laineus!
- Remove duplicate function definition and exposed internal code docs from `RectangleCanvasRenderer`.
- Fix duplicate texture name resulting from `RenderTexture#saveTexture`. Thanks @UnaiNeuronUp!
- Fix framebuffers (in filters and DynamicTextures) using mipmaps incorrectly. Now filters do not render with mipmaps. Thanks @Flow!
- Fix lack of default export in ESM build. Thanks @kibertoad!
