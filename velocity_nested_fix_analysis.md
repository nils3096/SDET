# Velocity Template - Fix for Nested serviceCharacteristic Array

## Problem Identified
The template is adding a **new** `serviceCharacteristic` array at the `serviceQualificationItem` level, instead of adding to the **existing** `serviceCharacteristic` array inside the `service` object.

## Current Structure (What's Happening)
```json
{
  "id": "CCMS_SQI_ID_0",
  "qualificationResult": "Feasible",
  "service": {
    "@type": "service",
    "serviceCharacteristic": [
      // Original characteristics are here
    ]
  },
  "@type": "serviceQualificationItem",
  "externalId": "...",
  "serviceCharacteristic": [
    // Extra characteristics are being added here (WRONG PLACE)
  ]
}
```

## Desired Structure (What We Want)
```json
{
  "id": "CCMS_SQI_ID_0",
  "qualificationResult": "Feasible",
  "service": {
    "@type": "service",
    "serviceCharacteristic": [
      // Original characteristics
      // + Extra characteristics (RIGHT PLACE)
    ]
  },
  "@type": "serviceQualificationItem",
  "externalId": "..."
}
```

## Solution: Target the Nested Array
We need to be more specific and target the `serviceCharacteristic` array that's inside the `service` object.

### Fixed Code
```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Target the serviceCharacteristic array inside the service object specifically
  #set($searchPattern = '"service": {"@type": "service","serviceCharacteristic": [')
  #set($replacementPattern = '"service": {"@type": "service","serviceCharacteristic": [' + $extraChars + ',')
  #set($patchedJson = $originalJson.replace($searchPattern, $replacementPattern))
  
  ## Also handle case with spaces around elements
  #set($searchPattern2 = '"service": { "@type": "service", "serviceCharacteristic": [')
  #set($replacementPattern2 = '"service": { "@type": "service", "serviceCharacteristic": [' + $extraChars + ',')
  #set($patchedJson = $patchedJson.replace($searchPattern2, $replacementPattern2))
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

### Alternative Approach: More Flexible Pattern Matching
If the JSON formatting varies, try this more flexible approach:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Find and replace the serviceCharacteristic array within the service object
  #set($patchedJson = $originalJson)
  
  ## Try different formatting patterns for the service object
  #set($patterns = [
    '"service":{"@type":"service","serviceCharacteristic":[',
    '"service": {"@type": "service","serviceCharacteristic": [',
    '"service": { "@type": "service", "serviceCharacteristic": [',
    '"service":{"@type":"service", "serviceCharacteristic":[',
    '"service": {"@type":"service", "serviceCharacteristic": ['
  ])
  
  #foreach($pattern in $patterns)
    #set($replacement = $pattern + $extraChars + ',')
    #set($patchedJson = $patchedJson.replace($pattern, $replacement))
  #end
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

### Safest Approach: String Manipulation
If the above patterns don't work reliably, use this approach:

```velocity
#elseif($itemStr.startsWith("{") && $itemStr.endsWith("}"))
  #set($itemLength = $itemStr.length())
  #set($endIndex = $itemLength - 1)
  #set($originalJson = $itemStr.substring(1, $endIndex))
  #set($extraChars = '{"name": "Committed Capacity","valueType": "Committed Capacity","mandatory": true,"value": "Capacity Tiers [1..4] - (Capacity (%), Availability (%))"}, {"name": "Performance Objectives","valueType": "Performance Objectives","mandatory": true,"value": "Service Availability (%) Per Performance Tier / Per CoS: (Frame Delay, Inter-Frame Delay Variation, Frame Loss)"}')
  
  ## Find the position of the service object's serviceCharacteristic array
  #set($serviceIndex = $originalJson.indexOf('"service":'))
  #if($serviceIndex >= 0)
    #set($serviceSection = $originalJson.substring($serviceIndex))
    #set($charIndex = $serviceSection.indexOf('"serviceCharacteristic": ['))
    #if($charIndex >= 0)
      ## Build the replacement for just the service section
      #set($beforeChar = $serviceSection.substring(0, $charIndex + 25))  ## 25 = length of '"serviceCharacteristic": ['
      #set($afterCharIndex = $charIndex + 25)
      #set($afterChar = $serviceSection.substring($afterCharIndex))
      #set($newServiceSection = $beforeChar + $extraChars + ',' + $afterChar)
      
      ## Replace the service section in the original JSON
      #set($beforeService = $originalJson.substring(0, $serviceIndex))
      #set($afterServiceIndex = $originalJson.indexOf('}', $serviceIndex + $serviceSection.indexOf('}')) + 1)
      #set($afterService = $originalJson.substring($afterServiceIndex))
      #set($patchedJson = $beforeService + $newServiceSection + $afterService)
    #else
      #set($patchedJson = $originalJson)
    #end
  #else
    #set($patchedJson = $originalJson)
  #end
  
  #set($currentItem = '{"id": "CCMS_SQI_ID_' + $foreach.index + '","qualificationResult": "Feasible",' + $patchedJson + '}')
```

## Recommendation
Start with the first approach (targeting the specific pattern) since it's simpler and should work for your consistent JSON structure.