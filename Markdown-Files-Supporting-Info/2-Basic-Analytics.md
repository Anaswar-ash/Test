Section 2 – Basic Analytics:Proposed Flow
2.1 Goal & Overview (markdown only)
State the three pillars: Market Activity, Execution Efficiency, Profitability

List all derived attributes I will define and justify each:

fill_rate = TRADE / NEW per symbol

minute = ts_seconds // 60

cancel_to_trade = CANCEL / TRADE per minute

notional = price × quantity

State which DataFrame is used where (df_clean vs df_dir)

2.2 Market Activity (code + markdown)
Answers: "Produce a set of analytics to describe the data"

Total volume by instrument → bar chart

Event type distribution → bar chart

Side distribution buy vs sell → bar chart

Summary table per symbol (events, trades, volume, trade share %)

2.3 Execution Efficiency: Fill Rate (code + markdown)
Answers: "Define attributes not directly observable"

Derive fill_rate = TRADE / NEW per symbol

Justify: "Fill rate is not directly recorded; I approximate it as the ratio of TRADE to NEW events per instrument as a proxy for execution quality"

Horizontal bar chart per symbol

Short interpretation

2.4 Time Profile of the Session (code + markdown)
Answers: "Not immediately obvious insight"

Derive minute = ts_seconds // 60

Derive cancel_to_trade = CANCEL / TRADE per minute

Two charts:

Trades vs Cancellations line chart

Cancel-to-trade ratio line chart

Clearly label the non-obvious insight:

2.5 Notional Value Analysis (code + markdown)
Answers: "Define attributes not directly observable"

Derive notional = price × quantity

Justify: "Notional captures the real economic value of each trade and is more meaningful than raw quantity alone, especially when comparing across instruments with very different price levels"

Notional distribution by instrument → boxplot

Top 10 largest notional trades → table

2.6 Strategy PnL Leaderboard (code + markdown)
Answers: "Produce analytics to describe the data"

Take last inception_pnl per endtag sorted by time

Green/red bar chart

Short interpretation: winners vs losers

2.7 Summary of Findings (markdown only)
Two to three sentences tying all findings together

Restate the non-obvious insight clearly

Forward reference:

text
> Abnormal periods identified in the time profile are explored
> in depth in Section 3.
Brief requirement checklist
Brief Requirement	Where Covered
Produce analytics to describe the data	2.2, 2.3, 2.6
Include visualisations	Every subsection
Define non-observable attributes with justification	2.3, 2.4, 2.5
At least one non-obvious insight	2.4 clearly labelled
