# Debug JSON Structure to Fix serviceCharacteristic Injection

## Problem
The search pattern isn't matching the actual JSON structure, so the replacement isn't happening.

## Debug Step 1: See the Raw JSON
First, let's see exactly what the JSON looks like before processing:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  
  ## DEBUG: Show the exact JSON structure
  #set($debugJson = $originalJson.replace('"', '\"'))
  #set($currentItem = '{"DEBUG_ORIGINAL_JSON": "' + $debugJson + '"}')
```

## Debug Step 2: Test Different Patterns
Try this to see which pattern exists:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  
  ## Test different patterns to see which one exists
  #set($pattern1 = '"service": {"@type": "service","serviceCharacteristic": [')
  #set($pattern2 = '"service": { "@type": "service", "serviceCharacteristic": [')
  #set($pattern3 = '"service":{"@type":"service","serviceCharacteristic":[')
  #set($pattern4 = '"service": {"@type": "service", "serviceCharacteristic": [')
  
  #set($found1 = $originalJson.contains($pattern1))
  #set($found2 = $originalJson.contains($pattern2))
  #set($found3 = $originalJson.contains($pattern3))
  #set($found4 = $originalJson.contains($pattern4))
  
  #set($currentItem = '{"DEBUG": {"pattern1": ' + $found1 + ', "pattern2": ' + $found2 + ', "pattern3": ' + $found3 + ', "pattern4": ' + $found4 + '}}')
```

## Most Likely Fix
Based on the JSON you showed earlier, try this pattern:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Try the most likely pattern from your JSON structure
  #set($searchPattern = '"@type": "service", "serviceCharacteristic": [')
  #set($replacementPattern = '"@type": "service", "serviceCharacteristic": [' + $extraChars + ',')
  #set($patchedJson = $originalJson.replace($searchPattern, $replacementPattern))
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

## Alternative: Generic serviceCharacteristic Pattern
If the above doesn't work, try this more generic approach:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Look for any serviceCharacteristic array that's not empty
  #set($patchedJson = $originalJson)
  
  ## Pattern: serviceCharacteristic with opening bracket followed by opening brace (indicating non-empty array)
  #set($searchPattern = '"serviceCharacteristic": [{"')
  #set($replacementPattern = '"serviceCharacteristic": [' + $extraChars + ',{"')
  #set($patchedJson = $patchedJson.replace($searchPattern, $replacementPattern))
  
  ## Also try without space after colon
  #set($searchPattern2 = '"serviceCharacteristic":[{"')
  #set($replacementPattern2 = '"serviceCharacteristic":[' + $extraChars + ',{"')
  #set($patchedJson = $patchedJson.replace($searchPattern2, $replacementPattern2))
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

## Step-by-Step Approach
1. First run the debug code to see the exact JSON structure
2. Use that information to create the correct search pattern
3. Apply the fix with the correct pattern