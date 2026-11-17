# Lyft Driver Lifetime Value

How much is a Lyft driver worth, and what makes some drivers worth more than others? This project estimates the **lifetime value (LTV)** of 837 Lyft drivers from 184,209 rides, finds what drives that value, and segments drivers into groups that Lyft can target with different retention and incentive strategies.

The full analysis is in [`lyft-ltv.ipynb`](lyft-ltv.ipynb).

## Tableau Dashboard

*Coming soon:* an interactive Tableau Public dashboard built from the cleaned data in [`Data/Cleaned-Data`](Data/Cleaned-Data).

<!-- Replace this comment with the Tableau Public link and a screenshot, e.g.
[![Dashboard preview](images/dashboard.png)](https://public.tableau.com/app/profile/...)
-->

## Key Findings

- **Average driver LTV is about $2,980** over the roughly three months of data (median $2,959). Together, drivers generated about $2.49 million in revenue.
- **Volume drives value.** Total time spent driving (r = 0.99), rides per day (0.99), total rides (0.99), and total distance (0.98) are almost perfectly correlated with LTV. How long a driver has been on the platform matters much less (r = 0.52).
- **Drivers stay an average of 55 days** between onboarding and their last ride in the data. The data ends while many drivers were still active, so this is a lower bound.
- **A quarter of drivers bring in over half the revenue.** K-means clustering splits drivers into four segments:

| Segment | Drivers | Average LTV | Average rides | Share of revenue |
|---|---|---|---|---|
| Excellent | 54 | $8,425 | 606 | 18% |
| Good | 156 | $5,535 | 408 | 35% |
| Fair | 287 | $3,389 | 256 | 39% |
| Low value | 340 | $597 | 43 | 8% |

**Recommendations:** keep excellent and good drivers on the platform, help fair drivers drive more often, find out why low-value drivers leave early, tailor incentives to each segment, and re-run the segmentation regularly.

## Data

The data comes from the StrataScratch *Lyft Driver Lifetime Value* data project. StrataScratch states that all rides took place in San Francisco and all timestamps are in UTC. Rides run from March 28 to June 26, 2016.

| File | Description |
|---|---|
| `Data/Raw-Data/driver_ids.csv` | One row per driver: `driver_id`, `driver_onboard_date` |
| `Data/Raw-Data/ride_ids.csv` | One row per ride: `ride_id`, `driver_id`, `ride_distance` (meters), `ride_duration` (seconds), `ride_prime_time` (surge %) |
| `Data/Raw-Data/ride_timestamps.csv` | One row per ride event: `ride_id`, `event`, `timestamp` |
| `Data/Cleaned-Data/lyft_clean.csv` | One row per ride, after joining, cleaning, and adding ride features |
| `Data/Cleaned-Data/driver_info.csv` | One row per driver, with LTV, driver features, and cluster label |

Each ride's cost is calculated from this rate card: $2.00 base fare, $1.15 per mile, $0.22 per minute, and a $1.75 service fee. The surge (prime time) percentage applies to the base, distance, and time charges. Fares are capped between $5 and $400.

## Variables Added

### Ride level (`lyft_clean.csv`)

| Variable | Description |
|---|---|
| `ride_distance` | Ride distance, converted from meters to **miles** |
| `ride_duration` | Ride duration, converted from seconds to **minutes** |
| `requested_at`, `accepted_at`, `arrived_at`, `picked_up_at`, `dropped_off_at` | Timestamp of each ride event, pivoted from the `event` column into one column per event |
| `ride_total_cost` | Price of the ride from the rate card, including surge and service fee, capped between $5 and $400 |
| `ride_accept_response_time` | Time from the ride request to the driver accepting it |
| `ride_arrival_time` | Time from the driver accepting the ride to arriving at the pick-up location |
| `ride_wait_time` | Time the driver waits at the pick-up location before the rider gets in |
| `ride_length` | Distance category, split into thirds by ride distance: `short_ride`, `med_ride`, `long_ride` |
| `ride_time` | Time of day the ride was accepted (UTC): `morning_ride` (7:00–15:59), `evening_ride` (16:00–21:59), `night_ride` (22:00–6:59) |
| `ride_date` | Date the ride was accepted |

### Driver level (`driver_info.csv`)

| Variable | Description |
|---|---|
| `total_ride_count` | Total number of rides the driver gave |
| `total_distance` | Total miles driven on rides |
| `total_duration` | Total minutes spent on rides |
| `lifetime_value` | **Driver LTV:** sum of `ride_total_cost` across all of the driver's rides |
| `perday_ridecount_mean` | Average rides per day across every day in the dataset |
| `accept_response_time_mean` | Average `ride_accept_response_time` |
| `arrival_time_mean` | Average `ride_arrival_time` |
| `wait_time_mean` | Average `ride_wait_time` |
| `long_ride`, `med_ride`, `short_ride` | Number of rides the driver gave in each distance category |
| `morning_ride`, `evening_ride`, `night_ride` | Number of rides the driver accepted in each time-of-day category |
| `driver_duration` | Days from the driver's onboarding date to their most recent accepted ride |
| `rides_per_day` | `total_ride_count` divided by `driver_duration` |
| `cluster_st2` | K-means segment: 0 = fair, 1 = low value, 2 = excellent, 3 = good |

## Project Structure

```
.
├── Data/
│   ├── Raw-Data/          # original StrataScratch CSVs
│   └── Cleaned-Data/      # outputs of the notebook, used for the Tableau dashboard
├── lyft-ltv.ipynb         # full analysis
├── requirements.txt
└── README.md
```

## How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/tresgus/mdl-lyft--ltv.git
   cd mdl-lyft--ltv
   ```
2. Install the dependencies. The notebook was built with Python 3.13 in Anaconda; seaborn and yellowbrick are the only packages Anaconda doesn't include by default.
   ```bash
   pip install -r requirements.txt
   ```
   Or, from inside the notebook: `%pip install seaborn yellowbrick`
3. Open `lyft-ltv.ipynb` in Jupyter or VS Code and run all cells.

## Tools

Python (pandas, NumPy, Matplotlib, seaborn, scikit-learn, yellowbrick), Jupyter, and Tableau Public.

## Limitations

- The data covers one city over about three months, so driver lifetimes are cut off at the end of the data.
- Ride prices come from an assumed rate card, not actual fares.
- The k-means clusters were built on unscaled features, so the columns with the largest values have the most influence on the segments.
