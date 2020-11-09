# MetaCSV : describing CSV files, typing CSV data

Copyright (C) 2020 J. Férard <https://github.com/jferard>

License: GPLv3

**Work in progress**

# Overview
MetaCSV is an open specification for a CSV file description. This description is written in a small auxiliary CSV file that may be stored along wih the CSV file itself. This auxilary file should provide the informations necessary to read and type the content of the CSV file.

MetaCSV aims to be simple and extensible.

MetaCSV interpreters will be able to load CSV files into a typed table format, such as SQL or ODS.

## Example
Here's the content of the circular [`meta_csv.mcsv`](meta_csv.mcsv) file, with some comments:

domain | key            | value
-------|----------------|-------
file   | encoding       | UTF-8
file   | lineterminator | \r\n
csv    | delimiter      | ,
csv    | doublequote    | true
csv    | quotechar      | "
data   | col/0/type     | text
data   | col/1/type     | text
data   | col/2/type     | any

The first column, `domain`, should to be one of the following: `file`, `csv` or `data`. The second column is a `key` which might have multiple parts separated by slashes (`/`). The third column is a `value` which might also have several parts separated by slashes (`/`).

The `file` and `csv` domains are pretty obvious: they describe the format of the csv file. In this case, this is a RFC4180 csv file, encoded with the UTF-8 charset.

The`data` domain deals with the types of the columns.

Note that some `domain`-`key` pairs have default values. By default:
* the csv file is a RFC4180 csv file;
* the encoding is UTF-8;
* the column data type is `text`

Hence the previous example could be written:

domain | key            | value
-------|----------------|-------
data   | col/2/type     | any

## The `data` domain

Whereas the `file` and `csv` domains are pretty obvious, the `data` domain is a (little bit) more complex. The `key`s are always `col/<n>/type`, but the `value`s format depend on the datatype.

Here are some examples of multipart values (note that slashes may be escaped if necessary):

domain | key            | value
-------|----------------|------------------
...    | ...            | ...
data   | col/0/type     | date/YYYY-MM-DD
data   | col/1/type     | date/DD\/MM\/YYYY
data   | col/2/type     | float/,/.
data   | col/3/type     | boolean/vrai/faux
data   | col/4/type     | currency/pre/$/,/.
data   | col/5/type     | currency/post/€/,/.

## List of software using MetaCSV
Give me a break! I'm working on it...

## Difference with CSVSchema
CSVSchema was designed by The National Archives (United Kingdom). It is:

> a Grammar which describes a language for expressing rules to validate a CSV
> [The National Archives - Digital Preservation](http://digital-preservation.github.io/csv-schema/)

 Here's a synoptic presentation of the differences:

|               | CSVSchema           | MetaCSV
|----------------|---------------------|---------------------
|goal            | validate a CSV file | interpret a CSV file
|format          | specific language   | CSV file

# Full specification (DRAFT 0)
## Domain

The `domain` column can be either `file`, `csv` or `data`

## The `file` domain
The `file` domain describes the encoding and line separator of the text file.

Accepted keys are `encoding` and `lineterminator`.

### The `file`-`encoding` value
The value is an encoding as in https://www.iana.org/assignments/character-sets/character-sets.txt.

Default value is `UTF-8`.

### The `file`-`lineterminator` value
The value is one of: `\n`, `\r\n`, `n\r`, `\r` or any other byte sequence that is the line terminator of the CSV file.

Default value is `\r\n`.

## The `csv` domain
The keys are based on the [Python `csv` module](https://docs.python.org/3/library/csv.html#dialects-and-formatting-parameters):

Accepted keys are: `delimiter`, `doublequote`, `escapechar`, `quotechar`, `quoting`, `skipinitialspace`.

### The `csv-delimiter` value
The delimiter.

Default value is `,`.

### The `csv-doublequote` value
If true, double `quotechar` to escape `quotechar`. If false, use `escapechar` to escape `quotechar`.

Default value is `true`.

### The `csv-escapechar` value
If `doubleqote` is false, the escape char.

No default value.

### The `csv-quotechar` value
The quote char.

Default value is `"`.

### The `csv-skipinitialspace` value
If true, skip the space following the delimiter.

Default value is `false`.

## The `data` domain
The `data` domain describe the data contained in the CSV file, especially the type of the columns.

Each key has the format `col/<n>/type`, where `<n>` is the number of the column, starting at 0.

### The `data`-`col/<n>/type` value.
The value decribe the type of a column.

The format of the value is always: `type/parameter 1/parameter 2/.../parameter n`.

Further details:
* there may be zero parameters
* if a parameter is empty or null, it is blank (two slashes are consecutive).
* if the last parameters are empty or null, they may be ommited.

The data types are based on [ODF value types](http://docs.oasis-open.org/office/v1.2/os/OpenDocument-v1.2-os-part1.html#__RefHeading__1417680_253892949):

type        | description
------------|------------------------------------
`boolean`   | a true or false value
`currency`  | a currency value
`date`      | a date value
`datetime`  | a datetime value
`float`     | a float value
`integer`   | an integer value
`percentage`| a percentage value
`text`      | a text value
`any`       | a column having multiple value types

### The `boolean` value type
The `boolean` value type has the format:

    boolean/<true word>/<false word>

where `<true word>` is the lower case word for true, and `<false word>` is the lower case word for false.

Examples: `boolean/1/0`, `boolean/x` (false is empty), `boolean/t/f`.

### The `currency` value type
The `currency` value type has the format:

    currency/<pre|post>/<currency symbol>/<integer value|float value>

where:
* `<pre|post>` is either `pre` or `post` to indicate if the currency symbol is before or after the amount.
* `<currency symbol>` is the currency symbol (a sign, a code, a name)
* `<integer value|float value>` is either an integer value or a float value description.

Examples: `currency/post/€/float/ /,`

### The `date` value type
The `date` value type has the format:

    date/<date format>/<locale>

where:
* `<date format>` is the format of the date as in https://www.postgresql.org/docs/current/functions-formatting.html.
* `<locale>` is the name of the locale with the format `<language>_<country>`.

The locale may be omitted, but is useful for localized day names and month names.

Examples: `date/YYYY-MM-DD`

### The `datetime` value type
The `datetime` value type has the format:

    date/<datetime format>/<locale>

See the `date` value type for details.

### The `float` value type
The `float` value type has the format:

    float/<thousands separator>/<decimal separator>

where:
* `<thousands separator>`: the separator to group digits by thousands.
* `<decimal separator>`: the separators between the integer part and the fractional part of the number.

Examples: `float//.`, `float/,/.`

### The `integer` value type
The `integer` value type has the format:

    integer/<thousands separator>

See the `float` value type for details.

Examples: `integer`, `integer/ `

### The `percentage` value type
The `integer` value type has the format:

    percentage/<pre|post>/<percentage symbol>/<float value>

See the `currency` value type for details.

Examples: `percentage/post/%/float//.`

### The `text` value type
The `text` value type has the format:

    text

Example: `text`

### The `any` value type
The `any` value type has the format:

    any/<free data>

This tells the interpreter that it has to interpret the column values, but does not provide any further information unless `<free data>` is provided.

Examples: `any`
