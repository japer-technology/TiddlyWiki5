# TiddlyWiki5 Architecture: How It Works and How Single HTML Files Are Created

## Table of Contents
1. [Introduction](#introduction)
2. [Repository Structure](#repository-structure)
3. [Core Concepts](#core-concepts)
4. [The Boot Process](#the-boot-process)
5. [How Single HTML Files Are Created](#how-single-html-files-are-created)
6. [Build System](#build-system)
7. [Module System](#module-system)
8. [Templates and Rendering](#templates-and-rendering)
9. [Command Line Interface](#command-line-interface)
10. [Development Workflow](#development-workflow)

## Introduction

TiddlyWiki5 is a unique non-linear personal web notebook that can operate as a complete interactive wiki in JavaScript. Its most remarkable feature is the ability to package the entire application, including all content and functionality, into a single HTML file that can be opened and edited in a web browser.

This document explains the architecture of TiddlyWiki5 and the process by which it creates these self-contained single HTML files.

## Repository Structure

The TiddlyWiki5 repository is organized into several key directories:

### Core Directories

- **`/boot`** - Contains the boot kernel (`boot.js`) that initializes TiddlyWiki
  - `boot.js` - The main boot kernel that creates the barebones TW environment
  - `bootprefix.js` - Initializes the `$tw` global object
  - `boot.css.tid` - Basic CSS styles used before the parsing engine boots

- **`/core`** - The core plugin containing essential functionality
  - `/modules` - Core JavaScript modules organized by functionality
  - `/templates` - Core templates for rendering HTML, JSON, CSS, etc.
  - `/ui` - User interface components
  - `/language` - Internationalization files
  - `/images` - SVG icons and images

- **`/core-server`** - Server-side functionality for Node.js
  - `/commands` - Command-line commands (build, render, save, listen, etc.)

- **`/plugins`** - Official and third-party plugins
  - Each plugin extends TiddlyWiki with additional features

- **`/themes`** - Visual themes for TiddlyWiki

- **`/languages`** - Translation files for different languages

- **`/editions`** - Different configurations of TiddlyWiki
  - `empty` - Minimal empty wiki
  - `tw5.com` - The main tiddlywiki.com website
  - `prerelease` - Prerelease version with additional plugins
  - `dev` - Developer documentation
  - Each edition contains a `tiddlywiki.info` file defining its configuration

- **`/bin`** - Build scripts for generating distributions

## Core Concepts

### Tiddlers

The fundamental unit of content in TiddlyWiki is called a "tiddler." A tiddler is similar to a note card or a wiki page and consists of:
- A title (unique identifier)
- A body (the content)
- Metadata fields (tags, type, created/modified dates, etc.)

### WikiText

TiddlyWiki uses its own markup language called WikiText, which is similar to other wiki markup languages but with its own syntax and features. WikiText is parsed and rendered by the core engine.

### Widgets

TiddlyWiki's user interface is built using a widget tree. Widgets are JavaScript objects that can:
- Render themselves to the DOM
- Update themselves when data changes
- Handle user interactions
- Contain child widgets

### Filters

Filters are expressions used to select and manipulate sets of tiddlers. They use a powerful declarative syntax like:
- `[tag[MyTag]]` - Select tiddlers with tag "MyTag"
- `[is[system]]` - Select system tiddlers
- `[!is[system]sort[title]]` - Non-system tiddlers sorted by title

## The Boot Process

The boot process is the sequence by which TiddlyWiki initializes itself, whether in a browser or Node.js environment:

### 1. Initial Setup (bootprefix.js)

The `bootprefix.js` file creates the `$tw` global object that serves as the namespace for all TiddlyWiki functionality:

```javascript
var $tw = {
    boot: {},
    utils: {},
    modules: {},
    wiki: null
    // ... other properties
};
```

### 2. Boot Kernel (boot.js)

The `boot.js` file is the main boot kernel. It:

1. **Detects the environment** (browser vs. Node.js)
2. **Loads the core modules** from the embedded store
3. **Creates the wiki store** - The central repository for tiddlers
4. **Executes startup modules** - Modules tagged with `module-type: startup`
5. **Initializes the UI** (in browser) or processes commands (in Node.js)

The boot process in a browser follows this sequence:

```
bootprefix.js (creates $tw object)
    ↓
boot.js (main boot kernel)
    ↓
Load embedded tiddlers from store area
    ↓
Decrypt encrypted tiddlers (if needed)
    ↓
Execute startup modules
    ↓
Render the page
```

### 3. Module Loading

TiddlyWiki uses a modular architecture where functionality is divided into modules. Each module has a `module-type` field that categorizes its purpose:

- `startup` - Runs during initialization
- `command` - Command-line command
- `widget` - UI widget
- `filter` - Filter operator
- `saver` - Mechanism for saving changes
- `parser` - Content parser
- And many more...

### 4. Startup Sequence

Startup modules can specify dependencies using `before` and `after` arrays, creating a dependency graph that determines execution order. Common startup modules include:

- `load-modules` - Loads all JavaScript modules
- `rootwidget` - Creates the root widget
- `render` - Renders the page to the DOM
- `story` - Manages the story river (list of open tiddlers)

## How Single HTML Files Are Created

The creation of a single HTML file is one of TiddlyWiki's most impressive features. This section explains the process in detail.

### The Single File Structure

A TiddlyWiki HTML file contains:

1. **HTML document structure** (DOCTYPE, html, head, body tags)
2. **Metadata** (charset, viewport, version info)
3. **Boot CSS** - Basic styles used before JavaScript runs
4. **Store area** - The tiddlers encoded as JSON or DIV elements
5. **Library modules** - System JavaScript libraries
6. **Boot kernel** - The boot.js and bootprefix.js code
7. **Static content** - Fallback content for when JavaScript is disabled

### Template-Based Generation

The single HTML file is generated using the template system. The key template is:

**`$:/core/templates/tiddlywiki5.html`** - The main HTML template that defines the structure

This template includes several sub-templates:

- `$:/core/templates/store.area.template.html` - Embeds tiddlers in the page
- `$:/core/templates/css-tiddler` - Renders CSS tiddlers
- `$:/core/templates/javascript-tiddler` - Renders JavaScript tiddlers
- `$:/core/templates/version` - Includes version information

### The Store Area

The store area is where all tiddlers are embedded. TiddlyWiki supports two formats:

#### JSON Format (Default in v5.2.0+)

```html
<script class="tiddlywiki-tiddler-store" type="application/json">
[
  {
    "title": "MyTiddler",
    "text": "Content here",
    "tags": "[[Tag1]] [[Tag2]]",
    "created": "20231123120000000",
    "modified": "20231123120000000"
  },
  ...
]
</script>
```

#### DIV Format (Legacy)

```html
<div id="storeArea" style="display:none;">
  <div title="MyTiddler" tags="[[Tag1]] [[Tag2]]">
    <pre>Content here</pre>
  </div>
  ...
</div>
```

### The Save Filter

The `saveTiddlerFilter` determines which tiddlers are included in the saved file. It's defined in the save template and typically excludes:

- Temporary tiddlers (prefixed with `$:/temp/`)
- Popup state tiddlers (prefixed with `$:/state/popup/`)
- Pending import tiddlers
- Boot kernel files (since they're included separately)
- System JavaScript libraries with library[yes] field

The default filter looks like:
```
[is[tiddler]] -[prefix[$:/state/popup/]] -[prefix[$:/temp/]] -[prefix[$:/HistoryList]] -[status[pending]plugin-type[import]] -[[$:/boot/boot.css]] -[is[system]type[application/javascript]library[yes]] -[[$:/boot/boot.js]] -[[$:/boot/bootprefix.js]] +[sort[title]]
```

### Self-Modification

When running in a browser, TiddlyWiki can modify itself and save changes. The process works like this:

1. User makes changes (edit tiddlers, add tags, etc.)
2. Changes are stored in the wiki store (in memory)
3. User clicks "Save" button
4. **Saver modules** detect the best save mechanism:
   - Direct file system access (if available)
   - Download API (prompts user to save file)
   - TiddlyFox extension
   - TiddlyIE extension
   - Or other custom savers
5. The entire HTML file is regenerated with updated tiddlers
6. The file is saved, replacing the old version

## Build System

### Build Commands

TiddlyWiki uses a command-based build system. Commands are defined in the `/core-server/commands` directory.

#### Key Commands

**`--build <target>`**
Executes a build target defined in `tiddlywiki.info`. Example build targets from `editions/empty/tiddlywiki.info`:

```json
{
  "build": {
    "index": [
      "--render", "$:/core/save/all", "index.html", "text/plain"
    ],
    "empty": [
      "--render", "$:/core/save/all", "empty.html", "text/plain",
      "--render", "$:/core/save/all", "empty.hta", "text/plain"
    ]
  }
}
```

**`--render <tiddler> <filename> <type> [<template>]`**
Renders a tiddler to a file. When rendering `$:/core/save/all`:
1. Loads the `$:/core/templates/tiddlywiki5.html` template
2. Processes all the embedded transclusions and filters
3. Generates the complete HTML with all tiddlers embedded
4. Writes to the specified output file

**`--listen [port]`**
Starts an HTTP server that serves the wiki and allows editing through the browser.

**`--output <directory>`**
Sets the output directory for subsequent commands.

### Edition Configuration (tiddlywiki.info)

Each edition has a `tiddlywiki.info` file that defines:

```json
{
  "description": "Edition description",
  "plugins": [
    "tiddlywiki/markdown",
    "tiddlywiki/highlight"
  ],
  "themes": [
    "tiddlywiki/vanilla",
    "tiddlywiki/snowwhite"
  ],
  "build": {
    "index": [
      "--render", "$:/core/save/all", "index.html", "text/plain"
    ]
  }
}
```

### Building from Command Line

To create a single HTML file:

```bash
# Basic command
tiddlywiki <edition> --build index

# Example: Build empty edition
tiddlywiki editions/empty --output ./output --build index

# Custom build with specific commands
tiddlywiki editions/empty --output ./output \
  --render "$:/core/save/all" "index.html" "text/plain"

# Build with a different edition
tiddlywiki editions/prerelease --output ./dist --build index
```

### The Build Process Step-by-Step

1. **Initialize** - TiddlyWiki boots with specified edition
2. **Load plugins** - Plugins specified in `tiddlywiki.info` are loaded
3. **Load tiddlers** - All tiddler files from the edition are loaded
4. **Execute commands** - Build commands are executed in sequence
5. **Render template** - The save template is rendered
6. **Embed tiddlers** - All tiddlers are embedded in the store area
7. **Embed boot code** - Boot kernel JavaScript is embedded
8. **Embed library modules** - System libraries are embedded
9. **Write file** - The complete HTML is written to disk

## Module System

### Module Types

TiddlyWiki's module system is central to its extensibility. Key module types:

- **`startup`** - Initialization code
- **`command`** - CLI commands
- **`widget`** - UI components
- **`filter`** - Filter operators
- **`saver`** - Save mechanisms
- **`parser`** - Content parsers (WikiText, HTML, etc.)
- **`macro`** - Reusable template code
- **`library`** - Third-party libraries (jQuery, etc.)
- **`utils`** - Utility functions
- **`route`** - HTTP route handlers (server mode)

### Module Definition

Modules are typically JavaScript files with special header comments:

```javascript
/*\
title: $:/core/modules/widgets/button.js
type: application/javascript
module-type: widget

Button widget

\*/
(function(){

"use strict";

var Widget = require("$:/core/modules/widgets/widget.js").widget;

var ButtonWidget = function(parseTreeNode,options) {
    this.initialise(parseTreeNode,options);
};

// Inherit from the base widget class
ButtonWidget.prototype = new Widget();

// Render this widget into the DOM
ButtonWidget.prototype.render = function(parent,nextSibling) {
    // Implementation...
};

exports.button = ButtonWidget;

})();
```

### Module Loading

In the browser:
1. Modules are embedded in the HTML as `<script>` tags
2. Boot kernel extracts and registers them
3. `require()` function loads dependencies

In Node.js:
1. Modules are loaded from filesystem
2. Standard Node.js `require()` is used
3. Additional modules can be added via plugins

## Templates and Rendering

### Template System

TiddlyWiki uses its own template system based on WikiText. Templates can:
- Transclude other tiddlers using `{{TiddlerTitle}}`
- Use variables with `<<variableName>>`
- Apply filters with `{{{ [filter[expression]] }}}`
- Execute widgets with `<$widget>...</$widget>`

### Key Templates

**Save Templates:**
- `$:/core/save/all` - Main entry point for saving complete wiki
- `$:/core/templates/tiddlywiki5.html` - HTML structure template
- `$:/core/templates/store.area.template.html` - Store area template

**Render Templates:**
- `$:/core/ui/PageTemplate` - Main page layout
- `$:/core/ui/ViewTemplate` - How individual tiddlers are displayed
- `$:/core/ui/EditTemplate` - Tiddler editing interface

### Widget Rendering

The rendering process:

1. **Parse** - WikiText is parsed into a parse tree
2. **Widget tree** - Parse tree is converted to widget tree
3. **Render** - Widgets render themselves to DOM nodes
4. **Refresh** - When data changes, widgets selectively update

Example widget tree for `<$list filter="[tag[Example]]">`:
```
ListWidget
  ├─ TextWidget ("Item 1")
  ├─ TextWidget ("Item 2")
  └─ TextWidget ("Item 3")
```

## Command Line Interface

### Basic Usage

```bash
# Display version
tiddlywiki --version

# Initialize a new wiki
tiddlywiki mynewwiki --init server

# Start a server
tiddlywiki mynewwiki --listen

# Build static HTML
tiddlywiki editions/empty --output ./dist --build index

# Render specific tiddler
tiddlywiki editions/empty --render "[all[tiddlers]]" "output.html" "text/html"

# Load and then build
tiddlywiki editions/empty --load mydata.json --output ./dist --build index
```

### Command Chaining

Commands are processed left to right:

```bash
tiddlywiki editions/empty \
  --output ./dist \
  --render "$:/core/save/all" "full.html" "text/plain" \
  --render "$:/core/save/all" "index.html" "text/plain"
```

### Plugin Loading

Load additional plugins:

```bash
# Named plugin
tiddlywiki +plugins/tiddlywiki/markdown mywiki --listen

# Plugin from path
tiddlywiki ++./my-custom-plugin mywiki --listen
```

## Development Workflow

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/TiddlyWiki/TiddlyWiki5.git
   cd TiddlyWiki5
   ```

2. **Install dependencies** (optional, for linting)
   ```bash
   npm install
   ```

3. **Start development server**
   ```bash
   npm run dev
   # Or manually:
   node tiddlywiki.js editions/tw5.com-server --listen
   ```

4. **Build distributions**
   ```bash
   # Build empty edition
   node tiddlywiki.js editions/empty --output ./output --build index
   
   # Build with custom options
   node tiddlywiki.js editions/prerelease --output ./dist --build index
   ```

### Testing Changes

1. **Edit core files** in `/core/modules`, `/core/templates`, etc.
2. **Reload** the development server or rebuild
3. **Test** in browser
4. **Run tests**
   ```bash
   npm test
   ```

### Creating Custom Builds

To create a custom edition:

1. Create edition directory:
   ```bash
   mkdir -p editions/mycustom/tiddlers
   ```

2. Create `tiddlywiki.info`:
   ```json
   {
     "description": "My Custom Edition",
     "plugins": [
       "tiddlywiki/highlight",
       "tiddlywiki/markdown"
     ],
     "themes": [
       "tiddlywiki/vanilla"
     ],
     "build": {
       "index": [
         "--render", "$:/core/save/all", "index.html", "text/plain"
       ]
     }
   }
   ```

3. Add custom tiddlers to `tiddlers/` directory

4. Build:
   ```bash
   node tiddlywiki.js editions/mycustom --output ./output --build index
   ```

### Build Scripts

The `/bin` directory contains useful build scripts:

- **`build-site.sh`** - Builds the complete tiddlywiki.com website
- **`serve.sh`** - Starts a development server
- **`test.sh`** - Runs tests
- **`readme-bld.sh`** - Generates README files from tiddlers

## Advanced Topics

### External JavaScript Core

TiddlyWiki can be split into two files:
1. The JavaScript core (`tiddlywikicore-5.x.x.js`)
2. The HTML file with just the tiddlers

This reduces file size when distributing multiple wikis that share the same core.

Build command:
```bash
tiddlywiki editions/empty --output ./output --build externalcore
```

### Encryption

TiddlyWiki supports encrypting the entire wiki with a password:

1. In browser: Set password via control panel
2. On save: Tiddlers are encrypted using AES
3. On load: Password prompt appears
4. After decryption: Wiki functions normally

### Server Mode

Running TiddlyWiki as a Node.js server:

```bash
tiddlywiki mywiki --listen port=8080
```

Features:
- Real-time synchronization
- Multiple users (with authentication plugins)
- File-based storage on server
- API endpoints for external access

### Static Site Generation

TiddlyWiki can generate static HTML sites:

```bash
tiddlywiki editions/tw5.com --output ./static \
  --rendertiddlers "[!is[system]]" "[encodeuricomponent[]addsuffix[.html]]" \
  "text/plain" "$:/core/templates/static.tiddler.html"
```

This creates individual HTML files for each tiddler, useful for:
- SEO-friendly websites
- Documentation sites
- Blog-style content

## Conclusion

TiddlyWiki5's architecture elegantly solves the challenge of creating a fully-functional web application in a single HTML file. Key innovations include:

1. **Self-contained design** - Everything needed is embedded in one file
2. **Template-based generation** - Flexible system for creating HTML
3. **Modular architecture** - Extensible through plugins
4. **Boot kernel** - Efficient initialization in browser or Node.js
5. **Widget-based UI** - Reactive interface that updates efficiently
6. **Dual-mode operation** - Works standalone or as client-server

The single HTML file generation process leverages:
- WikiText template engine for dynamic content
- JSON or DIV-based store area for tiddlers
- Embedded JavaScript for the boot kernel and modules
- Self-modification capability for saving changes

This architecture makes TiddlyWiki unique: a zero-install, privacy-respecting, portable personal wiki that users truly own and control.

## Additional Resources

- **Official Website**: https://tiddlywiki.com
- **Developer Documentation**: https://tiddlywiki.com/dev
- **GitHub Repository**: https://github.com/TiddlyWiki/TiddlyWiki5
- **Community Forum**: https://talk.tiddlywiki.org
- **GitHub Discussions**: https://github.com/TiddlyWiki/TiddlyWiki5/discussions

## Contributing

To contribute to TiddlyWiki5:

1. Read `contributing.md` for guidelines
2. Sign the Contributor License Agreement (CLA)
3. Submit pull requests via GitHub
4. Follow the coding style and PR conventions
5. Include documentation for new features

For questions or discussions, use GitHub Discussions or the community forum at talk.tiddlywiki.org.
