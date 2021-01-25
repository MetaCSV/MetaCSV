# MetaCSV : describing CSV files, typing CSV data

Copyright (C) 2020 J. Férard <https://github.com/jferard>

License: GPLv3

**Work in progress**

# Overview
MetaCSV is an open specification for a CSV file description. This description
is written in a small auxiliary CSV file that may be stored along with the CSV
file itself. This auxiliary file should provide the information necessary to
read and type the content of the CSV file. The standard extension is ".mcsv".

MetaCSV aims to be simple and extensible.

MetaCSV interpreters will be able to load CSV files into a typed table format,
such as SQL or ODS.

**The format is free to use (using it in your own software will not make it
GPL) but if you derive a format from MetaCSV, please comply with the license.**

## Example
Here's the content of the self-referential [`meta_csv.mcsv`](meta_csv.mcsv)
file, with some comments:

domain | key             | value
-------|-----------------|-------
file   | encoding        | UTF-8
file   | line_terminator | \r\n
csv    | delimiter       | ,
csv    | double_quote    | true
csv    | quote_char      | "
data   | col/0/type      | text
data   | col/1/type      | text
data   | col/2/type      | object/base64

The first column, `domain`, should to be one of the following: `file`, `csv`
or `data`. The second column is a `key` which might have multiple parts
separated by slashes (`/`). The third column is a `value` which might also have
several parts separated by slashes (`/`).

The `file` and `csv` domains are pretty obvious: they describe the format of
the csv file. In this case, this is a RFC4180 csv file, encoded with the UTF-8
charset.

The`data` domain deals with the types of the columns.

Note that some `domain`-`key` pairs have Canonical values. By default:
* the csv file is a RFC4180 csv file;
* the encoding is UTF-8;
* the column data type is `text`

Hence the previous example could be written:

domain | key            | value
-------|----------------|-------
data   | col/2/type     | object/base64

## The `data` domain

Whereas the `file` and `csv` domains are pretty obvious, the `data` domain is a
(little bit) more complex. The `key`s are always `col/<n>/type`, but the
`value`s format depend on the datatype.

