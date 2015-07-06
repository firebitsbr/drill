---
title: "Text Files: CSV, TSV, PSV"
parent: "Data Sources and File Formats"
---

Best practices for reading text files are:

* Select data from particular columns  
* CAST data  
* Use a distributed file system  

### Select Data from Particular Columns

Converting text files to another format, such as Parquet, using the CTAS command and a SELECT * statement is not recommended. Instead, select data from particular columns using the [COLUMN[n] syntax]({{site.baseurl}}/docs/querying-plain-text-files), and then assign meaningful column
names using aliases. For example:

    CREATE TABLE parquet_users AS SELECT CAST(COLUMNS[0] AS INT) AS user_id,
    COLUMNS[1] AS username, CAST(COLUMNS[2] AS TIMESTAMP) AS registration_date
    FROM `users.csv1`;

You need to select particular columns instead of using SELECT * for performance reasons. Drill reads CSV, TSV, and PSV files into a list of
VARCHARS, rather than individual columns. While parquet supports and Drill reads lists, as of this release of Drill, the read path for complex data is not optimized. 

### Cast data

You can also improve performance by casting the VARCHAR data to INT, FLOAT, DATETIME, and so on when you read the data from a text file. Drill performs better reading fixed-width than reading VARCHAR data. 

### Use a Distributed File System
Using a distributed file system, such as HDFS, instead of a local file system to query the files also improves performance because currently Drill does not split files on block splits.

## Configuring Drill to Read Text Files
In the storage plugin configuration, you can set the following attributes that affect how Drill reads CSV, TSV, PSV (comma-, tab-, pipe-separated) files.  ["List of Attributes and Definitions"]({{site.baseurl}}/docs/plugin-configuration-basics/#list-of-attributes-and-definitions):

* String lineDelimiter = "\n";  
  One or more characters used to denote a new record. Allows reading files with windows line endings.  
* char fieldDelimiter = ',';  
  A single character used to separate each value.  
* char quote = '"';  
  A single character used to start/end a quoted value.  
* char escape = '"';  
  A single character used to escape a quote inside of a value.  
* char comment = '#';  
  A single character used to denote a comment line.  
* boolean skipFirstLine = false;  
  Set to true to avoid reading headers as data. 

For more information about storage plugin configuration, see ["List of Attributes and Definitions"]({{site.baseurl}}/docs/plugin-configuration-basics/#list-of-attributes-and-definitions).

You can deal with a mix of text files with and without headers either by creating two separate format plugins or by creating two format plugins within the same storage plugin. The former approach is typically easier than the latter.

### Creating Two Separate Format Plugins
Format plugins are associated with a particular storage plugin. Storage plugins define a root directory that Drill targets when using the storage plugin. You can define separate storage plugins for different root directories, and define each of the format attributes to match the files stored below that directory. All files can use the .csv extension.

For example:

Storage Plugin A

    "csv": {
      "type": "text",
      "extensions": [
        "csv"
      ],
      "delimiter": ","
    },
    . . .


Storage Plugin B

    "csv": {
      "type": "text",
      "extensions": [
        "csv"
      ],
      "comment": "&",
      "skipFirstLine": true,
      "delimiter": ","
    },

### Creating Two Format Plugins within the Same Storage Plugin
Give a different extension to files with a header and to files without a header, and use a storage plugin that looks something like the following example. This method requires renaming some files to use the csv2 extension.

For example:

    "csv": {
      "type": "text",
      "extensions": [
        "csv"
      ],
      "delimiter": ","
    },
    "csv_with_header": {
      "type": "text",
      "extensions": [
        "csv2"
      ],
      "comment": "&",
      "skipFirstLine": true,
      "delimiter": ","
    },
