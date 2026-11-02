# Data Validation

1. Upon data collection, the research team should have received:
    1. A reference code from each participant (e.g., extracted from survey responses)
    2. A verification code from each participant
    3. An email sent by Tesco titled "Customer data request" that contains the link to download the data from each participant (it is likely that each request will result in multiple emails)

2. The research team should check that:
    1. The verification codes are in the correct format, which consists of 6-digit combinations of numbers and letters (they do _not_ have to have numbers and can have only letters)
    2. An email from Tesco was indeed received

# Data Preparation

1. Download the data. By default, the name of each .JSON file should be `[reference_code]-transactions.json`.

2. Below, the command line tool `jq` will be used to manipulate .JSON files. There are, of course, other tools to perform these operations, but that is beyond the scope of this documentation. See the official [download page](https://jqlang.github.io/jq/download/) to install `jq`.

3. Merge all .JSON files into one file. Specifically, the output will be a Newline Delimited .JSON file, that is, the data of each participant will be written as a new line.

```
jq -c . ./path/*.json | sed 's/\\"//g'| sed 's/\\r//g' | sed 's/\\\\/ /g' | sed 's/\\t//g' | sed 's/\\n//g' > ./merged.json
```

The first part of the one-liner (i.e., `jq -c ...`) will print each .JSON file onto a new line in the output file, while the next few parts (i.e., `sed ...`) are ad-hoc solutions to handle some characters that can cause issues when importing the data into another ecosystem. Here the code will delete all instances of `\"`, `\r`, and other patterns because they cause errors in PostgreSQL.

# Data Import

## PostgreSQL

This section assumes a database has been set up to store the incoming data. Connect to the database via `psql`, `pgcli`, or other interfaces.

Below is an example of a SQL script that will reshape the data into a clean and two-dimensional format. For the sake of clarity, temporary tables will be used throughout, which need to be discarded every time the script is run in the same session (`DISCARD TEMP;`).

```sql
CREATE TEMP TABLE temp_data_import (col_jsonb jsonb);

\copy temp_data_import (col_jsonb) FROM './merged.json';

CREATE TEMP TABLE temp_data_by_customer AS
(
    SELECT
        col_jsonb -> 'requestId' AS reference_code,
        col_jsonb -> 'purchases' AS purchases
    FROM temp_data_import
);

CREATE TEMP TABLE temp_data_by_basket AS
(
    SELECT
        reference_code AS reference_code,
        jsonb_array_elements(purchases) AS basket
    FROM temp_data_by_customer
);

CREATE TEMP TABLE temp_data_by_item AS
(
    SELECT
        reference_code AS reference_code,
        basket -> 'timestamp' AS time_stamp,
        jsonb_array_elements(basket -> 'items') AS item
    FROM temp_data_by_basket
);

CREATE TEMP TABLE temp_data_by_basket_clean AS
(
    SELECT
        reference_code AS reference_code,
        basket -> 'timestamp' AS time_stamp,
        basket -> 'type' AS channel,
        basket -> 'numberOfItems' AS number_of_items,
        basket -> 'basketValueNet' AS basket_value_net,
        basket -> 'basketValueGross' AS basket_value_gross,
        basket -> 'overallBasketSavings' AS basket_savings_overall,
        basket -> 'payment' -> 0 -> 'type' AS payment_type,
        basket -> 'payment' -> 0 -> 'amount' AS payment_amount,
        basket -> 'payment' -> 0 -> 'category' AS payment_category
    FROM temp_data_by_basket
);

CREATE TEMP TABLE temp_data_by_item_clean AS
(
    SELECT
        reference_code AS reference_code,
        time_stamp AS time_stamp,
        item -> 'name' AS name,
        item -> 'quantity' AS quantity,
        item -> 'price' AS price,
        item -> 'volume' AS volume
    FROM temp_data_by_item
);
```