Here are some examples of multipart values (note that slashes may be escaped
if necessary):

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
My work:
* [ColumnDet](https://github.com/jferard/ColumnDet) can export its result to a
MetaCSV file.
* [java-mcsv](https://github.com/jferard/java-mcsv) is the reference
implementation for *MetaCSV libraries* (see definitions below): able to parse
MetaCSV files, read/type CSV data on the fly in Java, and to read and write
`ResultSet`s.
* [py-mcsv](https://github.com/jferard/py-mcsv) aims to be a *MetaCSV library*
(see definitions below): able to parse MetaCSV files and read/type CSV data
on the fly.

My work - TODO:
* [CSVInspector](https://github.com/jferard/CSVInspector) will use MetaCSV as a
 communication format.
* [FastODS](https://github.com/jferard/py-mcsv) 0.7.4 will be able to generate
 a Calc document from a  CSV file having MetaCSV data.

External work:
* help yourself! The MetaCSV format is free to use...


## Difference with CSVSchema
CSVSchema was designed by The National Archives (United Kingdom). It is:

> a Grammar which describes a language for expressing rules to validate a CSV
> [The National Archives - Digital Preservation](
http://digital-preservation.github.io/csv-schema/)

 Here's a synoptic presentation of the differences:

|               | CSVSchema           | MetaCSV
|---------------|---------------------|---------------------
|goal           | validate a CSV file | interpret a CSV file
|format         | specific language   | CSV file

The choice of the CSV format to type the CSV file guarantees that MetaCSV
libraries will be easy to implement and only depend on a CSV reader/writer
library or module only.

# Full specification (DRAFT 0)
## 0. Definitions
### 0.1 MetaCSV
A *MetaCSV text* is an UTF-8 text (string, bytes, file...) compliant with this
specification.

A *MetaCSV structure* is a programming language structure or class that
represents a MetaCSV format.

A *MetaCSV format* is the underlying format represented by a MetaCSV text or
structure.

A *canonical MetaCSV format* is a specific *MetaCSV format* (see below).

A *MetaCSV parser* must translate a *MetaCSV text* to a *MetaCSV structure*
instance.

A *MetaCSV renderer* must translate a *MetaCSV structure* instance to a
*MetaCSV text*.

### 0.2 CSV and MetaCSV
A *CSV text* is an encoded text (string, bytes, file...) that represents a
table. [The RFC4180 specification](https://tools.ietf.org/html/rfc4180)
describes a standardized CSV format.

A *MetaCSV format* (text or structure instance) is *compatible* with a *CSV
text* if the *CSV text* records can be read and typed according to the *MetaCSV
format*. Several *MetaCSV formats* may be compatible with the same *CSV text*.

A *MetaCSV reader* must read and type a *CSV text* along with any *compatible
MetaCSV format*.

A *MetaCSV writer* must write typed records to a *CSV text* and produce a
*compatible MetaCSV format*. If the user does not require a *MetaCSV format*,
the produced *MetaCSV format* should be *canonical*.

### 0.3 MetaCSV library (or module)
A *MetaCSV library* (or module) must provide a *MetaCSV structure*, a *MetaCSV
parser* and a *MetaCSV renderer*.

It should provide a *MetaCSV reader* and a *MetaCSV writer*.

It should not depend on non standard libraries (or modules) other than a
library (or module) to read and write CSV files.

If some formats are handled by the standard libraries (or modules), the MetaCSV
library may provide a conversion to those formats.

## 1. Domain, key, value
The `domain` column can be either `file`, `csv` or `data`.

Each domain has its specific set of keys. A `key` may have multiple parts
separated by slashes (`/`). A key part may be empty.

Each key accept some values. A `value` may also have several parts separated
by slashes (`/`). A value part may be empty.

## 2. The `file` domain
The `file` domain describes the encoding and line separator of the text file.

Accepted keys are `encoding` and `line_terminator`.

### 2.1 The `file`-`encoding` value
The value is an encoding as in
https://www.iana.org/assignments/character-sets/character-sets.txt.

Canonical value is `UTF-8`.

*Note: A parser may accept the `UTF-8-SIG` encoding as:

domain | key            | value
-------|----------------|------------------
file   | encoding       | UTF-8
file   | bom            | true

But a renderer must not emit the `UTF-8-SIG` encoding.*

### 2.1 The `file`-`bom` value
The value is a boolean. If the encoding is `UTF-8` and the `file - bom`
value is true, the file starts with the infamous Microsoft's BOM.

Canonical value is `false`.


### 2.3 The `file`-`line_terminator` value
The value is one of: `\n`, `\r\n`, `n\r`, `\r` or any other byte sequence that
is the line terminator of the CSV file.

Canonical value is `\r\n`.

## 3. The `csv` domain
The keys are based on the [Python `csv` module](    https://docs.python.org/3/library/csv.html#dialects-and-formatting-parameters):

Accepted keys are: `delimiter`, `double_quote`, `escape_char`, `quote_char`,
`quoting`, `skip_initial_space`.

### 3.1 The `csv-delimiter` value
The delimiter.

Canonical value is `,`.

### 3.2 The `csv-double_quote` value
If true, double `quote_char` to escape `quote_char`. If false, use
`escape_char` to escape `quote_char`.

Canonical value is `true`.

### 3.3 The `csv-escape_char` value
If `double_quote` is false, the escape char.

No Canonical value.

### 3.4 The `csv-quote_char` value
The quote char.

Canonical value is `"`.

### 3.5 The `csv-skip_initial_space` value
If true, skip the space following the delimiter.

Canonical value is `false`.

## 4. The `data` domain
The `data` domain describe the data contained in the CSV file, especially the
type of the columns.

Each key has the format `col/<n>/type`, where `<n>` is the number of the
column, starting at 0.

### 4.1 the `data - null_value` value
The value to be mapped to a string representing the absence of a value, as NULL
in SQL.

Example: `<NULL>`

Canonical value is an empty string.

This value may be overridden by the  value of `data - col/n/null_value`.

### 4.2 The `data`-`col/<n>/type` value
The value describe the type of a column.

The format of the value is always: `type/parameter 1/parameter 2/.../parameter
n`.

Further details:
* there may be zero parameters
* if a parameter is empty or null, it is blank (two slashes are consecutive).
* if the last parameters are empty or null, they may be omitted.

The data types are based on [ODF value types](
http://docs.oasis-open.org/office/v1.2/os/OpenDocument-v1.2-os-part1.html#__RefHeading__1417680_253892949):

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
`object`    | a value of another type

The reader needs information about the specific format of the values of a data
type. For instance, a true boolean may be `1`, `true`, `vrai`... The reader
does not have to guess what is "true" and what is "false": the MetaCSV format
provides parameters for the data types and these parameters make the data
unambiguous.

The writer needs the same information to write the data.

### The `boolean` value type
The `boolean` value type has the format:

    boolean/<true word>/<false word>

where `<true word>` is the lower case word for true, and `<false word>` is the
lower case word for false.

Examples: `boolean/1/0`, `boolean/x` (false is empty), `boolean/t/f`.

Canonical form: `boolean/true/false`

### The `currency` value type
The `currency` value type has the format:

    currency/<pre|post>/<currency symbol>/<integer value|float value>

where:
* `<pre|post|>` is either `pre`, `post` or empty to indicate if the currency
symbol is before or after the amount, or is omitted.
* `<currency symbol>` is the currency symbol (a sign, a code, a name)
* `<integer value|float value>` is either an integer value or a float value description.

Examples: `currency/post/€/float/ /,`

Canonical form: `currency//<currency symbol>/float//.`

### The `date` value type
The `date` value type has the format:

    date/<date format>/<locale>

where:
* `<date format>` is the format of the date as in the [Unicode Locale Data Markup Language (LDML)](http://www.unicode.org/reports/tr35/tr35-dates.html#Date_Field_Symbol_Table).
* `<locale>` is the name of the locale with the format `<language>_<country>`.

The locale may be omitted, but is useful for localized day names and month names.

Examples: `date/yyyy-MM-dd`

Canonical form: `date/yyyy-MM-dd`

Note: the Java [`SimpleDateFormat`](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/text/SimpleDateFormat.html) covers a subset of the symbols of the Unicode LDML.

### The `datetime` value type
The `datetime` value type has the format:

    date/<datetime format>/<locale>

See the `date` value type for details.

Canonical forms: `date/yyyy-MM-dd'T'HH:mm:ss[.S+][X]`

### The `float` value type
The `float` value type has the format:

    float/<thousands separator>/<decimal separator>

where:
* `<thousands separator>`: the separator to group digits by thousands.
* `<decimal separator>`: the separators between the integer part and the fractional part of the number.

Examples: `float//.`, `float/,/.`

Canonical form: `float//.`

### The `integer` value type
The `integer` value type has the format:

    integer/<thousands separator>

See the `float` value type for details.

Examples: `integer`, `integer/ `

Canonical form: `integer`

### The `percentage` value type
The `integer` value type has the format:

    percentage/<pre|post>/<percentage symbol>/<float value>

See the `currency` value type for details.

Examples: `percentage/post/%/float//,`

Canonical form: `percentage/post/%/float//.`

### The `text` value type
The `text` value type has the format:

    text

Example: `text`

Canonical form: `text`

### The `object` value type
The `object` value type has the format:

    object/<parameters>

#### A catch-all type
Basically, the `object` type is a catch-all type for columns whose type is not one of the explicit MetaCSV types (`boolean`, `currency`, `date`, `datetime`, `float`, `integer`, `percentage`, `text`). The column data may have various types, or one type that is not defined by MetaCSV.

MetaCSV allows extra parameters to specify the actual type of the column, but those parameters are **not standardized**.

If there is no extra parameter:
* A *MetaCSV reader* must return this value as text and let the user handle the value.
* A *MetaCSV writer* must accept a textual representation of the value given by the user.

If there are some extra parameters:
* A *MetaCSV reader* may return this value as text and let the user handle the value, or may use the given parameters to return a typed value.
* A *MetaCSV writer* must accept a textual representation of the value or *MetaCSV writer* use the given parameters to create the textual representation of the value given by the user.

The behavior of *MetaCSV reader* s and *MetaCSV writer* s that belong to a same library must be consistent: an `object` typed value written by the *MetaCSV writer* and read by the *MetaCSV reader* will be the same as the initial value. **But there is no general guarantee that foreign *MetaCSV reader* s and *MetaCSV writer* s will be consistent**.

#### Examples
If the type of the column is simply:

    object

But the user *knows* that the column contains JSON data, the user will be able to parse it.

If the type of the column is:

    object/base64

The *MetaCSV reader* may decode the base64 value to return binary data. And the *MetaCSV writer* may encode binary data in base64 (and must accept a base64 encoded value).

No canonical form.

### 4.3 The `data`-`col/<n>/null_value` value
The value to be mapped to a string representing the absence of a value, as NULL
in SQL, for the column <n>.

Example: `<NULL>`

Canonical value is an empty string.

This value overrides the value of `data - null_value` for this column.

*Note: This is useful when columns have a different sources and different marks
to signal the absence of a value.
