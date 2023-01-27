---
layout: post
title: Calculating Savings Plan utilization and coverage from AWS Cost and Usage Reports
date: 2021-11-26 16:10 -0500
tags:
  - AWS
  - Billing
  - Cost and Usage Reports
---

<small>Reading time: 20 minutes</small>

In [AWS Cost and Usage Reports](https://aws.amazon.com/aws-cost-management/aws-cost-and-usage-reporting/), the accounting of [savings plans](https://aws.amazon.com/savingsplans/) and the usage that is or is not covered by those plans can be a little tricky. There are a number of different [line items](https://docs.aws.amazon.com/cur/latest/userguide/cur-sp.html#cur-sp-lineitems) that collectively provide a complete picture of actual usage, how the cost of that usage is affected by savings plans, and how the bill is affected by those costs.

- [Data dictionary](https://docs.aws.amazon.com/cur/latest/userguide/data-dictionary.html)

_Note: this article assumes some familiarity with both the general structure of Cost and Usage Reports, and some fundamentals of AWS billing, such as blended vs. unblended costs._

## Types of line items

### lineItem/ProductCode – ComputeSavingsPlans

There will be line items with a **lineItem/ProductCode** of `ComputeSavingsPlans` for each savings plan in your inventory for each period covered by the report (i.e., each day for daily reports, each hour for hourly reports). So if you have one savings plan, there will be one `ComputeSavingsPlans` line item per period. If you have 10 savings plans, there will be 10 `ComputeSavingsPlans` line items. These lines exist even for periods in the future, and the values are updated over time as newer reports are generated.

The details of these line items will vary depending on the plans’ payment options. AllUpfront plans will have blended and unblended costs of $0 (since they were already paid for, and have no further actual impact on the bill). Plans with other payment options will contribute real costs to the bill each period, and the blended and unblended costs for their line items will reflect that.

The **lineItem/UsageAmount** indicates the number of hours of savings plan commitment each line item represents. Line items for the current day in daily repots (i.e., today at the time the CUR was generated) will still have a usage amount of 24, but will reflect only some hours of the day, as we’ll see below.

### lineItem/LineItemType – Usage & SavingsPlanCoveredUsage

Usage line items for resources that are eligible for savings plan come in two flavors: `Usage` and `SavingsPlanCoveredUsage`. The former is used for line items that are actually billed at on-demand rates (i.e., not covered by a savings plan), and the latter is used for line items whose cost is covered by a savings plan. `SavingsPlanCoveredUsage` line items include the cost of the usage both before and after the savings plan discount is applied.

The costs after the plan discount is applied may indicate a cost that has already been paid, such as with AllUpfront savings plan purchase options. For example a `SavingsPlanCoveredUsage` line item may show an on-demand cost of $100, and a discounted cost of $50, but that $50 may have been paid for at the time the savings plan was purchased. This is an indication of the _value_ of the usage, not an amount being billed.

### lineItem/LineItemType – SavingsPlanNegation

Some or all of the costs indicated in each `SavingsPlanCoveredUsage` line item are offset with a `SavingsPlanNegation` line item, which has a negative cost. A single `SavingsPlanNegation` line items may offset costs from multiple `SavingsPlanCoveredUsage` line items.

For example, `SavingsPlanCoveredUsage` line item for a NoUpfront plan may show an on-demand cost of $100, and a discounted cost of $75. The $25 difference would appear as a `SavingsPlanNegation` line item with a `-25` cost, resulting in $75 real billed cost. For an AllUpfront savings plan, the `SavingsPlanNegation` cost would be `-100`, since the usage wouldn't add any real costs to the current bill, since it was pre-paid.


## Savings Plan utilization

The only line items needed to calculate savings plan utilization from a CUR are those where **lineItem/ProductCode** product code is `ComputeSavingsPlans` and the **lineItem/LineItemType** is `SavingsPlanRecurringFee`. And the only columns needed from these line items are **savingsPlan/UsedCommitment** and **savingsPlan/TotalCommitmentToDate**.

> _**Note:** the **TotalCommitmentToDate** column is the sum of the **AmortizedUpfrontCommitmentForBillingPeriod** and **RecurringCommitmentForBillingPeriod** columns. Whether commitment is recurring or amortized for each line item depends on its payment option, but the distinction is irrelevant in this case._

Each of these commitment columns represents a dollar amount corresponding to the hourly spend commitment that was selected when a savings plan was purchased. Savings plans are always based on spend commitments, not on resource usage, like hours for EC2 instances or memory-duration for Lambda functions.

The **TotalCommitmentToDate** value is a fixed value based on the hourly commitment chosen when purchasing the savings plan. Every line item in CUR for this savings plan for the lifetime of the plan will have that same **TotalCommitmentToDate** (which will be the hourly commitment for hourly CUR, or 24 times the hourly commitment for daily CUR). This value is the maximum spend that can be covered by the savings plan represented in that line item, for the given period.

The **UsedCommitment** value represents how much spend was _actually_ covered by the particular savings plan represented in that line item. It reflects the value of usage during the line item's period, regardless of when the cost shows up on a bill (i.e., it's not skewed by upfront costs). The closer this value gets to the **TotalCommitmentToDate** value, the higher the utilization of the savings plan.

Consider a scenario where you have a savings plan with a $0.75 hourly commitment, in a daily CUR, this would appear as line items with a fixed **TotalCommitmentToDate** value of `18` per line (0.75 * 24 = 18). Assume this is your only savings plan.

If there was a day where your account used no savings plan-eligible resources (i.e., no EC2 intances, no Lambda invocations, and no Fargate tasks), the **TotalCommitmentToDate** for that day's line item would be `18` (because it's a fixed value), and the **UsedCommitment** would be `0`, since there were no resource costs that could be covered by the commitment. This would be a utilization rate of 0%, which can be calcualted with `UsedCommitment / TotalCommitmentToDate`, or `0 / 18 = 0`.

