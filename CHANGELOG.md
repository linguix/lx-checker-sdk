# Changelog

All notable changes to the Linguix Checker SDK will be documented in this file.

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
