# VBF EFT Analysis

Main analysis


# Scale Uncertainty Choice

We identified that standard scale uncertainties may not capture the full systematic uncertainty arising from different scale choices used in the MC production. The proposed method to empirically measure and account for this additional uncertainty source is sound. The Proposed Solution is:

1.  Test at reconstruction level: Compare the kappa SM sample with EFT SM sample at their nominal weights to quantify the difference at reco level
2.   Account for sample differences: Handle the fact that EFT samples have boosted filters while kappa samples don't
3.  Enhanced uncertainty calculation:
-   For SM samples: Include the kappa SM sample directly as an additional weight in the envelope calculation
-   For EFT samples: Apply the relative difference found in the SM comparison as an additional flat uncertainty (nominal × (1 ± relative uncertainty))

Looking at my code, this would likely require modifications to the `calculate_scale_uncertainty()` function in `systematics.py` to incorporate this additional uncertainty source into the envelope calculation.

I can see how to implement the enhanced scale uncertainty calculation. The key insight is that you need to incorporate the difference between kappa samples (dynamic scale = -1) and EFT samples (dynamic scale = 2) as an additional uncertainty source.

Here's how I would modify the `calculate_scale_uncertainty` function in systematics.py to implement this:

- Enhanced Scale uncertainty with Dynamic Scale Difference

Now you need to modify the configuration system to support this. Here's how you could update your configuration to include the dynamic scale difference parameter:

- Enhanced Configuration System for Dynamic Scale Difference

Now, here's how you would update your configuration file to enable this enhanced uncertainty calculation:

- Enhanced Configuration YAML for Dynamic Scale Difference

Finally, here's how you would implement the empirical measurement step to determine the relative difference between kappa and EFT samples:

- Script to Measure Kappa vs EFT SM Difference

# Enhanced Scale Uncertainty Integration Guide

## Overview
This guide walks you through implementing the enhanced scale uncertainty calculation that accounts for differences between kappa samples (dynamic scale = -1) and EFT samples (dynamic scale = 2).

## Step 1: Measure the Empirical Difference

First, you need to empirically measure the difference between your kappa SM and EFT SM samples at reconstruction level.

### 1.1 Process Both Sample Types
Run your existing analysis on both:
- **Kappa SM sample**: VBFhh_l1cvv1cv1 (DSID 537540) - represents κ_λ=1, c_V=1, c_V=1
- **EFT SM sample**: MGPy8EG_hhjj_bbbb_vbf_EFT_SM (DSID 545851) - represents SM point in EFT framework

### 1.2 Run Measurement Script
```bash
# After processing both samples with your current pipeline
python measure_kappa_eft_difference.py \
    --kappa-file /path/to/VBFhh_l1cvv1cv1_boosted_skim.parquet \
    --eft-file /path/to/MGPy8EG_hhjj_bbbb_vbf_EFT_SM_boosted_skim.parquet \
    --output-config enhanced_scale_config.yaml \
    --truth-diff 0.20  # Known 20% truth-level difference
```

## Step 2: Update systematics.py

### 2.1 Replace/Add Functions
Add the enhanced calculation functions to your `systematics.py`:

```python
# Add these functions from the enhanced_scale_uncertainty artifact
# - calculate_scale_uncertainty_with_dynamic_scale_diff()
# - _set_default_scale_values()
# - get_enhanced_scale_systematics()
```

### 2.2 Update Existing Functions
Modify your existing `get_scale_systematics()` function to support enhanced calculation:

```python
def get_scale_systematics(events, evtweights, process, year, enhanced_config=None):
    # Implementation from enhanced_config_system artifact
```

### 2.3 Update get_systematics_weights()
Add the `enhanced_config` parameter:

```python
def get_systematics_weights(
    events, evtweights, process, year, run_resolved, run_boosted,
    resolved_event_cut, boosted_event_cut, systematics, 
    enhanced_config=None  # Add this parameter
):
    # Implementation from enhanced_config_system artifact
```

## Step 3: Update Analysis Code

### 3.1 Modify processDf() in First_Nominal_Analysis.py
Add the enhanced configuration parameter:

