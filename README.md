# Time-of-Day & Day-of-Week Pattern Analysis

Identifying the peak hours, weekend and weekday differences and seasonal trends in EV charging demand, to guide pricing, staffing and supply decisions.

## The Question

When do customers charge, and does it change by day of the week, season, region or charger type? This project measures the hourly and weekly rhythm of demand, tests for seasonal and holiday effects, checks how the time-of-use price lines up with demand, and compares behaviour across regions and charger types. Station numbers grew from 8 to 33 over the period, so every comparison over time uses sessions per station, not raw session counts.

## Repo Structure

```
├── notebooks/        → full analysis notebook
├── output/           → CSV outputs
├── dashboard/        → Power BI .pbix file + page screenshots
├── slides/           → presentation deck
└── documentation/    → project documentation and insight summary
```

## Data

The source is the EV charging dataset: `stations` (33 stations in 9 regions), `chargers` (199 chargers: 95 rated 50 kW, 69 rated 150 kW and 35 rated 300 kW) and `sessions` (294,024 sessions, January 2022 – December 2024). The raw file is not included in this repo. It is hosted as a Kaggle Dataset: https://www.kaggle.com/datasets/sanyazuberi/ev-charging-raw-dataset.

There are no missing values, every one of the 1,096 days has sessions, and total cost equals price per kWh times energy for every session.

Output files (in `output/`):
- `hourly_demand.csv`: share of sessions by hour, with its tier (peak, shoulder or off-peak), weekday and weekend shares, and average price per kWh
- `weekday_hour_demand.csv`: sessions starting per station per hour for each weekday and hour
- `daily_demand.csv`: one row per day: sessions, stations open and sessions per station
- `seasonal_index.csv`: monthly seasonal index with its 95% interval
- `peak_offpeak_summary.csv`: share of sessions, energy and revenue by tier, with session rates and estimated chargers in use
- `pricing_by_year.csv`: base price, premium price and markup, sessions per station and revenue per station by year
- `region_behaviour.csv`: weekend ratio, peak share, premium-window share, price index and energy per session by region
- `region_hourly_profile.csv`: share of each region's sessions by hour
- `charger_type_behaviour.csv`: session length, energy, revenue, timing and busy share at peak by charger type
- `charger_type_hourly_profile.csv`: share of each charger type's sessions by hour

## How Patterns Were Measured

