# Velocity Template JSON Output Issues and Solutions

## Problems Identified

Your original Velocity template had two main issues:

1. **`extendedParameters` showing literal variable name**: The output showed `$extendedParametersJson` instead of the actual JSON value
2. **`serviceQualificationItem` showing literal method calls**: The array showed `$tool.json.stringify($itemMap)` strings instead of actual JSON objects

## Root Causes

1. **Variable interpolation issue**: When using intermediate variables for JSON stringify operations, Velocity sometimes doesn't properly evaluate them in the final output
2. **Method evaluation in loops**: The `$tool.json.stringify($itemMap)` calls within the foreach loop weren't being properly evaluated

## Solutions Provided

### Solution 1: Direct Method Calls (`velocity_template_fix.vm`)
- Removed the intermediate `$extendedParametersJson` variable
- Used `$tool.json.stringify($tc.saved.extendedParameters)` directly in the JSON output
- Kept the original foreach loop structure but simplified it

### Solution 2: Pre-build Arrays (`velocity_template_alternative.vm`)
- Build the `serviceQualificationItems` array first using a foreach loop
- Use `$tool.json.stringify($serviceQualificationItems)` to output the entire array at once
- Use `${tool.json.stringify(...)}` syntax with explicit braces for better parsing

### Solution 3: Manual JSON Construction (`velocity_template_manual.vm`)
- Manually construct the JSON structure without relying heavily on `$tool.json.stringify()`
- Build each service qualification item object manually within the foreach loop
- Use conditional logic to include optional fields

## Recommended Approach

Try **Solution 2** first (`velocity_template_alternative.vm`) as it:
- Pre-processes the data outside the JSON structure
- Uses explicit brace syntax `${...}` for better variable evaluation
- Maintains clean separation between data processing and output formatting

If Solution 2 doesn't work, try **Solution 3** (`velocity_template_manual.vm`) which manually constructs the JSON and should be the most reliable.

## Key Changes Made

1. **Removed intermediate variables** that were causing literal output
2. **Used explicit brace syntax** `${variable}` instead of just `$variable` for better parsing
3. **Pre-processed arrays** before JSON output
4. **Added null checks** for `extendedParameters`
5. **Simplified the foreach loop** logic for service qualification items

## Testing

After implementing one of these solutions, your output should show:
- Actual JSON object for `extendedParameters` instead of `$extendedParametersJson`
- Proper JSON objects in the `serviceQualificationItem` array instead of literal strings
- Correct `id` values like `CCMS_SQI_ID_0`, `CCMS_SQI_ID_1`, etc. for each item