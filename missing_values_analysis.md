# Missing ServiceQualificationItem Values - Analysis & Solutions

## Problem Summary

Your current template correctly shows:
- ✅ **`extendedParameters`** as proper JSON object
- ❌ **`serviceQualificationItem`** objects missing original values (only showing `id` and `qualificationResult`)

## Root Cause

The `serviceQualificationItem` objects from your request aren't being properly accessed. The original values exist but our property access methods aren't working.

## Diagnostic Steps

### Step 1: Run Diagnostic
First, run `velocity_item_diagnostic.vm` to understand:
- What type of objects are in your `serviceQualificationItem` array
- What methods/properties are available
- Whether they implement Map interface

### Step 2: Try Solutions in Order

#### Solution A: **Robust Multi-Approach** (`velocity_template_robust_preserve.vm`)
- Tries 4 different methods to extract properties:
  1. **Map.keySet()** - if item is a Map
  2. **Map.entrySet()** - if item has entrySet
  3. **toString() parsing** - if toString contains key=value pairs
  4. **Manual property access** - tries common TMF properties

#### Solution B: **Direct Map Copy** (`velocity_template_original_approach.vm`)
- Uses `putAll()` to copy all properties from original item
- Adds required fields on top

## Expected Outcomes

### If diagnostic shows Map-like objects:
```
Has entrySet: YES
Is Map-like: YES
Map keys: [service, quantity, expectedActivationDate, ...]
```
→ **Solution A** should work with keySet/entrySet approach

### If diagnostic shows custom objects:
```
Has entrySet: NO
Is Map-like: NO
Available methods: [getService, getQuantity, ...]
```
→ Need custom getter-based approach

### If diagnostic shows serialized data:
```
First Item toString(): {service: {...}, quantity: 1, ...}
```
→ **Solution A** will use toString parsing approach

## Your Original Request Structure

Based on your template, your request likely contains:
```json
{
  "serviceQualificationItem": [
    {
      "service": { /* service object */ },
      "quantity": 1,
      "expectedActivationDate": "...",
      "place": [...],
      "category": "...",
      "serviceSpecification": {...},
      // ... other TMF properties
    }
  ]
}
```

## Troubleshooting

If neither solution works:
1. **Run the diagnostic** to see object structure
2. **Check the `_debug` field** in fallback output
3. **Look for getter methods** like `getService()`, `getQuantity()`
4. **Consider reflection-based approach** if objects use private fields

## Alternative: Custom Property Names

If your objects use different property names, modify the fallback section:
```velocity
#if($item.serviceInfo),"service": $item.serviceInfo#end
#if($item.qty),"quantity": $item.qty#end
#if($item.activationDate),"expectedActivationDate": "$item.activationDate"#end
```

## Next Steps

1. Run `velocity_item_diagnostic.vm` 
2. Share the diagnostic output
3. Try `velocity_template_robust_preserve.vm`
4. If still missing values, we'll create a custom solution based on the diagnostic results