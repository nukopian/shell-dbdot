% MF(1) 1.0.1 | MySQL Fetcher & Formatter
% Iain Campbell

## NAME
mf - MySQL Fetcher & Formatter

## SYNOPSIS
	mf [options] [--] [sql-statement]
	mf [options] [-H|--host host] [-D|--database name] [options] [--] [sql-statement]
	[DB_HOST=hostname] [DB_NAME=database] mf [options] [--] [sql-statement]

## DESCRIPTION
Using `ssh` to connect to the database host, execute a SQL statement on the remote MySQL database inside a `READ ONLY` session, in which we also `SET autocommit=0`. If access to critical data is not easily accessed by API or other tools then we can use this `mf` to quickly cover any functional gaps in order to help us automate tasks and present data in a number of useful formats suitble for further processing.

When no output format is selected, `mf` will defult to presenting data in the same tabular manner as the MySQL Monitor:

	+----------+----------+
	| column-1 | column-2 |
	+----------+----------+
	| value    | value    |
	+----------+----------+

### Options

`-H`, `--hostname` _host_

Use this option to specify the database host. The default is whatever `$DB_HOST` is set to.

`-D`, `--database` _name_

Use this option to specify the database name. The default is whatever `$DB_NAME` is set to, or (failing that) `vz`.

`-X`, `--xml`

Use this option to render output as XML.

`-J`, `--json`

Use this option to render output as JSON. The result set is retrieved as XML, transformed into JSON and then rendered using `jq`.

`-T`, `--json-tsv`

Use this option to render output as tab-separated values. The result set is retrieved as XML, transformed into JSON and the end result is what you might expect from appending a `@tsv` stage to the end of a `jq` pipeline.

`-C`, `--json-csv`

Use this option to render output as comma-separated values. The result set is retrieved as XML, transformed into JSON and the end result is what you might expect appending a `@csv` stage to the end of a `jq` pipeline.

`-t`, `--table`

Use this option to produce columnar output, with all columns being aligned.

Any `NULL` values will be converted to `-`, as will any empty columns. Binary data is rendered as unquoted `0x...` hex sequences.

`-c`, `--csv ["single" | "1" | "'" | "double" | "2" | '"']`

Use this option to produce CSV output when more control over the quoting regime is required.

It starts with the same columnar output that the `-t, --table` option produces, and reduces that down to comma-separted values. Any `NULL` values will be collapsed down to sequential commas (i.e. `,NULL,` becomes `,,`), and binary data is rendered as unquoted `0x...` hex sequences.

**Be careful**, when using the default, to separate the option from the SQL statement using `--` or you may see some strange errors.

`-b`, `--brackets ["round" | '()' | "square" | '[]' | "braced" | "braces" | '{}' ]`

This option is only useful when used in conjunction with `-C, --json-csv` and `-c, -csv` options.

Use this option when you wish to surround each row of the result set in brackets, which can be useful if a list of tuples is required. While the default is round brackets `()`, you may choose square brackets or braces.

**Be careful**, when using the default, to separate the option from the SQL statement using `--` or you may see some strange errors.

`--header`, `--no-header`

This option is only useful when presenting results using the default output format, or in conjunction with the `-T, --json-tsv`, `-C, --json-csv`, `-t, --table` and `-c, --csv` options.

Use it to include (or exclude) the column headings in the output.

`-i`, `--iso8601`

Make `YYYY-MM-DD HH:MM:SS` timestamps ISO-8601-compliant.

This is done by inserting `T` between the date and time (`YYYY-MM-DDTHH:MM:SS`), which can sometimes be helpful in disambiguating date and time separators from column separators in some output formats.

`-h`, `--help`

Display usage notes.

## PREREQUISITES
This utility requires access to both `yq` and `jq` tools if you intend to use the `-J, --json`, `-T, --json-tsv`, or `-C, --json-csv` options. On macOS, these can be installed using Homebrew:

	brew install yq jq

## EXAMPLES

### Example 1

	mf "select id, type, enabled from product where ram_mib < 16384 and family in ('gd-vps4') order by type, id"

	+--------------------------+------+---------+
	| id                       | type | enabled |
	+--------------------------+------+---------+
	| oh.hosting.c1.r1.d20.ct  | CT   |       1 |
	| oh.hosting.c1.r2.d40.ct  | CT   |       1 |
	| oh.hosting.c1.r4.d40.ct  | CT   |       1 |
	| oh.hosting.c2.r4.d100.ct | CT   |       1 |
	| oh.hosting.c2.r8.d100.ct | CT   |       1 |
	| oh.hosting.c3.r6.d150.ct | CT   |       1 |
	| oh.hosting.c4.r8.d200.ct | CT   |       1 |
	| oh.hosting.c1.r1.d20     | VM   |       1 |
	| oh.hosting.c1.r2.d40     | VM   |       1 |
	| oh.hosting.c1.r4.d40     | VM   |       1 |
	| oh.hosting.c2.r4.d100    | VM   |       1 |
	| oh.hosting.c2.r8.d100    | VM   |       1 |
	| oh.hosting.c3.r6.d150    | VM   |       1 |
	| oh.hosting.c4.r8.d200    | VM   |       1 |
	+--------------------------+------+---------+

