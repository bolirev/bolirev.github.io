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

### The obligation that matures on or before the bill

An obligation with no default, bought at or below par and held to its own maturity, locks a pre-tax yield. The price in the months before never has to be realised. The page will use a bond only when that bond matures on or before the bill, and it holds the latest such bond. A bond that matures later would have to be sold, and that sale is not a locked return. If the bond matures first, the months until the bill earn Tagesgeld. A bill with nothing in the list maturing in time stays in Tagesgeld for the whole wait. A fund that keeps rolling bonds forward has no maturity on the bill's date, so a rise in yields can still change the value in the month the bill is due.

The yield is a row in the list, not a single rate: a name, a pre-tax yield, and a maturity. The rows shipped with the page are Bundesbank reference yields on 9 October 2026, and only for Bunds whose price that day, accrued interest included, was at or below 100. The car is due in April 2029 and is matched to the Bund that matures that month, at 3.10%. The kitchen is due in October 2031 and is matched to the Bund that matures that month, at 3.23%. The down payment is due in October 2036, two months after the August 2036 Bund at 3.47%, so a match for it holds that Bund and then leaves the money in Tagesgeld.

Interest on Tagesgeld and on the obligation is taxed every year. On the page the rate is 26.375%, Abgeltungsteuer plus the Solidaritätszuschlag. Interest that has not yet reached a year-end is taxed when the money is spent. The accumulating ETF compounds gross, and the same rate applies to the gain only when the shares are sold. The Sparerpauschbetrag and the equity Teilfreistellung are left out, so small balances and the ETF are taxed more heavily here than under the full rule.

The monthly amount for the hard dates is the sum of those separate annuities, after tax. Money already saved is assigned to the nearest hard bill first, up to that bill's present value. Only what is left over starts the next bond.

On the panel below, with €10,000 already saved, Tagesgeld at 2.5%, and inflation at 2%, the starting projects are a €16,000 car in 30 months, a €28,000 kitchen in 60 months, and a €75,000 down payment in 10 years. The car's bill is about €16,800. The kitchen's is about €30,900. The car bond's present value is about €15,900, so the €10,000 fills part of it and none of it reaches the kitchen. The car still needs about €202 a month. The kitchen, filled from the monthly saving alone, needs about €485. Together that is **€687**, and the kitchen is the largest piece. The down payment can wait, so it is outside that figure. Matching it as well, on the August 2036 Bund, would take **€1,357 a month**.

Once a bond is bought it stays until its date. After that date, its monthly piece stops. The panel keeps taking whatever monthly saving you set, and what the remaining hard dates do not need goes to the ETF.

Below €687 the matched bonds cannot promise every hard date. A date has to move, a scope has to shrink, or part of a hard bill has to be left in the ETF, where a bad stretch of returns can miss it, and where the gain is taxed at the sale.

### What is allowed to take the risk

The panel starts at €1,375 a month, just above the €1,357 that would match every project. At the start, **€687** buys the two bonds and **€688** goes to the ETF, aimed at the down payment.

Across the illustrated path, the hard dates clear in every start month. The down payment is on time in 93% of them. In the worst start, the first month of the path, it is paid **15 months late**, at **€93,716**, after the bill has kept inflating. That wait is the risk the ETF is taking. The second chart is every other start month. The line sits near 100% when the down payment hits its month, and falls to the car and the kitchen's share of the cost when the down payment misses.

The wealth chart is three bands. On these opening projects the Tagesgeld band is empty, because both hard dates have a Bund maturing in the same month. The obligation band builds, drops when the car is paid, continues toward the kitchen, and drops to zero when the kitchen is paid. What remains is the ETF.

The slider "Hard-date saving in the ETF" puts that share of the hard-date saving into the ETF. At zero, the month's hard-date saving buys the matched obligation. At the opening €1,375 the illustrated path still pays both hard dates with the slider at 100%, because that saving is about twice the €687 match. At **€700**, the first step above €687, the slider at zero still clears both hard dates in every start, and the down payment is late in all of them. At 100% the hard dates clear in **67%** of starts, and in the worst window the kitchen is short **€14,439**.

### A path of your own

The page draws a path so it runs before any file is loaded: a 7% drift, 15% volatility, and three crashes of about −50%, −45% and −32%.

A file replaces it. Save UTF-8 CSV as `assets/data/equity-returns.csv`, or load it in the panel. One `date` column (`YYYY-MM`, oldest first, one row per month), then one column of monthly simple returns per history. `0.012` or `1.2%` means +1.2% that month. The header of each column is the name in the dropdown. A column of index levels named `level` is converted. Project rows, from a sheet or a paste, are `name,cost_today,due,flexible`, with `flexible` as `yes` or `no`. Obligation rows are `name,yield,due`. Inflation and the tax rate stay the controls on the page.

The panel slides that history under the plan and opens on the start month that pays the least on the ideal dates.

<iframe src="/assets/apps/project-funding-simulator.html" width="100%" height="2120" frameborder="0" loading="lazy" title="Project Funding Simulator"></iframe>

### What to move

The first lines above the chart are the two monthly amounts: the hard dates, each on its own obligation or in Tagesgeld, and the amount that would also match the projects that can wait. Your saving sits next to them. The wealth chart is one start month, split into Tagesgeld, obligations, and the ETF. The lower chart is every other start. A flat line means that, for this saving and this split, the start month does not change what gets paid on time. A dip is a window you would not want a due date to land in.
