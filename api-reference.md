# Linguix Checker SDK API Reference

This document provides a reference for the public API of the Linguix Checker SDK.

## LinguixCheckerSDK

### Methods

#### `initialize(apiKey: string, messenger?: ILinguixMessenger): void`

Initializes the SDK with the provided API key and optional messenger.

**Parameters:**
- `apiKey` (string): Your Linguix API key
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