### Example 2

	mf --table \
		"select id, type, enabled from product where ram_mib < 16384 and family in ('gd-vps4') order by type, id"

	id                        type  enabled
	oh.hosting.c1.r1.d20.ct   CT    1
	oh.hosting.c1.r2.d40.ct   CT    1
	oh.hosting.c1.r4.d40.ct   CT    1
	oh.hosting.c2.r4.d100.ct  CT    1
	oh.hosting.c2.r8.d100.ct  CT    1
	oh.hosting.c3.r6.d150.ct  CT    1
	oh.hosting.c4.r8.d200.ct  CT    1
	oh.hosting.c1.r1.d20      VM    1
	oh.hosting.c1.r2.d40      VM    1
	oh.hosting.c1.r4.d40      VM    1
	oh.hosting.c2.r4.d100     VM    1
	oh.hosting.c2.r8.d100     VM    1
	oh.hosting.c3.r6.d150     VM    1
	oh.hosting.c4.r8.d200     VM    1

### Example 3

	mf --csv --no-header \
		"select id, type, enabled from product where ram_mib < 16384 and family in ('gd-vps4') order by type, id"

	"oh.hosting.c1.r1.d20.ct","CT","1"
	"oh.hosting.c1.r2.d40.ct","CT","1"
	"oh.hosting.c1.r4.d40.ct","CT","1"
	"oh.hosting.c2.r4.d100.ct","CT","1"
	"oh.hosting.c2.r8.d100.ct","CT","1"
	"oh.hosting.c3.r6.d150.ct","CT","1"
	"oh.hosting.c4.r8.d200.ct","CT","1"
	"oh.hosting.c1.r1.d20","VM","1"
	"oh.hosting.c1.r2.d40","VM","1"
	"oh.hosting.c1.r4.d40","VM","1"
	"oh.hosting.c2.r4.d100","VM","1"
	"oh.hosting.c2.r8.d100","VM","1"
	"oh.hosting.c3.r6.d150","VM","1"
	"oh.hosting.c4.r8.d200","VM","1"

### Example 4

	mf --json-tsv --no-header \
		"select id, type, enabled from product where ram_mib < 16384 and family in ('gd-vps4') order by type, id"

	oh.hosting.c1.r1.d20.ct CT      1
	oh.hosting.c1.r2.d40.ct CT      1
	oh.hosting.c1.r4.d40.ct CT      1
	oh.hosting.c2.r4.d100.ct        CT      1
	oh.hosting.c2.r8.d100.ct        CT      1
	oh.hosting.c3.r6.d150.ct        CT      1
	oh.hosting.c4.r8.d200.ct        CT      1
	oh.hosting.c1.r1.d20    VM      1
	oh.hosting.c1.r2.d40    VM      1
	oh.hosting.c1.r4.d40    VM      1
	oh.hosting.c2.r4.d100   VM      1
	oh.hosting.c2.r8.d100   VM      1
	oh.hosting.c3.r6.d150   VM      1
	oh.hosting.c4.r8.d200   VM      1

### Example 5

	mf --json \
		"select id, type, enabled from product where ram_mib < 16384 and family in ('gd-vps4') order by type, id"

	[
		{
			"id": "oh.hosting.c1.r1.d20.ct",
			"type": "CT",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c1.r2.d40.ct",
			"type": "CT",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c1.r4.d40.ct",
			"type": "CT",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c2.r4.d100.ct",
			"type": "CT",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c2.r8.d100.ct",
			"type": "CT",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c3.r6.d150.ct",
			"type": "CT",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c4.r8.d200.ct",
			"type": "CT",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c1.r1.d20",
			"type": "VM",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c1.r2.d40",
			"type": "VM",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c1.r4.d40",
			"type": "VM",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c2.r4.d100",
			"type": "VM",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c2.r8.d100",
			"type": "VM",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c3.r6.d150",
			"type": "VM",
			"enabled": "1"
		},
		{
			"id": "oh.hosting.c4.r8.d200",
			"type": "VM",
			"enabled": "1"
		}
	]
