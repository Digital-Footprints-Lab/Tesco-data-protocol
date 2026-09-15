# Data Dictionary

This section assumes the data files have been imported into a PostgreSQL database following the protocol described in this repository.

## Data by Basket

| Column                   | Example                                |
| ----                     | ----                                   |
| `reference_code`         | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `time_stamp`             | `2025-01-10 20:15:00.333`              |
| `channel`                | `GHS` or `IN_STORE`                    |
| `number_of_items`        | 7                                      |
| `basket_value_net`       | 20.6                                   |
| `basket_value_gross`     | 20.6                                   |
| `basket_savings_overall` | 4.55                                   |
| `payment_type`           | `CARD`, `CASH`, etc                    |
| `payment_amount`         | 20.6                                   |
| `payment_category`       | `Visa`, `MasterCard`, etc              |

## Data by Item

| Column           | Example                                |
| ----             | ----                                   |
| `reference_code` | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `time_stamp`     | `2025-01-10 20:15:00.333`              |
| `name`           |                                        |
| `quantity`       | 1                                      |
| `price`          | 2.75                                   |
| `volume`         | 1.0                                    |

