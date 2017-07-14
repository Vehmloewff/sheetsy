- returns columns as an array as well as a map

# Use cases

- https only in the browser
- node and browser

# Things to know about your spreadsheet

The column names are generated by Google Sheets, not by this library.

Whether the first row is a frozen header or not, Google Sheets will treat it as if it is the header row of your sheet, and will use the values in those columns to generate the names of all of the column values in all the rows following.
