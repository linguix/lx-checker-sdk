# Linguix Checker SDK Callbacks

The Linguix Checker SDK provides callbacks that allow you to respond to various events during the checking process. These callbacks can be set either globally (for all elements) or per individual element.

## Available Callbacks

The SDK currently supports the following callbacks:

### onCheckResultReceived

This callback is triggered when check results are received from the Linguix API.

**Callback data:**
- `textStats`: Statistics about the checked text (readability scores, word count, etc.)
- `alertsCount`: The number of alerts (grammar/spelling issues) found in the text

#### TextStats Object

The `textStats` property provides detailed information about the analyzed text:

```typescript
interface ILinguixTextStats {
    wordsCount: number;        // Total number of words in the text
    charsCount: number;        // Total number of characters in the text
    avgWordLength: number;     // Average word length
    avgSentenceLength: number; // Average sentence length (in words)
    sentencesCount: number;    // Total number of sentences
    fleschIndex: number;       // Flesch Reading Ease score (0-100, higher is easier to read)
    textScore: number;         // Overall text quality score
    readingTimeSeconds: number; // Estimated reading time in seconds
    speakingTimeSeconds: number; // Estimated speaking time in seconds
}
```

These statistics can be useful for:
- Displaying readability metrics to users
- Tracking writing improvements over time
- Setting goals for content quality
- Estimating reading or speaking time for content

### onReplacementApplied

This callback is triggered when a user applies a replacement suggestion from the popover.

**Callback data:**
- `originalText`: The original text that was replaced
- `replacement`: The replacement text that was applied
- `description`: The description of the error as shown in the alert popover

The `description` field provides a human-readable explanation of the error type, such as "Determiner Discrepancies", "Punctuation Problems", or "Typography Issues". This matches what users see in the UI when they interact with the alert popover.

## Usage Examples

### Global Callbacks

Global callbacks are applied to all elements that are checked by the SDK:

```javascript
// Initialize with global callbacks
LinguixCheckerSDK.initialize({
  apiKey: 'your-api-key',
  callbacks: {
    // Called when check results are received
    onCheckResultReceived: (result) => {
      console.log('Global callback - Text stats:', result.textStats);
      console.log('Global callback - Number of alerts:', result.alertsCount);
    },
    
    // Called when a user applies a replacement from the popover
    onReplacementApplied: (data) => {
      console.log('Global callback - Original text:', data.originalText);
      console.log('Global callback - Applied replacement:', data.replacement);
      console.log('Global callback - Error description:', data.description);
    }
  }
});
```

### Element-Specific Callbacks

You can also set callbacks for specific elements:

```javascript
// Set callbacks for a specific element
const textarea = document.querySelector('textarea');
LinguixCheckerSDK.attachToElement(textarea, {
  callbacks: {
    onCheckResultReceived: (result) => {
      console.log('Element-specific callback - Check results:', result);
    },
    onReplacementApplied: (data) => {
      console.log('Element-specific callback - Original text:', data.originalText);
      console.log('Element-specific callback - Replacement:', data.replacement);
      console.log('Element-specific callback - Description:', data.description);
    }
  }
});
```

## Callback Behavior

When both global and element-specific callbacks are defined, both will be called when the corresponding event occurs. This allows you to have general handling at the global level while still implementing specialized behavior for specific elements.

## Use Cases

### Analytics

Use callbacks to track user engagement with the grammar checker:

```javascript
LinguixCheckerSDK.initialize({
  apiKey: 'your-api-key',
  callbacks: {
    onCheckResultReceived: (result) => {
      // Track how many issues were found
      analytics.track('grammarCheck', {
        alertsCount: result.alertsCount,
        wordCount: result.textStats.wordsCount,
        readability: {
          fleschIndex: result.textStats.fleschIndex,
          textScore: result.textStats.textScore,
          avgSentenceLength: result.textStats.avgSentenceLength
        },
        estimatedReadTime: result.textStats.readingTimeSeconds
      });
    },
    onReplacementApplied: (data) => {
      // Track when users apply suggestions
      analytics.track('suggestionApplied', {
        replacement: data.replacement,
        errorDescription: data.description
      });
    }
  }
});
```

### UI Updates

Update your application's UI based on grammar check results:

```javascript
LinguixCheckerSDK.attachToElement(document.getElementById('editor'), {
  callbacks: {
    onCheckResultReceived: (result) => {
      // Update a counter showing number of issues
      document.getElementById('issue-count').textContent = result.alertsCount;
      
      // Update readability score display
      document.getElementById('readability-score').textContent = 
        result.textStats.fleschIndex.toFixed(1);
        
      // Update word count
      document.getElementById('word-count').textContent = 
        result.textStats.wordsCount;
        
      // Update estimated reading time (convert to minutes)
      const readingMinutes = Math.ceil(result.textStats.readingTimeSeconds / 60);
      document.getElementById('reading-time').textContent = 
        `${readingMinutes} min read`;
    }
  }
});
