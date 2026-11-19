# Data Cleaning

## Steps
Measure: how many rows, how much revenue is affected.
Look: print a few of the affected rows. Is it an error, or is it real?
Decide: fix, drop, flag or keep, and why.
Verify: rerun the measure after the fix.

Keep the raw DataFrames untouched and build orders_clean, order_lines_clean, products_clean and returns_clean from them, so you can always compare before and after.

## Questions
1. Are there duplicate orders?

    * Order rows: 9810
    * Unique order IDs: 9780
    * Rows with missing order IDs: 0
    * Rows with duplicate ids but distinct other values: 0
    Measure: 30 rows containing duplicate order information
    
    Look: orders with the same id were exact duplicates, not an error when assigning an id to an order
    
    Decide: Dropped duplicate rows so order_total was not counted multiple times
    
    Verify: checking orders_clean table shows no more duplicate ids and comparing dataframe lengths show that 30 rows were dropped from orders_clean compared to the original orders table

2. What does status really contain?

    * Status contains 5 different options (delivered, returned, cancelled, shipped, processing)
    * Original orders table had typos and different capitalization in status column. orders_clean
    has normalized status column
    * Delivered status means order is complete and payment received.  Could have partial return
    * Returned status means full order was returned
    * Added column to orders_clean dataframe, 'refund_amount', that uses the data from the returns table to show how much of each order was refunded

    Measure: 741 rows in which status is not in ['delivered', 'returned', 'cancelled', 'shipped', 'processing']
    
    Look: most of the errors are either extra whitespace or different capitalization, one typo type for cancelled
    
    Decide: trimmed and converted all statuses to lowercase, replaced typos
    
    Verify: count of all rows after cleaning matches row count before (9780) and only five distinct statuses remain

3. What do the missing totals mean?

    * There are missing order_total values in the orders table. One option is to fill those values with 0, fill them with an average amount from the orders table, leaving them as NaN, or using the order_lines table to fill the missing values
    
    Measure: The sum of order_total in delivered orders before filling the missing values was $1928732.10. The sum after filling the values using the order_lines data was $1943996.79. The difference is $15,264.69
    
    Look: the missing values were all placed during black friday week, which is often a time when many purchases are made
    
    Decide: of the options listed above I decided to fill the missing values by calculating the values by looking in the order_lines table
    
    Verify: checked value for order_ids in order_lines matched up with value filled in orders_clean

4. Do the two revenue sources agree?

    Measure: Comparing the order_total rows in orders_clean vs calculating the order total using the lines from order_lines showed that 1466 out of the 9780 orders had different values.
    
    Look: By isolating the rows that had different revenue totals I found that all those orders had a non-null value for promo_code which would explain why the order_total was different between the two data sources.
    
    Decide: For calculating the revenue I decided to use the the amount found in orders_cleaned because they are the actual amount of money earned for each transaction. One issue that may affect the overal revenue data is that there were 23 rows in orders that had a promo code but were also missing the order_total amount. I filled the orders df with data calculated from the lines in order lines but the actual revenue might be different for those lines with promo codes. There is an option to apply the promo code discount to each of those orders but I will not in this case because other factors could affect the order_total such as sales tax that might result in an incorrect value.

    Verify: Checked that all rows with different revenue totals were also the rows with promo codes, explaining the discrepancy in the numbers.

5. What is wrong with the products dataframe?

    Measure: There were 29 rows in products that were missing a colour. There are 120 unique styles in products

    Look: For the repeating styles I took a look at all rows that shared a specific style ("Ysee Bodysuit"). Each of the rows that shared this style were similar but not duplicates.  There were multiple rows that shared the same style and colour but the sizes were different.

    Decide: In response to the missing values in the colours column I decided to fill the missing values with 'unknown' so that the row would remain in place and the colour field in those rows would be filled with something. For the styles I kept everyting as is because there were no duplicate rows.

    Verify: Checking the value counts of colours we see that 29 rows are now 'unknown' which matches the number of rows with missing values. The styles column stayed the same.

6. Are the very large orders errors?

    Measure: The average order total was $238.37. Calculating the upper fence for outliers (Q3 + 1.5 * IQR) yielded $631.35. There were 302 orders with totals over the upper fence.

    Look: The highest order total was $17969.70 and this order consisted of 103 units in the order. 253 of the 302 orders had a status of delivered.

    Decide: While these high value orders may affect the mean order_total, they should not be removed just because they are outliers. To get a more accurate mean we can remove the outliers when calculating which gives us a mean of $211.48.