discussions-to: (https://github.com/tronprotocol/tips/issues/791)

---
tip: [TBD]
title: Dynamic Administrative Pricing (DAP) Model for Energy in the TRON Network
author: Anton Erpulev <anton.erpulev@powersun.vip>
status: Draft
type: Standards Track
category: Core
created: 2025-08-27
---

# TIP-[TBD]: Dynamic Administrative Pricing (DAP) Model for Energy in the TRON Network

## Simple Summary

Replace the current static Energy pricing of 210 sun with an intelligent dynamic system that automatically adjusts based on market conditions and network load, offering strategic discounts during downturns while maintaining base pricing during stability.

## Abstract

TRON's static administrative price of 210 sun per Energy unit fails to adapt to changing market conditions and empirically observed asymmetric user behavior. Historical analysis reveals Energy demand is highly inelastic to price increases (elasticity -0.25 to -0.32) but highly elastic to price decreases (elasticity -3.0).

This proposal introduces a Dynamic Administrative Pricing (DAP) model based on a **philosophy of strategic stimulation**. Instead of maximizing revenue through price increases, the model uses 210 sun as a "Base Price" with algorithmic discounts based on real-time Market Trend (TRX price changes) and Network Load (Energy consumption changes). Backtesting demonstrates 16.66% increased network activity while maintaining comparable revenue levels (+6.85%).

## Motivation

### Current System Limitations

Static pricing creates two fundamental inefficiencies:

1. **Missed Revenue Optimization**: Fails to capture additional value from inelastic demand during high-activity periods
2. **Suppressed Network Growth**: Cannot stimulate price-sensitive users during quiet periods when lower costs would drive ecosystem expansion

### Empirical Evidence of Asymmetric Demand Elasticity

Analysis of historical Energy price changes reveals strong behavioral asymmetry:

| Date | Change Type | Price Δ | Consumption Δ | Elasticity | Demand Classification |
|------|-------------|---------|---------------|------------|----------------------|
| 2021-09-09 | Increase | +100% | -25% | -0.25 | Highly Inelastic |
| 2022-06-15 | Decrease | -50% | +150% | -3.0 | Highly Elastic |
| 2023-08-01 | Increase | +50% | -15% | -0.30 | Highly Inelastic |
| 2024-03-12 | Increase | +25% | -8% | -0.32 | Highly Inelastic |

This asymmetry indicates two distinct user segments:
- **Core Users** (inelastic): Power users, large dApps, arbitrage bots with high switching costs
- **Marginal Users** (elastic): Price-sensitive users and applications who activate based on economic viability

### Economic Rationale

The fundamental economic insight is that optimal pricing should vary with demand characteristics. Static pricing leaves economic value unrealized by:
- Under-pricing during inelastic periods (lost revenue)
- Over-pricing during elastic periods (lost network growth)

Dynamic pricing captures both opportunities through intelligent market-responsive adjustments.

## Specification

### Market State Classification System

The DAP model evaluates network conditions using two orthogonal metrics:

- **X-Axis (Market Trend)**: Month-over-month TRX average price change (%)
- **Y-Axis (Network Load)**: Month-over-month Energy consumption change (%)

Classification matrix with threshold values:

| State | TRX Price Trend | Energy Consumption | Pricing Strategy |
|-------|----------------|-------------------|------------------|
| **Stability** | ≥ -10% | ≥ -15% | Base Price (210 sun) |
| **Moderate Downturn** | < -10% OR < -15% | Calculated Discount |
| **Deep Downturn** | < -10% AND < -15% | Maximum Discount |

### Dynamic Pricing Mathematical Framework

#### Step 1: Normalized Downturn Indicators

For each metric k (TRX price or Energy consumption):

```
I_k = max(0, (-Δk - T_min) / (T_max - T_min))
```

Where:
- `I_k` = Normalized downturn indicator for metric k ∈ [0, 1]
- `Δk` = Percentage change of metric k over 30 days
- `T_min`, `T_max` = Threshold bounds
  - TRX: T_min = -10%, T_max = 0%
  - Energy: T_min = -15%, T_max = 0%

#### Step 2: Target Price Calculation

```
P_target = P_base - (P_base - P_floor) × min(1, I_TRX + I_Energy)
```

Where:
- `P_base` = 210 sun (Base Price)
- `P_floor` = 70 sun (Price Floor, 33% of base)
- Combined indicator sum determines discount depth

#### Step 3: Smoothed Price Implementation

```
P_t = P_t-1 + α × (P_target - P_t-1)
```

Where:
- `α` = 0.5 (smoothing factor preventing sharp transitions)
- `P_t-1` = Previous period price
- Exponential smoothing ensures predictable price evolution

### Implementation Architecture

#### Core Components

1. **Data Collection Module**
   - Automated TRX price monitoring (30-day rolling average)
   - Energy consumption tracking (monthly aggregation)
   - Market state classification engine

2. **Price Calculation Engine**
   - Real-time indicator computation
   - Target price determination
   - Smoothed price output generation

3. **Integration Points**
   - Modification of existing `sun_per_energy_unit` parameter
   - Block production price update mechanism
   - Historical logging for model refinement

#### Configurable Parameters

Super Representatives can adjust:

```javascript
{
  "enable_dap_model": true,           // Boolean activation flag
  "dap_base_price": 210,              // Base price in sun
  "dap_price_floor": 70,              // Minimum price floor
  "dap_smoothing_factor": 0.5,        // Adjustment smoothing
  "dap_trx_threshold_min": -10,       // TRX downturn threshold (%)
  "dap_energy_threshold_min": -15,    // Energy downturn threshold (%)
  "dap_update_frequency": 86400       // Update interval in seconds
}
```

## Rationale

### Backtesting Results and Economic Validation

Comprehensive analysis from September 19, 2024 demonstrates DAP model effectiveness:

#### Performance Summary

| Metric | Static Model | DAP Model | Absolute Change | Relative Change |
|--------|-------------|-----------|----------------|-----------------|
| **Total Revenue** | 10.41T sun | 11.12T sun | +0.71T sun | **+6.85%** |
| **Average Price** | 210.00 sun/eu | 197.10 sun/eu | -12.90 sun/eu | **-6.14%** |
| **Total Consumption** | 49.55T eu | 57.81T eu | +8.26T eu | **+16.66%** |

#### Visual Evidence from Backtesting

The model's effectiveness is demonstrated through three key visualizations from comprehensive backtesting:

![Figure 1: Price Comparison](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*hASIWYsFG1jelSzU3FG91Q.png)
*Figure 1: Historical static price (210 sun) vs. dynamic DAP model price, demonstrating strategic discounts during market downturns*

![Figure 2: Revenue Comparison](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*UeeDWHREP7Gn7nn8KnWysw.png)  
*Figure 2: Monthly revenue comparison showing that demand stimulation compensates for lower average pricing*

![Figure 3: Market State Dynamics](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*x6cIa_REScowmro21jmvTA.png)
*Figure 3: DAP price dynamics vs. target price, with shaded areas illustrating "missed opportunities" where static pricing was suboptimal*

#### Economic Interpretation

The backtesting results demonstrate a counterintuitive but economically sound outcome:

**By reducing the average resource price by 6.14%, the DAP model increases total revenue from TRX burning by 6.85%.**

This effect is achieved exclusively through significant network activity stimulation. Strategic price reductions during elastic demand periods lead to a 16.66% increase in Energy consumption, which more than compensates for the lower average price, creating a net positive effect for the ecosystem.

### Competitive Analysis

#### Comparison with Alternative Approaches

| Strategy | Revenue Impact | Network Growth | Implementation | Predictability |
|----------|---------------|----------------|----------------|----------------|
| **Static Reduction** | Negative | Positive | Simple | High |
| **Market-Based Pricing** | Variable | Variable | Complex | Low |
| **DAP Model** | **Positive** | **Positive** | **Moderate** | **High** |

#### Relationship to TIP #789

The DAP model complements rather than competes with immediate fee reduction proposals:

- **TIP #789**: Provides immediate relief through static reduction
- **DAP Model**: Delivers systematic long-term optimization
- **Sequential Implementation**: TIP #789 for urgent relief, DAP for sustained efficiency

### Technical Risk Management

#### Fallback Mechanisms

1. **Anomaly Detection**: Automatic reversion to static pricing if calculation anomalies detected
2. **Manual Override**: Super Representative emergency intervention capability  
3. **Circuit Breakers**: Price bounds enforcement during extreme market conditions

#### Monitoring Infrastructure

1. **Real-time Dashboards**: Model performance and state classification visibility
2. **Automated Alerts**: Unusual behavior detection and notification system
3. **Historical Analysis**: Continuous backtesting against new data for model refinement

## Economic Impact Assessment

### Network Economics

**Staking Incentives**: The model preserves fundamental staking economics while optimizing operational expenditure pricing. Long-term capital commitment remains advantageous for regular users.

**Market Dynamics**: Enhanced price responsiveness improves TRON's competitive positioning by maintaining cost advantages while optimizing for varying demand conditions.

### Security Implications

**Spam Prevention**: Economic barriers against network abuse remain intact through maintained price floors and inelastic demand preservation.

**Deflationary Pressure**: Increased TRX burning through optimized pricing enhances token economics and network security funding.

## Compatibility and Migration

### Backward Compatibility

- No smart contract modifications required
- Existing wallet integrations continue functioning
- Energy calculation methods remain unchanged
- dApp cost estimation logic preserved

### Future Extensibility

The DAP framework supports future enhancements:
- Bandwidth resource integration
- Cross-chain fee coordination mechanisms
- Advanced economic parameter optimization

## Conclusion

The Dynamic Administrative Pricing model represents a systematic evolution of TRON's economic architecture. By leveraging empirically observed demand elasticity asymmetry, DAP delivers:

1. **Proven Economic Benefits**: +6.85% revenue increase with +16.66% network growth
2. **Automated Optimization**: Reduced governance overhead through intelligent price adjustment
3. **Strategic Positioning**: Enhanced competitive advantage through sophisticated pricing
4. **Risk Management**: Comprehensive fallback mechanisms and monitoring infrastructure

Implementation positions TRON as a leader in blockchain economic optimization while delivering immediate practical benefits to users, developers, and the broader ecosystem.

The model's mathematical foundation, empirical validation, and comprehensive implementation plan provide a robust framework for sustainable network growth and revenue optimization.

---

**References:**
- [DAP Model Research Article](https://medium.com/@anton_erpulev/dynamic-administrative-pricing-dap-model-for-energy-in-the-tron-network-cda85d40b97b) - Comprehensive analysis with detailed backtesting results and visualizations
- TRON Network Documentation
- Historical Energy Pricing Data Analysis
