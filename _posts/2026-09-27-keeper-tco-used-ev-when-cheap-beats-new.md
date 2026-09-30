---
title: "Second hand car with no plan to resale"
categories:
  - Blog
tags:
  - finance
  - decision-making
  - EV
  - TCO
---


I recently bought a second hand car without a plan to resale but with the plan to keep it as long as it is financially the better option. 

Most car TCO write-ups assume you will sell. Purchase price is a sunk cost the day you drive off. Resale value is optional upside, not the base case. That changes both how you amortize the car and when a repair is “worth it.”

### The seller heuristic vs the keeper rule

The usual advice says: do not repair if the bill exceeds the car’s market value. That is a trader rule. It protects the asset you intend to exit.

I ask a different question. Suppose the car cost €12,000 and you keep it for 10 years, 120 months. That is €100 per month. A €1,200 repair that buys one more year does not stack another €100 on top of the €100. The repair comes on top of the price, and the year comes on top of the life: €13,200 over 132 months, which is still €100 per month. Care that you pay every month continues on top of that. Keep repairing when this extended €/month stays below the €/month of buying another car. The market value of the current car does not enter the comparison.

The hard cliff for a small EV is usually not a cascade of engine failures, it is rather a failing battery or broken chasis.

### How long does the battery last *for my use*?

I drive about 9,000 km per year. At that mileage, calendar aging dominates cycle wear. I am fine with 80 km of winter range. With a Spring-class WLTP near 230 km, a plausible winter range when new is around 115 km. That implies a usability floor near 70% State of Health (SoH). 

Since Battery capacity decay exponentially and assuming that 4 years ago the SoH was 100 % we get 11.4 years of remaining lifetime.

```text
SoH(t) = exp(−λ t)
λ = −ln(0.91) / 4 ≈ 0.0236 per year
years left = ln(0.91 / 0.70) / λ ≈ 11.4 years
```

So the purchase decision is not will the battery die next year? It is whether amortizing €9,490 over that remaining life, plus ordinary care and expected repairs, beats paying more for a newer Spring and enjoying lower early repair risk.

### What else belongs in keeper €/month?

Amortizing purchase to a soft end-of-life is only the first term. The rest of the stack is:

- Scheduled care: annual inspection, TÜV share, cheap city tires, fluids, 12V battery, filters.
- Rust treatment: a garage quote of about €2,000, expected to last at least ten years (~€17/month).
- Expected unplanned repairs, axle, suspension etc.
- Battery pack keep as a decision node, not a smear of €50/month forever. 

Energy, insurance, and tax are real, but they are similar across these near-substitutes. Thus, a new car will often not get much of a discount on energy, insurance or taxes, and therefore can be left out of the equation.

### The comparison

Four listings on purchase day, anonymized:

| Car | Price | Age (approx.) | km | SoH |
|-----|------:|--------------:|---:|----:|
| Cheap used 45 (bought) | €9,490 | 4.4y | 52,800 | 91% measured |
| Mid used 45 | €12,470 | 3.2y | 11,900 | estimated from age |
| Nearly new 65 | €15,990 | 1.7y | 4,500 | estimated |
| New 100 | €19,990 | 0 | 0 | 100% |

On purchase price amortized to the SoH/winter floor alone, the cheap used car lands near €70/month; the new one near €110/month. Adding scheduled care and the expected chassis-heavy repair bands raises everyone’s total, but under those defaults it still does not crown the new car.

You can stress the assumptions yourself:

<iframe src="/assets/apps/ev-keeper-tco-simulator.html" width="100%" height="1320" frameborder="0" loading="lazy" title="Keeper EV TCO Simulator"></iframe>

### What would overturn the cheap buy?

A few honest failure modes:

1. Winter floor is reached faster than expected.
2. A failed prevention. For example, skipping rust prevention might make repairing for resulting damage way to high.

### The point

If you never resell, “clever” means lowest keeper €/month over the life you will actually use, not the highest residual value in year three. 

The useful discipline is simple: amortize what you paid, budget the boring care, and treat the battery as a late  decision, not a reason to buy new by default.
