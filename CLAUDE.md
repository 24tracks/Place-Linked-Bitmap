# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Place Linked Bitmap is a **Sketch plugin** (for Sketch 43+) that lets users place external bitmap files (JPG, PNG, PSD, etc.) into Sketch documents and update them later when the source files change. It uses Sketch's CocoaScript plugin API with Objective-C bridge calls.

## Repository Structure

```
Place Linked Bitmap.sketchplugin/
  Contents/
    Sketch/
      manifest.json            # Plugin metadata, command definitions, keyboard shortcuts
      handlers.cocoascript     # Entry-point command handlers (imports PlaceLinkedBitmap.js)
      PlaceLinkedBitmap.js     # Core library object with all bitmap operations
appcast.xml                    # Sparkle-based auto-update feed
```

There is no build step. The `.sketchplugin` bundle is the plugin itself, loaded directly by Sketch.

## Architecture

- **`PlaceLinkedBitmap.js`** — A single `PlaceLinkedBitmap` object containing all helper methods: file path manipulation (relative/absolute conversion, URL encoding/decoding), bitmap layer creation, image fill application, layer querying, NSOpenPanel wrappers, and Sketch document utilities.
- **`handlers.cocoascript`** — Defines the four command handler functions that Sketch calls (registered in `manifest.json`). These are thin wrappers that get the document context and delegate to `PlaceLinkedBitmap` methods.
- **`manifest.json`** — Declares plugin identity (`io.kolo.sketch.place-linked-bitmap`), the four commands, menu structure, keyboard shortcuts, and the appcast URL.

## Key Concepts

- **Tagged layers**: The plugin stores the original file path on each placed layer using `[command setValue:forKey:onLayer:]`. The key is `"originalURL"`. This is how layers are identified during updates.
- **Relative paths**: File paths are stored relative to the `.sketch` document's directory when possible (prefixed with `./`), expanded at update time via `expandRelativePath()`.
- **Layer prefix**: Placed layers are named with the prefix `@: ` followed by the filename.
- **Sketch version branching**: `MSImageData` initialization differs for Sketch < 47 vs >= 47 (see `addFillToShapeLayer` and `updateShapeLayer`).

## Plugin Commands

| Command | Handler | Shortcut |
|---|---|---|
| Place Bitmap as New Layer… | `PLB_place_bitmap_as_new_layer` | Ctrl+Cmd+I |
| Place Bitmap as Layer Fill… | `PLB_place_bitmap_as_fill` | — |
| Update All Bitmaps | `PLB_update_bitmaps` | Ctrl+Cmd+U |
| Find All Tagged Layers | `PLB_find_all_tagged_layers` | — |

## Development Notes

- Code uses CocoaScript's Objective-C bridge syntax (e.g., `[[NSImage alloc] initWithContentsOfFile:]`, `[NSOpenPanel openPanel]`).
- To test, double-click the `.sketchplugin` bundle to install it in Sketch, then use the plugin menu.
- The `appcast.xml` powers Sketch's built-in plugin update mechanism via Sparkle.
