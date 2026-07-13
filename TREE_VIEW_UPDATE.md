# Tree View Update - Star Model Layers Info

## New Features Added

### 1. View Mode Dropdown

Added a new dropdown input to switch between two view modes:
- **Normal View**: Flat list of all layers (original format)
- **Tree View**: Hierarchical grouped view with layer ranges

### 2. Tree View Implementation

The Tree View provides a hierarchical structure similar to the image provided:

```
├── blocks (6.4 GB | BF16, F32, F8_E4M3, NVFP4)
│   ├── [0-27] (6.4 GB | BF16, F32, F8_E4M3, NVFP4)
│   │   ├── attn (1.9 GB | BF16, F32, F8_E4M3, NVFP4)
│   │   │   ├── gate (567.0 MB | F32, F8_E4M3, NVFP4)
│   │   │   ├── weight (504.0 MB | NVFP4)
│   │   │   ├── weight_scale (63.0 MB | F8_E4M3)
│   │   │   └── weight_scale_2 (112 B | F32)
│   │   ├── qknorm (14.6 KB | BF16)
│   │   │   ├── knorm (7.0 KB | BF16)
│   │   │   │   └── scale (7.0 KB | BF16)
│   │   │   └── qnorm (7.0 KB | BF16)
│   │   │       └── scale (7.0 KB | BF16)
│   │   └── wk (141.8 MB | F32, F8_E4M3, NVFP4)
│   │       ├── weight (126.0 MB | NVFP4)
│   │       ├── weight_scale (15.8 MB | F8_E4M3)
│   │       └── weight_scale_2 (112 B | F32)
```

### 3. Layer Grouping

**Consecutive Layer Ranges:**
- Groups 3+ consecutive numbered layers (e.g., blocks.0, blocks.1, blocks.2 → [0-2])
- Shows combined size and formats for the range
- Breaks down sub-components within the range

**Format Aggregation:**
- Shows all unique formats used in a group
- Displays total size for the group
- Example: `(6.4 GB | BF16, F32, F8_E4M3, NVFP4)`

### 4. Separate Output Files

Each view mode creates its own file:
- **Normal View**: `{model_name}_normal.txt`
- **Tree View**: `{model_name}_tree.txt`

This allows you to run both views and compare the outputs.

---

## Implementation Details

### View Mode Input

```python
"view_mode": (["Normal View", "Tree View"], {
    "default": "Normal View",
    "tooltip": "Normal View: Flat list of all layers. Tree View: Hierarchical grouped view with layer ranges."
})
```

### Normal View Format

```
layer_name                                                                       | Shape: [3072, 3072]      | Format: NVFP4              | Params:    9,437,184 | Size: 4.50 MB
```

### Tree View Format

```
├── prefix (total_size | formats)
│   ├── [start-end] (group_size | group_formats)
│   │   ├── component (component_size | component_formats)
│   │   │   ├── subcomponent (size | format)
```

### Grouping Logic

1. **Split by prefix**: Groups layers by first-level name (e.g., "blocks", "double_blocks")
2. **Extract numbers**: Finds numbered layers using regex `\.(\d+)\.`
3. **Group consecutive**: Combines 3+ consecutive numbers into ranges
4. **Sub-components**: Shows breakdown of components within ranges

---

## Features

### ✅ Hierarchical Structure
- Tree-like display with proper indentation
- Uses box-drawing characters (├──, │)
- Multiple nesting levels

### ✅ Layer Range Grouping
- Automatically groups consecutive layers
- Shows range as `[0-27]` instead of listing each
- Minimum 3 layers required for grouping

### ✅ Size and Format Aggregation
- Calculates total size for each group
- Lists all unique formats in the group
- Example: `(6.4 GB | BF16, F32, F8_E4M3, NVFP4)`

### ✅ Component Breakdown
- Shows sub-components within ranges
- Groups by component type (attn, mlp, norm, etc.)
- Displays size and formats for each component

### ✅ Dual File Output
- `_normal.txt` for flat view
- `_tree.txt` for hierarchical view
- Both can be generated independently

---

## Usage Examples

### Example 1: Compare Views
```
Star Model Layers Info
  - model_name: flux-dev-nvfp4
  - view_mode: Normal View
  
→ Output: flux-dev-nvfp4_normal.txt

Star Model Layers Info
  - model_name: flux-dev-nvfp4
  - view_mode: Tree View
  
→ Output: flux-dev-nvfp4_tree.txt
```

### Example 2: Tree View for Large Models
```
Star Model Layers Info
  - model_name: sdxl-base-1.0
  - view_mode: Tree View
  
→ Shows grouped structure:
   ├── input_blocks [0-11] (...)
   ├── middle_block (...)
   └── output_blocks [0-11] (...)
```

---

## Console Output

```
🔍 [Star Model Layers Info] Starting analysis...
📦 Loading model: flux-dev-nvfp4.safetensors
🔍 Analyzing layers in Tree View...
💾 Saving layer info to: output/modelinfo/flux-dev-nvfp4_tree.txt

============================================================
✅ Model analysis complete (Tree View)
Model: flux-dev-nvfp4
Total layers: 1,234
Total parameters: 12,345,678,901
File size: 11.92 GB
Analysis time: 2.3s
Report saved to: output/modelinfo/flux-dev-nvfp4_tree.txt
============================================================
```

---

## Format Abbreviations

To match the image style, formats are abbreviated:
- `NVFP4` → NVFP4
- `F8_E4M3` → F8_E4M3 (FP8)
- `BF16` → BF16
- `F32` → F32
- `INT8_CONVROT` → INT8_CONVROT
- `INT4_CONVROT` → INT4_CONVROT
- `MXFP8` → MXFP8

---

## Benefits

### 📊 Better Overview
Tree view provides a high-level structure overview without overwhelming detail

### 🔍 Easy Navigation
Grouped layers make it easier to find specific components

### 📏 Size Analysis
Quickly see which groups consume the most memory

### 🎯 Format Distribution
See at a glance which formats are used in each section

### 📝 Dual Output
Keep both views for different use cases

---

## Files Modified

1. **star_model_layers_info.py**
   - Added `view_mode` dropdown input
   - Added `_build_normal_view()` method
   - Added `_build_tree_view()` method
   - Added `_group_consecutive_layers()` method
   - Updated file naming with view suffix
   - Updated status message with view mode

---

## Technical Details

### Tree Building Algorithm

1. **Parse layer names**: Split by `.` to get hierarchy
2. **Group by prefix**: First-level grouping (blocks, double_blocks, etc.)
3. **Extract numbers**: Find numbered layers with regex
4. **Sort and group**: Group consecutive numbers (3+ minimum)
5. **Calculate aggregates**: Sum sizes, collect unique formats
6. **Build tree**: Recursive structure with proper indentation

### Grouping Threshold

- **Minimum 3 layers**: Only groups if 3+ consecutive layers exist
- **Example**: `[0-2]` groups layers 0, 1, 2
- **Non-grouped**: Layers 0, 1 (only 2) shown individually

---

**Status**: ✅ Complete and ready to test  
**Backward Compatible**: Yes (Normal View is default)  
**New Files**: Separate `_tree.txt` and `_normal.txt` outputs
