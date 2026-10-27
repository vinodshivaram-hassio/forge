# DR-002: Direction of the cost dial

Status: Proposed · 2026-10-27 · Brief open question 2

## Question

Which way does the cost dial run?

## Choice

1 is cheapest, 10 is best quality regardless of spend. Every run summary and every trace routing entry prints the dial as `dial: 3/10 (1 = cheapest)`.

## Reasons

- Matches the routing formula in brief section 13, where a higher dial raises the tier.
- Printing the scale every time removes the reversed-dial surprise.

## Rejected alternatives

- 1 as best (the "priority 1" convention): conflicts with the formula and with "turn it up for more".
