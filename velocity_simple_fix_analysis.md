# Velocity Template - Simple String Replacement Fix

## Problem
The regex escaping for `[` character is still causing issues even with proper escaping attempts.

## Simple Solution
Since you're doing a straightforward string replacement (not complex pattern matching), use `replace()` instead of `replaceFirst()` to avoid regex complications entirely.

## Fixed Code Section

Replace this problematic section:
```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  #set($replacementPattern = '"serviceCharacteristic": [')
  #set($replacementValue = "${replacementPattern}${extraChars},")
  #set($searchPattern = '"serviceCharacteristic"\\s*:\\s*\\\\[')
  #set($patchedJson = $originalJson.replaceFirst($searchPattern, $replacementValue))
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

With this simple version:
```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  ## Simple string replacement - no regex needed
  #set($searchText = '"serviceCharacteristic": [')
  #set($replacementText = '"serviceCharacteristic": [' + $extraChars + ',')
  #set($patchedJson = $originalJson.replace($searchText, $replacementText))
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

## Key Changes

1. **Use `replace()` instead of `replaceFirst()`**: No regex patterns needed
2. **Simple string matching**: Look for exact string `"serviceCharacteristic": [`
3. **Direct string concatenation**: Build replacement string with simple `+` operator
4. **No escaping issues**: No special regex characters to worry about

## Why This Works

- `replace()` does literal string matching, not regex pattern matching
- No need to escape special characters like `[`, `]`, `*`, etc.
- More predictable behavior in Velocity templates
- Simpler to read and maintain

## Alternative Approach (if you need whitespace flexibility)

If your JSON might have varying whitespace around the colon, you could do:
```velocity
#set($patchedJson = $originalJson.replace('"serviceCharacteristic":[', $replacementText))
#set($patchedJson = $patchedJson.replace('"serviceCharacteristic": [', $replacementText))
```

This handles both cases: with and without space after the colon.