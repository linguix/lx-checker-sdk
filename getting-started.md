# Linguix Checker SDK

## Table of Contents

- [Installation](#installation)
- [Basic Usage](#basic-usage)
  - [Initializing the SDK](#initializing-the-sdk)
  - [Connecting Elements](#connecting-elements)
    - [JavaScript API](#option-1-javascript-api)
    - [HTML Wrapper](#option-2-html-wrapper)
- [Cleanup](#cleanup)
- [Advanced: Split Architecture](#advanced-split-architecture)
- [Custom Elements Support](#custom-elements-support)
- [Usage Without npm/Node.js (CDN/Script Tag)](#usage-without-npmnodejs-cdnscript-tag)
- [Browser Extension Example](#browser-extension-example)

## Installation

You can use the Linguix Checker SDK in two ways:

### 1. Using npm (Recommended for modern web apps)

```bash
npm install @linguix.com/lx-checker-sdk
```

### 2. Using CDN/Script Tag (For any HTML, PHP/Laravel, Jinja, or static site)

> **Best Practice:** Place the `<script src="..."></script>` and the initialization code at the **end of your HTML body** (just before `</body>`). This ensures the DOM is loaded and improves page load performance.

Add this to your HTML or template:

```html
<!-- Place these just before </body> -->
<script src="https://cdn.jsdelivr.net/npm/@linguix.com/lx-checker-sdk/dist/bundle.min.js"></script>
<script>
  window.Linguix.LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });
  window.Linguix.LinguixCheckerSDK.attachToElement(document.querySelector('textarea'));
</script>
```

This will expose the SDK as `window.Linguix` for use in your scripts.

## Basic Usage

### Initializing the SDK

```javascript
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';

// Initialize with your API key
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });

// Or initialize with custom configuration
LinguixCheckerSDK.initialize({
  apiKey: 'your-api-key',
  url: 'https://your-custom-api-endpoint.com',
  options: {
    query: {
      clientToken: 'optional-client-token'
    }
  },
  language: 'en-US' // Force specific language instead of automatic detection
});
```

> **Security Note:** For production applications, consider using a proxy server to keep your API key secure. See the [Proxy Server Guide](proxy-server.md) for details.

> **Language Support:** By default, Linguix automatically detects the language of the text being checked, supporting 30+ most popular languages. If you need to force a specific language, you can use the `language` option, but note that manual language forcing is limited to: 'en-US', 'en-GB', 'en-ZA', 'en-CA', 'en-AU', 'en-NZ', 'pt-PT', 'pt-BR', 'de-DE', 'fr', 'pl-PL', 'es', 'it'.

### Connecting Elements

#### Option 1: JavaScript API

```javascript
// For textarea elements
const textarea = document.querySelector('textarea');
LinguixCheckerSDK.attachToElement(textarea);

// For contenteditable elements
const editor = document.querySelector('[contenteditable="true"]');
LinguixCheckerSDK.attachToElement(editor);
```

#### Option 2: HTML Wrapper

Wrap elements with `<linguix-checkable>` tags for automatic initialization:

```html
<linguix-checkable>
  <textarea></textarea>
</linguix-checkable>

<linguix-checkable>
  <div contenteditable="true"></div>
</linguix-checkable>
```

### Cleanup

```javascript
// Detach from a specific element
LinguixCheckerSDK.detachFromElement(element);

// Completely destroy SDK instance
LinguixCheckerSDK.destroy();
```

## Advanced: Split Architecture

The SDK supports split architecture where UI components run separately from networking/processing components. This is useful for:

- Browser extensions (content script + background script)
- Web applications with service workers
- Performance optimization for large documents

> **Note:** The SDK uses WebSockets for communication with Linguix servers. If you need HTTP transport instead, please contact us at hi@linguix.com.

### Background Component Setup

```javascript
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';
import { YourBackgroundMessenger } from './your-messenger';

// Initialize background component
const messenger = new YourBackgroundMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);
```

### Service Worker Environment

For background scripts running in a service worker environment (like browser extensions' background scripts) where browser APIs may not be available, use the worker-specific import:

```javascript
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk/worker';
import { YourBackgroundMessenger } from './your-messenger';

// Initialize background component in service worker
const messenger = new YourBackgroundMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);

// Or with custom configuration for proxy server
LinguixCheckerSDK.initialize({
  url: 'http://your-proxy-server.com:3000',
  options: {
    query: {
      clientToken: 'some-token'
    }
  }
}, messenger);
```

The worker import is a drop-in replacement that provides the same API but is optimized for environments without browser DOM APIs.

### Content Component Setup

```javascript
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';
import { YourContentMessenger } from './your-messenger';

// Initialize content component
const messenger = new YourContentMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);

// Attach to elements as usual
const textarea = document.querySelector('textarea');
LinguixCheckerSDK.attachToElement(textarea);
```

### Messenger Interfaces

Create custom messengers implementing these interfaces:

```typescript
interface ILinguixBackgroundMessenger {
    sendToContent(message: ILinguixMessage): void;
    onContentMessage(callback: (message: ILinguixMessage) => void): void;
    destroy(): void;
}

interface ILinguixContentMessenger {
    sendToBackground(message: ILinguixMessage): void;
    onBackgroundMessage(callback: (message: ILinguixMessage) => void): void;
    destroy(): void;
}

interface ILinguixMessage {
    type: string;
    id: string;
    payload?: any;
}
```

## Custom Elements Support

The SDK uses custom elements (`linguix-highlighter` and `linguix-alert`) for rendering UI components. For environments that don't support custom elements natively (like some Chrome extension content scripts), the SDK automatically loads the `@webcomponents/custom-elements` polyfill.

No additional configuration is needed - the polyfill is initialized when the SDK is loaded. However, if you're seeing errors related to custom elements not being defined, ensure that:

1. The polyfill is being loaded before any custom elements are used
2. Your bundler is correctly including the polyfill
3. There are no Content Security Policy (CSP) restrictions preventing the polyfill from working

If you need to manually initialize the polyfill earlier, you can import it directly:

```javascript
import '@webcomponents/custom-elements';
```

## Usage Without npm/Node.js (CDN/Script Tag)

You can use the Linguix Checker SDK in any server-rendered environment (PHP, Laravel Blade, Python Jinja, etc.) or static HTML by loading it from a CDN.

### 1. Add the SDK via CDN

> **Best Practice:** Place the `<script src="..."></script>` and the initialization code at the **end of your HTML body** (just before `</body>`).

```html
<!-- Place these just before </body> -->
<script src="https://cdn.jsdelivr.net/npm/@linguix.com/lx-checker-sdk/dist/bundle.min.js"></script>
<script>
  window.Linguix.LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });
  window.Linguix.LinguixCheckerSDK.attachToElement(document.querySelector('textarea'));
</script>
```

This will expose the SDK as `window.Linguix` for use in your scripts.

### 2. Initialize in a Script Tag

See above for best practice placement.

### 3. Example: Laravel Blade

```blade
<!-- resources/views/example.blade.php -->
<textarea></textarea>
<!-- Place these just before </body> -->
<script src="https://cdn.jsdelivr.net/npm/@linguix.com/lx-checker-sdk/dist/bundle.min.js"></script>
<script>
  window.Linguix.LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });
  window.Linguix.LinguixCheckerSDK.attachToElement(document.querySelector('textarea'));
</script>
```

### 4. Example: Jinja Template

```jinja
<!-- templates/example.html -->
<textarea></textarea>
<!-- Place these just before </body> -->
<script src="https://cdn.jsdelivr.net/npm/@linguix.com/lx-checker-sdk/dist/bundle.min.js"></script>
<script>
  window.Linguix.LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });
  window.Linguix.LinguixCheckerSDK.attachToElement(document.querySelector('textarea'));
</script>
```

### 5. Using <linguix-checkable> (No JS Needed)

You can use the custom element directly for automatic initialization:

```html
<linguix-checkable>
  <textarea></textarea>
</linguix-checkable>
<!-- Place this just before </body> -->
<script src="https://cdn.jsdelivr.net/npm/@linguix.com/lx-checker-sdk/dist/bundle.min.js"></script>
```

> **Security Note:** The API key will be visible in the page source. For production, consider using a proxy server to keep your API key secure. See the [Proxy Server Guide](proxy-server.md) for details.

## Browser Extension Example

For a minimal browser extension implementation:

```javascript
// background.js
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';

class ExtensionBackgroundMessenger {
    constructor() {
        // Listen for messages from content scripts
        browser.runtime.onMessage.addListener((message, sender) => {
            if (this.messageCallback) {
                // Add any extension-specific properties to the message
                // These will be automatically preserved and passed through
                const augmentedMessage = { ...message, tabId: sender.tab.id };
                this.messageCallback(augmentedMessage);
            }
        });
    }

    sendToContent(message) {
        // Use any message properties that were preserved from the original message
        // The SDK will automatically pass through all properties
        if (message && message.tabId) {
            browser.tabs.sendMessage(message.tabId, message);
        }
    }

    onContentMessage(callback) {
        this.messageCallback = callback;
    }

    destroy() {
        this.messageCallback = null;
    }
}

// Initialize background component
const messenger = new ExtensionBackgroundMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);
```

```javascript
// content.js
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';

class ExtensionContentMessenger {
    constructor() {
        // Listen for messages from background script
        browser.runtime.onMessage.addListener(message => {
            if (this.messageCallback) {
                this.messageCallback(message);
            }
        });
    }

    sendToBackground(message) {
        browser.runtime.sendMessage(message);
    }

    onBackgroundMessage(callback) {
        this.messageCallback = callback;
    }

    destroy() {
        this.messageCallback = null;
    }
}

// Initialize content component
const messenger = new ExtensionContentMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);

let currentElement = null;
document.addEventListener('focusin', (event) => {
    const element = event.target;
    
    if (element instanceof HTMLTextAreaElement || 
        (element instanceof HTMLElement && element.isContentEditable)
        && currentElement !== element) {
        
        if (currentElement) {
            LinguixCheckerSDK.detachFromElement(currentElement);
        }
        
        LinguixCheckerSDK.attachToElement(element);
        currentElement = element;
    }
}, true);
```

The SDK uses a generic message property passthrough mechanism, meaning any properties added to messages (like `tabId` or other extension-specific data) will be automatically preserved throughout the message flow. This allows for flexible integration with extension messaging systems without requiring changes to the SDK's core message handling.
