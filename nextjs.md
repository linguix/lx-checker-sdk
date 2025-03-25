# Using Linguix Checker SDK with Next.js

This guide explains how to integrate the Linguix Checker SDK with Next.js applications, including handling Server-Side Rendering (SSR).

## Installation

```bash
npm install @linguix.com/lx-checker-sdk
# or
yarn add @linguix.com/lx-checker-sdk
```

## Basic Integration

When using the SDK in a Next.js application, you need to:

1. Add the `'use client'` directive at the top of your component
2. Initialize the SDK inside a `useEffect` hook
3. Connect your text input elements to the SDK

### Example with Material UI

```jsx
'use client'

import { TextField } from "@mui/material";
import { useEffect, useRef } from "react";
import { LinguixCheckerSDK } from "@linguix.com/lx-checker-sdk";

export default function Home() {
  const textFieldRef = useRef(null);

  useEffect(() => {
      LinguixCheckerSDK.initialize({ apiKey: 'API_KEY' });

      if (textFieldRef.current) {
          LinguixCheckerSDK.attachToElement(textFieldRef.current);
      }
  }, []);

  return (
      <div style={{ maxWidth: 600, margin: '0 auto' }}>
        <h1>Demo: Linguix Checker with MUI</h1>
        <TextField
            label="Write something here..."
            variant="outlined"
            multiline
            rows={4}
            fullWidth
            inputRef={textFieldRef}
        />
      </div>
  );
}
```

## Important Notes

- The `'use client'` directive is required because the SDK uses browser-specific APIs
- Initialize the SDK inside `useEffect` to ensure it only runs in the browser
- The SDK will gracefully handle server-side rendering environments
- For optimal performance, consider adding cleanup in your useEffect return function:

```jsx
useEffect(() => {
    LinguixCheckerSDK.initialize({ apiKey: 'API_KEY' });

    if (textFieldRef.current) {
        LinguixCheckerSDK.attachToElement(textFieldRef.current);
    }

    // Cleanup on unmount
    return () => {
        if (textFieldRef.current) {
            LinguixCheckerSDK.detachFromElement(textFieldRef.current);
        }
    };
}, []);
```

## Troubleshooting

If you encounter any issues:

1. Make sure you're using the latest version of the SDK
2. Verify that the `'use client'` directive is at the top of your component
3. Check that you're initializing the SDK inside a `useEffect` hook
4. Ensure your API key is correct
