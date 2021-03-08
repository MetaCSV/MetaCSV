Columns are:

Column name         | Column type
---                 | ---
a boolean           | BOOLEAN
a currency          | CURRENCY_INTEGER
another currency    | CURRENCY_DECIMAL
a date              | DATE
a datetime          | DATETIME
a decimal           | DECIMAL
a float             | FLOAT
an integer          | INTEGER
a percentage        | PERCENTAGE_FLOAT
another percentage  | PERCENTAGE_DECIMAL
a text              | TEXT
an URL              | OBJECT

Values are:

column             | row 1               | row 2           | row 3
---                | ---                 | ---             | ---
a boolean          | True                | *&lt;NULL&gt;*  | False
a currency         | 5 (int)             | 17 (int)        | ReadError(7, "currency/€/post/integer")*
another currency   | 10.3 (decimal)      | 10.8 (decimal)  | 17.8 (decimal)
a date             | 1975-04-20          | 2017-11-21      | 2003-06-03
a datetime         | 1975-04-20 13:45:00 | *&lt;NULL&gt;*  | *&lt;NULL&gt;*
a decimal          | 10.3 (decimal)      | 1.03 (decimal)  | 32.5 (decimal)
a float            | 1232.0 (float)      | -2.50 (float)   | -2456.50 (float)
an integer         | 56 (int)            | 76 (int)        | 1786 (int)
a percentage       | 0.56 (float)        | 0.58 (float)    | 0.85 (float)
another percentage | 0.782 (decimal)     | 0.182 (decimal) | 0.128 (decimal)
a text             | meta csv &vert; (text)       | "java-mcsv" (text) | py-mcsv (text)
an URL             | https://github.com/jferard** |*&lt;NULL&gt;*      | *&lt;NULL&gt;*

&ast; Depending on the error handler. `ReadError` if `on_error = ERROR_TYPE`, may also be *&lt;NULL&gt;* (`on_error = NULL`), "7" (`on_error = TEXT`) or throw an exception (on_error = `EXCEPTION`).

&ast;&ast; An URL object if possible, a text object else.
