# Changelog

All notable changes to the Linguix Checker SDK will be documented in this file.

## [1.0.8] - 2025-04-02

### Improved

- **Browser Extension Support**: Enhanced compatibility and stability for browser extension environments

## [1.0.7] - 2025-04-01

### Added

- **Event Callbacks**: Added callback support for key SDK events
  - `onCheckResultReceived`: Triggered when check results are received, providing text statistics and alert count
  - `onReplacementApplied`: Triggered when a user applies a replacement suggestion from the popover
  - Callbacks can be set globally for all elements or individually per element
  - Both global and element-specific callbacks will be called when available

### Fixed

- **Development Environment Compatibility**: Resolved an issue that caused infinite loading and memory consumption in certain development environments
  - Modified internal code to ensure better compatibility with modern bundlers and development servers

### API Changes

- **ILinguixConfig Interface**:
  - Added `callbacks` property to the configuration object with optional callback functions
  - Added `ILinguixCallbacks` interface defining the callback function signatures
  - Added `ILinguixElementConfig` interface for element-specific configuration

- **LinguixCheckerSDK Class**:
  - Updated `attachToElement` method to accept element-specific options: `attachToElement(element: SupportedElement, options?: ILinguixElementConfig): void`

### Documentation

- Added detailed documentation for callbacks in [callbacks.md](callbacks.md)
- Updated examples to demonstrate both global and element-specific callback usage

## [1.0.6] - 2025-03-25

### Added

- **Next.js Server-Side Rendering (SSR) Compatibility**: Added support for Next.js applications with SSR
  - Modified the SDK to gracefully handle server environments where browser APIs are not available
  - Prevents "HTMLElement is not defined" errors during server-side rendering
  - Maintains full functionality in browser environments

### Usage in Next.js

When using the SDK in Next.js applications:

```javascript
// Add 'use client' directive at the top of components that directly use the SDK
'use client';

import { LinguixCheckerSDK } from '@linguix.com/lx-checker-sdk';

// Initialize as usual
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });
```

## [1.0.5] - 2025-03-20

### Changed

- **Simplified SDK Initialization**: Refactored the initialization methods to provide a more flexible API
  - Replaced separate `initialize()` and `initializeWithConfig()` methods with a single unified `initialize()` method
  - The new method accepts a configuration object that can include any combination of options
  - This allows for more flexible initialization patterns without breaking existing functionality

- **Added Language Forcing Option**: Added the ability to force a specific language for text checking
  - By default, Linguix automatically detects the language of the text (supporting 30+ most popular languages)
  - The new `language` option allows forcing a specific language when automatic detection is not reliable
  - Manual language forcing is limited to: 'en-US', 'en-GB', 'en-ZA', 'en-CA', 'en-AU', 'en-NZ', 'pt-PT', 'pt-BR', 'de-DE', 'fr', 'pl-PL', 'es', 'it'

### API Changes

- **ILinguixConfig Interface**:
  - Added optional `apiKey` property to the configuration object
  - Made the `url` property optional to support different initialization scenarios
  - Added optional `language` property to force specific language for text checking

- **LinguixCheckerSDK Class**:
  - New signature: `initialize(config: ILinguixConfig, messenger?: ILinguixMessenger): void`
  - The config object can include:
    - `apiKey`: Your Linguix API key
    - `url`: Optional URL for custom API endpoint or proxy server
    - `options`: Optional configuration for requests
    - `language`: Optional language code to force specific language for text checking

### Examples

**Basic initialization with API key**:
```javascript
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' });
```

**Initialization with custom endpoint**:
```javascript
LinguixCheckerSDK.initialize({
  url: 'https://your-custom-api-endpoint.com',
  options: {
    query: {
      clientToken: 'optional-client-token'
    }
  }
});
```

**Initialization with messenger for split architecture**:
```javascript
const messenger = new YourMessenger();
LinguixCheckerSDK.initialize({ apiKey: 'your-api-key' }, messenger);
```

**Initialization with language forcing**:
```javascript
LinguixCheckerSDK.initialize({
  apiKey: 'your-api-key',
  language: 'en-US'
});
```

## [1.0.4] and earlier

- Initial releases