If, on another day, there was significant plan-eligible usage (say, many thousdands of hours of EC2 instances running), the **TotalCommitmentToDate** would still be `18`, and the **UsedCommitment** would also be `18`, meaning that the savings plan covered as much of that usage as it could up to the commitment limit. The same calculation of `UsedCommitment / TotalCommitmentToDate` now gives `18 / 18 = 1`, or 100% utilization.

Because **UsedCommitment** reflects actual, accumulated usage, the line item for the current period (i.e., the hour or day during which the CUR was generated), will have a **UsedCommitment** value that changes over time. In other words, even for a day with 100% utilization, if a daily CUR is generated at noon, **UsedCommitment** would only show `9` in the previous example, and would be updated to be `18` once the entire day had elapsed. Therefore, it's best to only include line items for complete days or hours in the past, to avoid calculating artificially-low utilization rates for incomplete periods.

For a day where there was only enough resource usage to cost $0.25/hour (with the savings plan discount), you would see a **UsedCommitment** of `6` (`0.25 * 24 = 6`), yielding a utilization of `6 / 18 = 0.333`, or 33%.

Realistically, you will have multiple savings plan. Take care not to `sum` utilization rates of individual savings plans. For example, if you have two savings plans with 100% and 20% utilization rates respectively, it wouldn't make sense to add those rates together to come up with a 120% utilization rate. How you choose to aggregate data across savings plans will depend on your application and what you're trying to convey.

### QuickSight

To recreate the **Savings Plans utilization (% utilization)** graph from the Utilization report in Cost Explorer inside a QuickSight dashboard, start by creating a bar or line chart with filters for:

- **line_item_product_code** equals `ComputeSavingsPlan`
- **line_item_line_item_type** equals `SavingsPlanRecurringFee`

Create the calculated field:

`sum({savings_plan_used_commitment}) / sum({savings_plan_total_commitment_to_date})`

