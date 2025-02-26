# Linguix Checker SDK

The Linguix Checker SDK enables grammar and spell checking integration for web applications and browser extensions.
For styling options, see the [Styling Guide](styling.md).

## Installation

```bash
npm install @linguix.com/lx-checker-sdk
```

## Basic Usage

### Initializing the SDK

```javascript
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';

// Initialize with your API key
LinguixCheckerSDK.initialize('your-api-key');
```

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
LinguixCheckerSDK.initialize('your-api-key', messenger);
```

### Content Component Setup

```javascript
import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';
import { YourContentMessenger } from './your-messenger';

// Initialize content component
const messenger = new YourContentMessenger();
LinguixCheckerSDK.initialize('your-api-key', messenger);

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
LinguixCheckerSDK.initialize('your-api-key', messenger);
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
LinguixCheckerSDK.initialize('your-api-key', messenger);

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
