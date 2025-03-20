# Linguix Checker SDK API Reference

This document provides a reference for the public API of the Linguix Checker SDK.

## LinguixCheckerSDK

### Methods

#### `initialize(config: ILinguixConfig, messenger?: ILinguixMessenger): void`

Initializes the SDK with the provided configuration and optional messenger.

**Parameters:**
- `config` (ILinguixConfig): Configuration object with the following properties:
  - `apiKey` (optional string): Your Linguix API key, can be omitted if custom proxy server is used
  - `url` (optional string): URL of your proxy server
  - `options` (optional object):
    - `query` (optional object): Query parameters to include in requests
      - `clientToken` (optional string): Token for authentication with your proxy server
  - `language` (optional string): Force the checker to use a specific language instead of automatic detection. By default, Linguix supports 30+ most popular languages with automatic detection. Manual language forcing is limited to: 'en-US', 'en-GB', 'en-ZA', 'en-CA', 'en-AU', 'en-NZ', 'pt-PT', 'pt-BR', 'de-DE', 'fr', 'pl-PL', 'es', 'it'
- `messenger` (optional): Custom messenger implementation
  - Pass `ILinguixBackgroundMessenger` to initialize background component
  - Pass `ILinguixContentMessenger` to initialize content component
  - If not provided, initializes both components in the same context

#### `attachToElement(element: SupportedElement): void`

Attaches the checker to a textarea or contenteditable element.

**Parameters:**
- `element`: HTMLTextAreaElement or HTMLElement with contenteditable attribute

#### `detachFromElement(element: SupportedElement): void`

Detaches the checker from an element.

**Parameters:**
- `element`: The element to detach from

#### `destroy(): void`

Destroys the SDK instance and cleans up all resources.

## Custom Elements

### `<linguix-checkable>`

A wrapper element that automatically initializes the grammar checker for contained textarea or contenteditable elements.

**Usage:**
```html
<linguix-checkable>
  <textarea></textarea>
</linguix-checkable>
```

## Interfaces

### ILinguixMessage

```typescript
interface ILinguixMessage {
    type: string;    // Message type
    id: string;      // Unique input field identifier
    payload?: any;   // Optional message data
}
```

### ILinguixContentMessenger

Interface for implementing content-side messaging.

```typescript
interface ILinguixContentMessenger {
    sendToBackground(message: ILinguixMessage): void;
    onBackgroundMessage(callback: (message: ILinguixMessage) => void): void;
    destroy(): void;
}
```

### ILinguixBackgroundMessenger

Interface for implementing background-side messaging.

```typescript
interface ILinguixBackgroundMessenger {
    sendToContent(message: ILinguixMessage): void;
    onContentMessage(callback: (message: ILinguixMessage) => void): void;
    destroy(): void;
}
```

## Transport Layer

The SDK uses WebSockets for communication with Linguix grammar checking services. If you need HTTP transport instead, please contact us at hi@linguix.com.

## Service Worker Support

For background scripts running in service worker environments (like browser extensions' background scripts) where browser DOM APIs are not available, the SDK provides a specialized worker import:

```javascript
import { LinguixCheckerSDK } from '@textly/lx-checker-sdk/worker';
```

This is a drop-in replacement for the standard import that provides the same API but is optimized for environments without browser DOM APIs. When using this import:

1. The messenger parameter is required when initializing the SDK
2. The messenger must implement the `ILinguixBackgroundMessenger` interface
3. Only background component functionality is available (no UI components)

Example usage:

```javascript
import { LinguixCheckerSDK } from '@textly/lx-checker-sdk/worker';

// A messenger implementation is required in worker environments
const messenger = new YourBackgroundMessenger();

// Initialize with API key
LinguixCheckerSDK.initialize({apiKey: 'your-api-key'}, messenger);

// Or with custom configuration
LinguixCheckerSDK.initialize({
  url: 'http://your-proxy-server.com:3000',
  options: {
    query: {
      clientToken: 'some-token'
    }
  }
}, messenger);
```

## Proxy Server

For production applications, you may want to use a proxy server to keep your API key secure. See the [Proxy Server Guide](proxy-server.md) for implementation details.

When using a proxy server, initialize the SDK with the `initialize` method:

```javascript
LinguixCheckerSDK.initialize({
  url: 'http://localhost:3000',
  options: {
    query: {
      clientToken: 'some-token'
    }
  }
});
```