And set the field wells to:

- X axis: `line_item_usage_start_date`
- Value: `yourUtilizationRateCalculatedField`
- Group/Color: `line_item_product_code`

## Savings plan coverage

Calculating coverage rate for savings plans looks very different than utilization rate. Instead of looking at the `ComputeSavingsPlans` line items, we have to look at the individual service usage line items, like those for EC2 instance hours, or Lambda memory-duration.

For each service, the CUR will include separate line items for usage that was covered by a savings plan and usage that wasn't (i.e., on-demand usage). By comparing the costs of these two types of line items, we can determine what portion of total spend was covered by a saving plan.

Line items for usage that was acutally billed at on-demand rates will have a **lineItem/LineItemType** of `Usage`. For these lines, the **lineItem/BlendedCost** will be the billed amount for the resource usage.

Line items for usage that was covered by a savings plan will have a **lineItem/LineItemType** of `SavingsPlanCoveredUsage`. These lines will include a **lineItem/BlendedCost**, but it will represent the on-demand cost of that usage **if it hadn't** been covered by a savings plan. The **savingsPlan/SavingsPlanEffectiveCost** column provides the cost of that usage after the savings plan is applied.

For a given period, to determine the savings plan coverage you would take the total cost of all savings plan-eligible resource, and figure out what percent of that value only the savings plan covered line items makes up. By convention, this calculation is done with the **lineItem/BlendedCost** of both on-demand and discount line items. In this way, if you have $150 of on-demand usage, and some savings plan-covered usage that had an effective cost of $200 but an uncovered cost of $250, the calculation would be `250 / (150 + 250) = 0.625`, for a coverage rate of 62.5%. Which is to say, the coverage rate is based on the on-demand pricing of all resources, even those actually covered by savings plans.

### QuickSight

To recreate the **Savings Plans coverage (% of dollars spent)** graph from the Coverage report in Cost Explorer inside a QuickSight dashboard, start by creating a bar or line chart with filters for:

- **line_item_product_code** equals `AmazonEC2` or `AmazonECS` or `AWSLambda`
- **line_item_usage_type** equals `Fargate-GB-Hours` or `Fargate-vCPU-Hours` or `BoxUsage` or `Lambda-GB-Seconds`

Create the calculated field:

`percentOfTotal(sum({line_item_blended_cost}), [{line_item_usage_start_date}])`

And set the field wells to:

- X axis: `line_item_usage_start_date`
- Value: `yourPercentOfTotalCalculatedField`
- Group/Color: `line_item_line_item_type`

This will results in a chart with both `Usage` and `SavingsPlanCoveredUsage` groups. To get it to only show the percent value for savings plans, change the **Number of bar groups displayed** value in the **Group/Color** settings to `1`, and sort the chart by `line_item_line_item_type` ascending (a-z).

## On-Demand spend equivalent

The **On-Demand spend equivalent** value from the Utilization report in Cost Explorer is simply the sum of all **lineItem/BlendedCost** values where the **lineItem/LineItemType** equals `SavingPlanCoveredUsage`. It represents what resources that were covered by a savings plan would have cost without the savings plan.

## Savings Plan spend

The **Savings Plan spend** value from the Utilization report in Cost Explorer is simply the sum of all **savingsPlan/SavingsPlanEffectiveCost** values where the **lineItem/LineItemType** equals `SavingPlanCoveredUsage`. It represents the realized cost of all savings plans, even if those costs were pre-paid previously.

## On-Demand spend not covered

The **On-Demand spend not covered** value from the Coverage report in Cost Explorer is the sum of all **lineItem/BlendedCost** values where the **lineItem/LineItemType** equals `Usage`, for line items that are eligible for savings plans (EC2 instance hours, Lambda memory-duration, etc). It represents the cost of resources that were billed at on-demand prices, but could have been covered by a savings plan if there were more savings plan commitment available.
