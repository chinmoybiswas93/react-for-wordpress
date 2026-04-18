# React for WordPress Plugin Development — Documentation

> **Purpose:** Comprehensive reference for building blocks, block editor extensions, and wp-admin React screens.  
> **Level:** Mid-level developer with JavaScript background, learning the WordPress/React ecosystem.

## Quick Navigation

- **[Study Order](./STUDY_ORDER.md)** — Recommended learning path
- **[Quick Reference](./QUICK_REFERENCE.md)** — Cheatsheet and common patterns

## Core Topics

### Foundations
1. **[Modern JavaScript (ES6+)](./01-modern-javascript.md)**
   - ES Modules, Destructuring, Spread, Arrow Functions, Template Literals
   - Optional Chaining, Nullish Coalescing, Array Methods
   - Promises and async/await, JSON

2. **[React with @wordpress/element](./02-react-with-wordpress-element.md)**
   - JSX, Fragments, Conditional UI
   - Lists and Keys, Function Components
   - Controlled Inputs, Mounting Admin UIs

### React Patterns
3. **[React Hooks (Core)](./03-react-hooks-core.md)**
   - useState, useEffect, useLayoutEffect
   - useRef, useMemo, useCallback, useId
   - useReducer, Stale Closures

4. **[Composition and Architecture](./04-composition-and-architecture.md)**
   - Splitting Components, Lifting State
   - Custom Hooks, useContext + createContext
   - Portals, Error Boundaries

### Gutenberg Block Development
5. **[Gutenberg Blocks — Registration and Metadata](./05-gutenberg-blocks-registration.md)**
   - registerBlockType + block.json
   - Attributes, supports, Block Styles
   - Block Variations, Dynamic Blocks, deprecated

6. **[Block Editor UI (@wordpress/block-editor)](./06-block-editor-ui.md)**
   - useBlockProps, useBlockProps.save
   - InnerBlocks + useInnerBlocksProps
   - InspectorControls, BlockControls
   - RichText, MediaUpload/MediaUploadCheck

### Editor and Data Management
7. **[Editor Chrome and Data (@wordpress/data)](./07-editor-chrome-and-data.md)**
   - useSelect and useDispatch
   - Core Stores — Quick Reference
   - core-data — Entities

8. **[@wordpress/components — Common UI](./08-wordpress-components.md)**
   - Text and Choice Controls
   - Numeric Controls, Color Controls
   - Overlays — Modal and Popover

### Plugin Features
9. **[Internationalization (@wordpress/i18n)](./09-internationalization.md)**
   - Translation functions: __, _x, _n, sprintf
   - Generating POT files

10. **[REST API and Settings (@wordpress/api-fetch)](./10-rest-api-and-settings.md)**
    - Basic Usage — GET, POST, DELETE
    - Custom REST Routes, Plugin Options
    - Error Handling

11. **[Icons and Media](./11-icons-and-media.md)**
    - @wordpress/icons components
    - Responsive Preview modes

### Styling and Building
12. **[Styling for Plugins](./12-styling-for-plugins.md)**
    - Scoped CSS — Avoiding Clashes
    - Editor vs Front-end Styles
    - CSS Custom Properties, RTL Support

13. **[Build Tooling (@wordpress/scripts)](./13-build-tooling.md)**
    - Setup and Entry Points
    - .asset.php and Enqueueing
    - Webpack Externals

14. **[Scripts Outside React](./14-scripts-outside-react.md)**
    - view.js — Front-end Vanilla JS
    - Event Delegation
    - WordPress Interactivity API

### Advanced Topics
15. **[Security and Content Safety](./15-security-and-safety.md)**
    - Capabilities — current_user_can
    - Nonces, Sanitization vs Validation
    - React and Escaping

16. **[Optional / Advanced](./16-optional-advanced.md)**
    - SlotFill — Extending the Editor
    - Custom Stores
    - Performance — Avoiding Unnecessary Re-renders

## File Structure

```
docs/
├── README.md (this file)
├── STUDY_ORDER.md
├── QUICK_REFERENCE.md
├── 01-modern-javascript.md
├── 02-react-with-wordpress-element.md
├── 03-react-hooks-core.md
├── 04-composition-and-architecture.md
├── 05-gutenberg-blocks-registration.md
├── 06-block-editor-ui.md
├── 07-editor-chrome-and-data.md
├── 08-wordpress-components.md
├── 09-internationalization.md
├── 10-rest-api-and-settings.md
├── 11-icons-and-media.md
├── 12-styling-for-plugins.md
├── 13-build-tooling.md
├── 14-scripts-outside-react.md
├── 15-security-and-safety.md
├── 16-optional-advanced.md
├── STUDY_ORDER.md
└── QUICK_REFERENCE.md
```

## How to Use This Documentation

1. **New to React + WordPress?** Start with the [Study Order](./STUDY_ORDER.md)
2. **Need a specific answer?** Use the [Quick Reference](./QUICK_REFERENCE.md) or search the topic list above
3. **Deep dive on a topic?** Click into the full documentation file
4. **Cross-references?** Each topic links to related sections

## Key Resources

- [Official WordPress Block Editor Handbook](https://developer.wordpress.org/block-editor/)
- [@wordpress/element documentation](https://developer.wordpress.org/@wordpress/element)
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)
- [WordPress Plugin Handbook](https://developer.wordpress.org/plugins/)

---

*Each section builds on previous knowledge. Revisit sections after shipping features that use them.*