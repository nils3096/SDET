# Velocity Template - Debugging String Replacement

## Problem
The regex error is fixed, but the string replacement isn't working - the extra service characteristics aren't being added.

## Likely Causes

1. **Search string doesn't match actual JSON format**
2. **JSON might have different whitespace/formatting**
3. **The field might not exist in the original JSON**
4. **Case sensitivity issues**

## Debugging Steps

### Step 1: Add Debug Output
Temporarily add debug information to see what's actually in your JSON:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  
  ## DEBUG: Output the original JSON to see its format
  ## Remove these debug lines once you identify the issue
  #set($currentItem = '{"DEBUG_originalJson": "' + $originalJson.replace('"', '\\"') + '"}')
```

### Step 2: Check if serviceCharacteristic exists
Look for variations of the field name:

```velocity
## Check what's actually in the JSON
#set($hasServiceChar = $originalJson.contains('"serviceCharacteristic"'))
#set($hasServiceCharCap = $originalJson.contains('"ServiceCharacteristic"'))  
#set($hasServiceCharArray = $originalJson.contains('"serviceCharacteristic": ['))
#set($hasServiceCharArrayNoSpace = $originalJson.contains('"serviceCharacteristic":['))

#set($currentItem = '{"DEBUG": {"hasServiceChar": ' + $hasServiceChar + ', "hasServiceCharCap": ' + $hasServiceCharCap + ', "hasServiceCharArray": ' + $hasServiceCharArray + ', "hasServiceCharArrayNoSpace": ' + $hasServiceCharArrayNoSpace + '}}')
```

### Step 3: Multiple Search Patterns
Try different variations of the search string:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Try multiple search patterns
  #set($patchedJson = $originalJson)
  
  ## Pattern 1: With space after colon
  #set($searchText1 = '"serviceCharacteristic": [')
  #set($replacementText1 = '"serviceCharacteristic": [' + $extraChars + ',')
  #set($patchedJson = $patchedJson.replace($searchText1, $replacementText1))
  
  ## Pattern 2: Without space after colon
  #set($searchText2 = '"serviceCharacteristic":[')
  #set($replacementText2 = '"serviceCharacteristic":[' + $extraChars + ',')
  #set($patchedJson = $patchedJson.replace($searchText2, $replacementText2))
  
  ## Pattern 3: Empty array
  #set($searchText3 = '"serviceCharacteristic": []')
  #set($replacementText3 = '"serviceCharacteristic": [' + $extraChars + ']')
  #set($patchedJson = $patchedJson.replace($searchText3, $replacementText3))
  
  ## Pattern 4: Empty array without space
  #set($searchText4 = '"serviceCharacteristic":[]')
  #set($replacementText4 = '"serviceCharacteristic":[' + $extraChars + ']')
  #set($patchedJson = $patchedJson.replace($searchText4, $replacementText4))
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

## Most Likely Solutions

### Solution 1: Field doesn't exist - Add it
If `serviceCharacteristic` doesn't exist in the JSON, add it:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Add serviceCharacteristic field if it doesn't exist
  #if($originalJson.contains('"serviceCharacteristic"'))
    ## Field exists, try to inject into array
    #set($searchText = '"serviceCharacteristic": [')
    #set($replacementText = '"serviceCharacteristic": [' + $extraChars + ',')
    #set($patchedJson = $originalJson.replace($searchText, $replacementText))
    
    ## Also try without space
    #set($searchText2 = '"serviceCharacteristic":[')
    #set($replacementText2 = '"serviceCharacteristic":[' + $extraChars + ',')
    #set($patchedJson = $patchedJson.replace($searchText2, $replacementText2))
  #else
    ## Field doesn't exist, add it at the end
    #set($patchedJson = $originalJson + ',"serviceCharacteristic": [' + $extraChars + ']')
  #end
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

### Solution 2: Insert at end of JSON object
A safer approach - always add at the end:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Always add serviceCharacteristic at the end
  #set($serviceCharField = '"serviceCharacteristic": [' + $extraChars + ']')
  #set($patchedJson = $originalJson + ',' + $serviceCharField)
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

## Recommendation

Start with the debug output (Step 1) to see what your JSON actually looks like, then choose the appropriate solution based on what you find.