---
title: "How Much to Save for Projects That Cannot Start Today"
categories:
  - Blog
tags:
  - finance
  - investing
  - planning
---

A project you cannot start yet is a bill with a date. The car has a price today and a month you would rather buy it. The kitchen does too. So does a down payment. Between now and that month the bill grows with inflation, and each month's saving can sit in Tagesgeld, in an obligation held to a date, or in an equity ETF. How much you save, and how that saving splits, is one decision.

### The bill on the date you want

One inflation rate applies to every project. A cost due in t years is the cost today times (1 + inflation) raised to t. The rate sits on one slider.

### The obligation that matures on the hard date

A hard date a year or more away is matched to its own obligation, bought to mature in that month and held until then. Holding to that month is the safe period: the yield you locked is what you receive, and the price in the months before never has to be realised. A kitchen due in five years, on a date that cannot move, is covered by a five-year obligation. That bond is there for the kitchen. The car, due sooner, has a shorter bond of its own.

The yield on the slider is the assumption for a high-grade bond held to maturity. The page opens it at 3.5%, between the 2.5% Tagesgeld rate and the 7% drift of the equity path. The same slider can sit below Tagesgeld when the curve is inverted. Default is left out. A fund that keeps rolling bonds forward is a different instrument: it has no maturity on the project's date, so a rise in yields can still change what is there in the month the bill is due.

A bill due inside a year stays in Tagesgeld, in one shared pot.

The monthly amount for the hard dates is the sum of those separate annuities. Money already saved is assigned to the nearest hard bill first, up to that bill's present value. Only what is left over starts the next bond.

On the panel below, with €10,000 already saved, Tagesgeld at 2.5%, the obligation at 3.5%, and inflation at 2%, the starting projects are a €16,000 car in 30 months, a €28,000 kitchen in 60 months, and a €75,000 down payment in 10 years. The car's bill is about €16,800. The kitchen's is about €30,900. The €10,000 fills part of the car's bond, whose present value is about €15,400, so none of it reaches the kitchen. The car still needs about €189 a month. The kitchen, filled from the monthly saving alone, needs about €472. Together that is **€660**, and the kitchen is the largest piece. The down payment can wait, so it is outside that figure. Buying an obligation for it as well would take **€1,297 a month**.

Once a bond is bought it stays until its date. After that date, its monthly piece stops. The panel keeps taking whatever monthly saving you set, and what the remaining hard dates do not need goes to the ETF.

Below €660 the matched bonds cannot promise every hard date. A date has to move, a scope has to shrink, or part of a hard bill has to be left in the ETF, where a bad stretch of returns can miss it.

### What is allowed to take the risk

The panel starts at €1,300 a month, just above the €1,297 that would match every project with an obligation. At the start, **€660** buys the two bonds and **€640** goes to the ETF, aimed at the down payment.

Across the illustrated path, the hard dates clear in every start month. The down payment is on time in 90% of them. In the worst start, the first month of the path, it is paid **16 months late**, at **€93,871**, after the bill has kept inflating. That wait is the risk the ETF is taking. The second chart is every other start month. The line sits near 100% when the down payment hits its month, and falls to the car and the kitchen's share of the cost when the down payment misses.

The wealth chart is three bands. On these opening projects the Tagesgeld band is empty, because both hard dates are more than a year out. The obligation band builds, drops when the car is paid, continues toward the kitchen, and drops to zero when the kitchen is paid. What remains is the ETF.

The slider "Hard-date saving in the ETF" puts that share of the hard-date saving into the ETF. At zero, the month's hard-date saving buys the obligation. At the opening €1,300 the illustrated path still pays both hard dates with the slider at 100%, because that saving is far above the €660 match. At **€675**, the first step above €660, the slider at zero still clears both hard dates in every start, and the down payment is late in all of them. At 100% the hard dates clear in **64%** of starts, and in the worst window the kitchen is short **€15,205**.

### A path of your own

The page draws a path so it runs before any file is loaded: a 7% drift, 15% volatility, and three crashes of about −50%, −45% and −32%.

A file replaces it. Save UTF-8 CSV as `assets/data/equity-returns.csv`, or load it in the panel. One `date` column (`YYYY-MM`, oldest first, one row per month), then one column of monthly simple returns per history. `0.012` or `1.2%` means +1.2% that month. The header of each column is the name in the dropdown. A column of index levels named `level` is converted. Project rows, from a sheet or a paste, are `name,cost_today,due,flexible`, with `flexible` as `yes` or `no`. Inflation stays the slider.

The panel slides that history under the plan and opens on the start month that pays the least on the ideal dates. The obligation yield stays a slider.

<iframe src="/assets/apps/project-funding-simulator.html" width="100%" height="1680" frameborder="0" loading="lazy" title="Project Funding Simulator"></iframe>

### What to move

The first lines above the chart are the two monthly amounts: the hard dates, each on its own obligation or in Tagesgeld, and the amount that would also match the projects that can wait. Your saving sits next to them. The wealth chart is one start month, split into Tagesgeld, obligations, and the ETF. The lower chart is every other start. A flat line means that, for this saving and this split, the start month does not change what gets paid on time. A dip is a window you would not want a due date to land in.
