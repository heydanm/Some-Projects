## Introduction

The occurrence of ID duplicates in databases is a common problem that can affect data integrity and the accuracy of analyses made from that data. Duplicates can arise due to various reasons, such as human errors, issues in the unique ID generation process, or data integration from different sources. This can lead to complications in data analysis, such as incorrect performance indicator calculations or difficulties in tracking the history of a specific proposal.

In this post, I present a DAX solution to address these duplicates, should they occur.

## Business Problem

In the case at hand, we want to calculate the total value of won proposals in a specific region, considering only proposals with a closing date from 2023 onwards. However, there are duplicate ID_PROPOSAL values due to the duplication of products included in the same proposal.

##### We have this problematic table:

| ID_PROPOSAL | REGION | STATUS                   | CLOSING_DATE | PROPOSAL_TOTAL_VALUE |  PRODUCT  |
|-------------|--------|--------------------------|--------------|----------------------|-----------|
| ID_1        | A      | Won proposal             | 2023-02-01   | 100                  | Product A |
| ID_1        | A      | Won proposal             | 2023-02-01   | 100                  | Product B |
| ID_2        | B      | Won proposal             | 2023-02-03   | 200                  | Product A |
| ID_3        | C      | Won proposal             | 2023-01-04   | 300                  | Product C |
| ID_3        | C      | Won proposal             | 2023-01-04   | 300                  | Product C |
| ID_4        | A      | Unsuccessful proposal    | 2023-02-01   | 100                  | Product A |
| ID_5        | A      | Unsuccessful proposal    | 2023-02-01   | 100                  | Product B |
| ID_6        | B      | Won proposal             | 2022-02-03   | 200                  | Product A |
| ID_7        | C      | Won proposal             | 2022-01-04   | 300                  | Product C |
| ID_8        | C      | Unsuccessful proposal    | 2022-01-04   | 300                  | Product C |

## Solution: Using DAX Measure

To correctly calculate the total value of distinct won proposals without the duplication caused by duplicate products for the year 2023, we can use the following DAX measure:

### DAX Formula

```DAX
DISTINCT_VALUES_FROM_DISTINCT_WON_PROPOSAL =
CALCULATE(
    SUMX(
        DISTINCT('Table_1'[ID_PROPOSAL]),
        FIRSTNONBLANK('Table_1'[TOTAL_VALUE], 0)
    ),
    FILTER('Table_1', 'Table_1'[STATUS] = "Won proposal"),
    FILTER('Table_1', 'Table_1'[CLOSING_DATE].[Year] >= 2023)
)
```

### Measure Description
# The measure uses the following variables:

- 'Table_1': the name of the table containing the data to be analyzed.
- 'ID_PROPOSAL': the name of the column containing the unique identifier for each proposal.
- 'TOTAL_VALUE': the name of the column containing the total value of each proposal.
- 'STATUS': the name of the column indicating the status of each proposal (won, lost, in progress, etc.).
- 'CLOSING_DATE': the name of the column indicating the closing date of each proposal.
- 'Year': a function that extracts the year from the closing date of the proposal.

### Solution and Results
The objectives of the measure are as follows:

- Sum the total value of all won proposals in a specific region.
- Consider only the proposals with a closing date from 2023 onwards.
- Remove duplicates when counting the total value, considering only the first non-null value encountered.
- Apply filters to restrict the analysis to won proposals in region A.
- By implementing this measure in your Power BI, you can calculate the sum of the total values of all won proposals ('Won proposal') in a specific region and product, considering only the proposals that were closed after the year 2023.

##### The 'CALCULATE' function is used to modify the filter context of the 'Table_1' table and filter the data based on the specified conditions later in the measure.

##### The 'SUMX' function is used to sum the result of an expression for each row of a table. In this case, the expression is being applied to each distinct ID_PROPOSAL in the table, which is obtained using the DISTINCT function.

##### The 'FIRSTNONBLANK' function is being used to get the first non-empty value from 'Table_1'[TOTAL_VALUE]. If the first value encountered is empty, the value 0 is returned. In other words, this function is used to replace null values in the TOTAL_VALUE column with zero.

The measure "DISTINCT_VALUES_FROM_DISTINCT_WON_PROPOSAL" is a powerful and efficient solution for calculating the total value of won proposals, considering specific criteria and handling duplicates appropriately. By implementing this measure in Power BI, precise and reliable results can be obtained when summing the total values of won proposals in a specific region and product, taking into account the defined restrictions.

Through the use of the 'CALCULATE', 'SUMX', and 'FIRSTNONBLANK' functions, the filter context can be modified, the values can be summed for each distinct ID_PROPOSAL, and null values can be replaced with zero, ensuring the integrity of the calculations. With this measure, data analysts can gain a more comprehensive and accurate view of the won proposals, enabling more informed and effective decision-making.
