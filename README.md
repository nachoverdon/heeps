# Heeps
An "advanced" Heaps extension library.

## Ideology
* I looked at Heaps and found the lack of features and general spartan feature-list rather shitty.
* I'd push some of that to main library, but I doubt they'll be accepted.
* I do not blame or have a grudge against Heaps developers, it's their engine which they develop for their needs that arise internally. And they want it to be as simple and dumb as possible, avoiding anything too advanced. It's their right and their vision.
* My vision is to have a **good and hopefully robust engine** that you can use and have many features out of the box. This is the purpose of this library. 
* Everyone free to PR features they want, I'm not picky.
* I also take feature requests, but don't expect me to jump right onto it. ;)
* **Flash is in the past**. I do not plan to support flash target. If it works - it's a miracle.
* Code style is all over the place, I'll fix it eventually.
* No, seriously, if something from Heeps going to be merged into stock Heaps - it's cool. I just not going to do PRs with such features just to argue with devs about if that's really needed, will it add unnecessary complexity, etc. I'm doing this in my free time and for fun, and arguing kills the mood for me.

## Structure
* `h2d`, `h3d` and `hxd` packages used in the same way as Heaps do - 2D, 3D and general stuff.
* `hxd.heaps` package user for internal Heaps features, like macro functions.
* `hxd.tools` designated for `using hxd.tools.NTools;` that would extend standard functionality of existing Heaps objects.

## Features

For a list of additional classes see `docs`. Listing every single class here is just not feasible at this point.  
Below are specific remarks for some of the features or objects that I still have to document properly.

* `h2d.Tilemap` - Basic Tilemap renderer. NOT PERFECT. It does not care about render-order and shit. I'll fix and improve it eventually, but not now.
* `h3d.scene.S2DPlane` - A 2D plane that renders s2d objects on it. Uses texture rendering. For more primitive approach see `h3d.scene.TileSprite`.
* `hxd.res.GifImage` - Animated gif support with Res. Use `toAnimation` and `toFrames` to get animation data. `toImage` can be used to obtain spritesheet Image.
* `hxd.res.TiledMapFile` - when `format-tiled` library used, replaces `hxd.res.TiledMap` and provides better support for it.
* `ManifestFileSystem` - js-oriented alternative to stupid embedding into .js file. More tricky to operate, and requires preloading. See below.

### Type Patching
Library also uses ~~black magic~~ macros to patch some base classes.  
Can be disabled by `-D heeps_disable_patch_<path>`. E.g. `-D heaps_disable_patch_h2d_object` for `h2d.Object` patch. Note that some patches may rely on others. Alternatively `-D heeps_disable_patch` will disable all type patching.
* `h2d.Object` - Added `originX` and `originY` that control origin offset for transofmrations. `transform` and `absoluteTransform` for overriding transform matrix with custom one.
* `h2d.Layers` - Added `moveChild`, `addAt` and `getChildLayerIndex` for more control over children positioning.

### Tiled support

Library provides rudimentary Tiled map editor integration. It's integration far from complete, and may be improved.  
Tiled .tmx maps can be accessed via `hxd.Res` and parsed with `toMap()` function. Heeps will try to resolve all dependencies automatically, however it can be done manually afterwards.  
Returned object will contain parsed `format.tmx.TiledMap` and a list of loaded tilesets that can be used to get `h2d.Tile` instances with ease.

In future `h2d.tiled` will contain helper classes that would provide ability to visualise Tiled maps.
Currently there is only a draft version of layer renderer for Tiled utilizing `h2d.SpriteBatchExt`. But it is limited to 8 unique textures per layer as well as not supports render-order (uses default right-bottom) and orientation other than orthogonal.

### Manifest FS
Since we are sane people and don't want 50+MB js file that contains Base64-encoded game assets, we obviously want to load those files separately. Manifest-FS provides ability to load those files from a manifest file.  
This approach requires some prep-work to get it running, but beats embedding everything in JS.  
First, you have to generate manifest file with `hxd.fs.ManifestBuilder.create`, `generate` and `initManifest`. Last acts exactly the same as `Res.init*` functions and bakes manifest into the code. First just generates manifest FS in macro call and second one just does convert and optionally saves manifest to your res folder. I trust people here are smart enough to figure out how to utilize those two, so I'll focus on one I use, e.g. `initManifest` method.
> I should note that all docs about resource management make you believe that you have to initialize them in `main`. THIS. IS. WRONG. Instead, you need to initialize them by overriding `hxd.App.loadAssets` and call `onLoaded` after everything's loaded. We're good? Good.

> Todo: Fix an example, because it doesn't work.

`initManifest` does not create typical `Loader` instance. Instead, it creates `hxd.res.ManifestLoader`, which you then should populate with progress handlers and call `loadManifestFiles`. Here's an example code of how you do this:
```haxe
override private function loadAssets(onLoaded) {
  var loader:hxd.res.ManifestLoader = hxd.fs.ManifestBuilder.initManifest();
  loader.onLoaded = () -> { trace("All loaded!"); onLoaded(); }
  loader.onFileLoadStarted = (f) -> trace("Started loading file: " + f.path);
  loader.onFileLoaded = (f) -> trace("Finished loading file: " + f.path);
  // This only happens when you use JS target, since sys target is synchronous.
  loader.onFileProgress = (f, loaded, total) -> trace("Loading file progress: " + f.path + ", " + loaded + "/" + total);
  loader.loadManifestFiles();
}
```
Additionally, if you want to visualize loading progress, but don't want anything fancy - there is `h2d.ui.ManifestProgress` with primitive progress-bar if you don't need any fancy mumbo-jumbo and just want your resources loaded while showing player that it actually loads.

## Extra flags
* `-D gif_disable_margin` - Disables 1px top/bottom margin that avoids pixel bleeding for gif spritesheet generation.