```python
def processDf(
    events,
    # ... existing parameters ...
    enhanced_scale_config=None,  # Add this
    # ... rest of parameters ...
):
    # ... existing code ...
    
    # Update systematics call
    if mc and systematics is not None:
        events, evtweights = get_systematics_weights(
            # ... existing parameters ...
            enhanced_config=enhanced_scale_config  # Add this
        )
```

### 3.2 Update processEJNT.py
Modify `run_selection()` to pass through the enhanced configuration:

```python
def run_selection(events, cfg):
    kwargs = {
        # ... existing kwargs ...
    }
    
    # Add enhanced scale configuration if present
    if "enhanced_scale_uncertainty" in cfg["general"]:
        kwargs["enhanced_scale_config"] = cfg["general"]["enhanced_scale_uncertainty"]
    
    # ... rest of function ...
```

## Step 4: Update Configuration

### 4.1 Add Enhanced Configuration Section
Add to your YAML configuration file:

```yaml
general:
  # ... existing config ...
  
  # Enhanced scale uncertainty (add this section)
  enhanced_scale_uncertainty:
    use_enhanced_scale_uncertainty: true
    kappa_sm_relative_difference: 0.20  # Use value from your measurement
    apply_to_eft_only: true  # Only apply to EFT samples, not pure kappa samples
    save_individual_components: true
```

### 4.2 Update Output Variables
Add new branches to save enhanced uncertainty information:

```yaml
which_out_variables:
  # ... existing variables ...
  
  # Enhanced scale uncertainty branches
  - scale_uncertainty_enhanced
  - scale_dynamic_diff_used
  - scale_dynamic_up_weight
  - scale_dynamic_down_weight
```

## Step 5: Validation and Testing

### 5.1 Test with Known Samples
Run the enhanced calculation on a sample where you know the expected behavior:

```bash
# Process with enhanced uncertainty enabled
python processEJNT.py -c enhanced_config.yaml
```

### 5.2 Validation Checks
Check that:
1. Enhanced uncertainty is larger than standard uncertainty
2. Dynamic scale variations are included in envelope
3. Different sample types get appropriate treatment
4. Backward compatibility maintained for non-enhanced mode

### 5.3 Compare Results
Compare results between:
- Standard scale uncertainty
- Enhanced scale uncertainty
- Different relative difference values

## Step 6: Sample-Specific Logic

### 6.1 Automatic Sample Detection
The code automatically detects sample types:

```python
# EFT samples get the dynamic scale difference applied
if "eft" in process.lower() and "sm" in process.lower():
    use_enhanced = True
    
# Kappa samples might not need it (they use "correct" scale already)
elif any(kappa_point in process.lower() for kappa_point in ["l0cvv0cv1", "l1cvv1cv1"]):
    use_enhanced = False  # or True, depending on your choice
```

### 6.2 Configuration Override
You can override automatic detection in the config:

```yaml
enhanced_scale_uncertainty:
  # Force application to specific processes
  apply_to_processes: ["efthhsm", "efthh"]
  # Or apply to all signals
  apply_to_all_signals: true
```

## Step 7: Production Considerations

### 7.1 Performance Impact
The enhanced calculation adds minimal overhead:
- 2 additional variations in envelope calculation
- No significant computational cost increase

### 7.2 Memory Usage
Additional branches stored:
- `scale_uncertainty_enhanced`: boolean flag
- `scale_dynamic_diff_used`: used relative difference
- `scale_dynamic_*_weight`: dynamic scale variations

### 7.3 Backwards Compatibility
- Standard scale uncertainty remains default
- Enhanced mode only active when explicitly configured
- Existing analyses unaffected

## Troubleshooting

### Common Issues

1. **Missing PDF branches**: Ensure your samples have the required generator weight branches
2. **Sample detection fails**: Check process naming in your filelist
3. **Large uncertainties**: Validate your empirical measurement
4. **Configuration not loaded**: Check YAML syntax and path

### Debug Mode
Enable debug output:

```yaml
enhanced_scale_uncertainty:
  debug_mode: true  # Adds detailed logging
```

### Validation Plots
Create validation plots comparing:
- Standard vs enhanced uncertainties
- Individual scale variations
- Dynamic scale contributions

This implementation provides a robust framework for including the dynamic scale uncertainty while maintaining flexibility and backwards compatibility with existing analyses.