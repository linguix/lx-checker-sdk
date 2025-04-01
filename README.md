# Linguix Checker SDK Documentation

## Overview

Linguix Checker SDK provides grammar and spell checking functionality for web applications and browser extensions. The SDK can check text in textareas and contenteditable elements, highlighting errors and offering suggestions for correction.

## Features

- Grammar and spelling error detection
- Contextual correction suggestions
- Support for textarea and contenteditable elements
- HTML and JavaScript integration options
- Split architecture for browser extensions
- Customizable styling

## Documentation

- [Getting Started](getting-started.md) - Basic setup and usage guide
- [API Reference](api-reference.md) - Detailed method and interface reference
- [Styling Guide](styling.md) - Customization options for UI components
- [Next.js Integration](nextjs.md) - Guide for using the SDK with Next.js
- [Callbacks](docs/callbacks.md) - Documentation for event callbacks
- [Changelog](CHANGELOG.md) - Version history and updates

## Quick Start

```javascript
// Install
npm install @linguix.com/lx-checker-sdk

// Import
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';

// Initialize
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });

// Or with language forcing (when you know what language your users will use)
LinguixCheckerSDK.initialize({ 
  apiKey: 'your-api-key',
  language: 'en-US'
});

// Attach to elements
const textarea = document.querySelector('textarea');
LinguixCheckerSDK.attachToElement(textarea);

// Or wrap elements with <linguix-checkable> tags
<linguix-checkable>
  <textarea></textarea>
</linguix-checkable>
```

## Advanced Usage

The SDK supports a split architecture where UI and networking components run in separate contexts, ideal for browser extensions:

```javascript
// Background script
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';
import { YourBackgroundMessenger } from './messenger';

const messenger = new YourBackgroundMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);

// Content script
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';
import { YourContentMessenger } from './messenger';

const messenger = new YourContentMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);

// Attach to elements as usual
const textarea = document.querySelector('textarea');
LinguixCheckerSDK.attachToElement(textarea);
```