- **Sessions per station per day.** Each day's sessions divided by the stations open that day (a station counts as open from 1 January of its activation year). This removes the effect of new stations opening.
- **Peak, shoulder and off-peak hours.** Each hour's share of sessions is compared with the average hour (4.17%). Peak hours have at least 1.5 times the average, off-peak hours at most half, and the rest are shoulder hours.
- **Weekday and weekend.** Monday to Friday against Saturday and Sunday. The weekend-to-weekday ratio is sessions per weekend day divided by sessions per weekday.
- **Seasonality.** Each day is divided by the average for its weekday, then averaged by month. The seasonal index is a month's average divided by the overall average. A one-way test checks for any month effect, seasons are compared, a trend line is fitted, and Christmas Day and New Year's Day are compared with a normal day of the same weekday.
- **Pricing.** Prices are compared inside and outside the premium window (16:00 to 20:59) and across years, with the implied price elasticity (change in sessions per station divided by change in price). A what-if moves the premium to the five busiest hours, assuming customers do not change their behaviour, and works out the fall in sessions that would cancel the gain.
- **Chargers in use.** Estimated by multiplying sessions starting per station per hour by the average session length (35 minutes), then comparing with the average of 6 chargers per station. This does not merge overlapping sessions.
- **Region and charger-type comparisons.** Chi-square tests check whether the hourly pattern and the weekend share differ by more than chance. A price index (each session's price divided by the network average for its year) stops regions that opened in a higher-priced year from being misread.

## Key Findings

- **Demand has a strong daily rhythm.** The 8 peak hours (09:00 to 16:59) carry 61.8% of sessions, the 7 shoulder hours (06:00 to 08:59 and 17:00 to 20:59) 28.1%, and the 9 off-peak hours (21:00 to 05:59) only 10.0%. The busiest hours are 10:00 to 14:59 (41.0% together) and the quietest are 22:00 to 04:59, each taking 0.9%.
- **Weekends are 26% lower than weekdays,** with the same hourly shape. Weekdays average 15.02 sessions per station per day and weekend days 11.05, and the ratio is stable across years (0.742, 0.735 and 0.730).
- **Chargers are idle most of the time.** Even in the busiest hour (Monday 13:00, 1.25 session starts per station per hour) only about 12% of an average station's chargers are in use. The quietest time (Sunday 04:00) sees 0.09 session starts per station per hour.
- **There is no seasonality.** The monthly index runs from 0.988 to 1.006, a range of under 2%, and a test finds no month effect (p = 0.86). The seasons are indistinguishable and the three-year trend (-0.32% a year) is not significant (p = 0.12).
- **There is no holiday dip.** Christmas Day and New Year's Day ranged from 0.95 to 1.06 of a normal day of the same weekday, within the usual day-to-day spread of about 6%.
- **Raw growth is new stations.** Sessions rose from 40,640 in 2022 to 86,132 in 2023 and 167,252 in 2024, but only because stations grew from 8 to 17 to 33. Sessions per station per day stayed at 13.92, 13.88 and 13.85.
- **The price premium misses the demand peak.** A premium of about 10% applies from 16:00 to 20:59, the same on weekdays and weekends. It covers 22.7% of sessions and 24.4% of revenue, while the peak tier overlaps it in one hour only (16:00).
- **No response to price is visible.** The average price rose 25% in 2023 and fell 8% in 2024, yet sessions per station were almost unchanged (implied elasticity -0.01 and +0.03, with no consistent direction). This rests on three yearly price points.
- **Regions behave identically.** In every region, sessions per station per day are 13.76 to 14.00, the weekend-to-weekday ratio is 0.72 to 0.74 and the peak-hour share is 61.4% to 62.2%. The hourly pattern does not differ by region (p = 0.94), nor does the weekend share (p = 0.69). Energy per session does differ (33.0 kWh in Wales to 52.0 kWh in South West), but that comes from the charger mix.
- **Charger types share the same timing but not the same sessions.** A 50 kW session lasts 49.5 minutes and delivers 33.0 kWh, against 24.5 minutes and 49.0 kWh at 150 kW and 19.5 minutes and 75.5 kWh at 300 kW. A 50 kW charger is busy 15.9% of weekday peak time, against 7.8% and 6.8%.

## Peak and Off-Peak Usage

| Tier | Hours | Share of sessions | Share of revenue | Weekday session starts per station per hour | Chargers in use per station |
|---|---|---|---|---|---|
| Peak | 09:00 to 16:59 | 61.8% | 61.1% | 1.16 | 0.68 (11.3%) |
| Shoulder | 06:00 to 08:59, 17:00 to 20:59 | 28.1% | 29.1% | 0.60 | 0.35 (5.9%) |
| Off-peak | 21:00 to 05:59 | 10.0% | 9.8% | 0.17 | 0.10 (1.6%) |

## Pricing and Demand

| Year | Base price per kWh | Premium price per kWh | Sessions per station per day | Revenue per station per day |
|---|---|---|---|---|
| 2022 | 0.500 | 0.551 | 13.92 | 352 |
| 2023 | 0.625 | 0.687 | 13.88 | 417 |
| 2024 | 0.575 | 0.632 | 13.85 | 377 |

Moving the 10% premium to 10:00 to 14:59 (41.0% of sessions and 40.0% of revenue) and removing it from 16:00 to 20:59 would raise revenue by 1.8% if customers did not change their behaviour. The gain disappears if sessions in those five hours fall by more than 4.1%. Since no response to price can be seen in this data, it cannot say whether the move would shift any demand out of the peak.

## Behaviour by Charger Type

| Charger type | Session length | Energy per session | Revenue per session | Busy at weekday peak |
|---|---|---|---|---|
| 50 kW | 49.5 min | 33.0 kWh | 19.59 | 15.9% |
| 150 kW | 24.5 min | 49.0 kWh | 28.95 | 7.8% |
| 300 kW | 19.5 min | 75.5 kWh | 44.56 | 6.8% |

The hour-by-hour test across charger types is borderline (p = 0.046), but the largest difference in any hour is 0.42 percentage points, and grouped into peak, shoulder and off-peak tiers there is no difference (p = 0.97). With 72 hour-by-type comparisons, one borderline result is expected by chance, so the timing is treated as the same.

## Recommendations

**Pricing**
1. **Pilot moving the premium to the demand peak** (10:00 to 14:59) at a few stations. The static gain is 1.8%, and the effect on peak demand cannot be predicted because no price response is visible.
2. **Do not read the lack of price response as permission for large rises.** It rests on three yearly price points, so test small changes first.

**Staffing and maintenance**
3. **Concentrate attended support on weekdays from 09:00 to 16:59** (62% of sessions). Cover can be lighter at weekends (26% fewer sessions) and overnight (10% of sessions).
4. **Schedule planned maintenance in the quietest window,** weekends from 00:00 to 04:59 (about one session start per station every 10 hours), and avoid weekdays from 10:00 to 14:59.

**Supply**
5. **No time of day needs extra chargers.** Even in the busiest hour only about 12% of an average station's chargers are in use.
6. **Do not plan around seasons or holidays.** Plan on weekday, weekend and hour patterns, and on the number of stations.
7. **One network-wide tariff and rota is enough,** since regions and charger types share the same timing.
8. **Watch 50 kW chargers first for congestion,** since they are busy 15.9% of weekday peak time.
9. **Monitor monthly** the peak-hour share (61.4% to 62.2% across regions), the weekend-to-weekday ratio (0.72 to 0.74) and sessions per station per day (about 13.9). A move outside these ranges means behaviour has changed.

## Dashboard

Built in Power BI, four pages:
- **Hourly & Weekly Demand**: KPI cards, share of sessions by hour coloured by tier, weekday vs weekend hourly profile, sessions per station by weekday, and a weekday-by-hour heatmap
- **Peak, Off-Peak & Pricing**: share of sessions and revenue by tier, chargers in use by tier, share of sessions against price by hour, base and premium prices by year, and sessions per station by year
- **Seasonal Trends**: a year slicer, sessions per station by month and year, raw sessions by month, and the seasonal index with its 95% interval
- **Region & Charger Type**: hourly heatmap by region, weekend ratio and peak share by region, hourly profile by charger type, session length and peak busy share by charger type, and two summary tables

See `dashboard/` for the `.pbix` file and page screenshots.

## Reproducing This Project

1. Download the raw dataset from the Kaggle link above.
2. Run the notebook in `notebooks/` from top to bottom in a fresh session. It writes the CSV files and the `figures/` folder.
3. Open the `.pbix` in `dashboard/` and refresh the data sources if the CSV paths have changed.

## Tools Used

Python (pandas, NumPy, SciPy, Matplotlib, Seaborn) · Kaggle Notebooks · Power BI

## Limitations

- The dataset appears simulated. Every station earns the same sessions a day all year, with no seasonal or holiday effects and no response to price, so real-world patterns may differ.
- Pricing results rest on three yearly price points, and the what-if assumes customers do not change their behaviour.
- Chargers in use are estimated from average session length, and busy time is not merged for overlapping sessions.
- Available time assumes each station operates from 1 January of its activation year, because the installation dates conflict with session activity.
- Revenue figures are in the dataset's price units. Staffing levels, costs and queue data are not in the data, so the staffing and maintenance recommendations rest on session counts only.
- Region and charger-type comparisons are over the whole three years. Wales has one station and Yorkshire & Humber two, so their patterns rest on few sites.
