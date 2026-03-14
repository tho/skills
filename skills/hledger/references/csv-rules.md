# CSV Rules Reference

## Table of Contents

- [Overview](#overview)
- [Rules file location](#rules-file-location)
- [Minimal rules file](#minimal-rules-file)
- [Directives](#directives)
- [Field assignments](#field-assignments)
- [Conditional rules](#conditional-rules)
- [Amount handling](#amount-handling)
- [Advanced features](#advanced-features)
- [Complete examples](#complete-examples)

## Overview

hledger reads CSV/SSV/TSV files using a corresponding `.rules` file that describes
how to map CSV fields to hledger journal entries. File format is auto-detected from
extension (`.csv`, `.ssv`, `.tsv`) or prefix (`csv:`, `ssv:`, `tsv:`).

## Rules file location

By default, `foo/FILE.csv` looks for `foo/FILE.csv.rules`. Override with `--rules FILE`.

## Minimal rules file

```
skip         1
fields       date, description, , amount
date-format  %m/%d/%Y
```

This produces transactions with `expenses:unknown` and `income:unknown` accounts.

## Directives

### `skip`

Skip N header lines: `skip 1` or `skip N`.

### `separator`

Override field separator: `separator ;` or `separator \t` or `separator ","`.

### `date-format`

strptime format for parsing dates from CSV:

```
date-format %m/%d/%Y       # MM/DD/YYYY
date-format %-m/%-d/%Y     # M/D/YYYY (no leading zeros)
date-format %Y-%b-%d       # YYYY-Mmm-DD
date-format %-m/%-d/%Y %H:%M %p  # with time (parsed but unused)
```

Common format codes: `%Y` (4-digit year), `%y` (2-digit year), `%m` (month 01-12),
`%-m` (month 1-12), `%d` (day 01-31), `%-d` (day 1-31), `%b` (abbreviated month name).

### `newest-first`

Declare CSV record ordering: `newest-first` (default assumed) or `oldest-first`.
Affects overlap detection for `hledger import`.

### `intra-day-reversed`

Within each day, records are in reverse order relative to the file's overall date order.

### `decimal-mark`

Override decimal mark for amount parsing: `decimal-mark ,` or `decimal-mark .`.

### `timezone`

Add timezone to parsed dates: `timezone America/New_York`.

### `source`

Specify the CSV data file (supports globs): `source bank*.csv`.
Allows importing from rules file directly: `hledger import bank.csv.rules`.

### `archive`

Enable automatic archiving of imported CSV files to `data/` directory.

### `encoding`

Specify text encoding: `encoding windows-1252`.

### `balance-type`

Control balance assertion type: `balance-type ==` or `balance-type ==*`.

### `include`

Include another rules file:

```
include common.rules
include categorisation.rules
```

## Field assignments

### `fields` list

Name CSV columns and optionally assign them to hledger fields:

```
fields date, description, , amount
fields date, payee, note, amount, , , balance
```

Unnamed fields use empty or `_`. Named fields become available for `%fieldname` interpolation.

### Direct field assignment

Assign hledger fields directly (overrides `fields`):

```
date        %1            # from CSV column 1
description %3 | %4       # combine fields
account1    assets:checking
account2    expenses:unknown
amount      %4
comment     note: %5
```

### Available hledger fields

| Field | Purpose |
|-------|---------|
| `date` | Transaction date |
| `date2` | Secondary date |
| `status` | Transaction status (empty, `!`, `*`) |
| `code` | Transaction code |
| `description` | Transaction description |
| `comment` | Transaction or posting comment (use `comment1`, `comment2` for posting-specific) |
| `account1` | First posting account |
| `account2` | Second posting account |
| `accountN` | Nth posting account |
| `amount` | Sets amount for posting 1 (and negated for posting 2) |
| `amount1` | Amount for posting 1 only |
| `amount2` | Amount for posting 2 only |
| `amount-in` | Inflow amount (unsigned, applied to amount1) |
| `amount-out` | Outflow amount (unsigned, negated for amount1) |
| `amount1-in` / `amount1-out` | Separate in/out for posting 1 |
| `amount2-in` / `amount2-out` | Separate in/out for posting 2 |
| `currency` | Prepended to amounts (sets `currency1` and `currency2`) |
| `currency1` / `currency2` / `currencyN` | Currency for specific posting |
| `balance` | Balance after transaction (generates balance assertion) |
| `balance1` / `balance2` / `balanceN` | Balance for specific posting |

### Field interpolation

Reference CSV fields with `%fieldname` or `%N` (1-based column number):

```
fields date, , , gross, fee, net, , , , , , , txnid, title
description %title (%txnid)
comment     gross: %gross, fee: %fee
amount      %net
```

**Important**: `%fieldname` references the original CSV value, not a previously assigned
hledger field value.

## Conditional rules

### `if` block

Match CSV records and apply rules conditionally:

```
if MATCHER
 account2  expenses:groceries
 comment   category: food
```

Multiple matchers (OR logic):

```
if
MATCHER1
MATCHER2
 account2  expenses:food
```

### Matchers

Matchers are case-insensitive regular expressions matched against the entire CSV record
(all fields joined) by default:

```
if groceries
 account2  expenses:food:groceries

if walmart|costco|safeway
 account2  expenses:food:groceries
```

Field-specific matching with `%fieldname REGEX`:

```
if %description groceries
 account2  expenses:food:groceries

if %amount ^-
 account2  expenses:unknown
```

### Multiple matchers

Multiple matchers in an `if` block use OR logic (any match triggers):

```
if
amazon
amzn
 account2  expenses:shopping
```

For AND logic, use separate nested `if` blocks or `&` prefix:

```
if
& %description amazon
& %amount ^-[0-9]{3}
 account2  expenses:shopping:large
```

### Match groups

Capture groups in matchers can be referenced with `\1`, `\2`, etc.:

```
if %description (.+) TRANSFER
 account2  assets:\1
```

### `if` table

Compact tabular form for many simple categorization rules:

```
if,account2
groceries,expenses:food:groceries
pharmacy,expenses:health
gas station,expenses:transport:fuel
netflix|spotify,expenses:entertainment:subscriptions
```

The first field is the matcher, subsequent fields are assignments. The header row names
the target fields.

## Amount handling

### Setting amounts

Common patterns for CSV with separate debit/credit columns:

```
# Two amount columns (in/out)
fields date, description, amount-in, amount-out
```

```
# Single amount column (negative = outflow)
fields date, description, , amount
```

```
# Conditional amount based on type field
fields date, description, type, rawamt
if %type deposit
 amount  %rawamt
if %type withdrawal
 amount  -%rawamt
```

### Negating amounts

Negate with `-` prefix in assignment:

```
amount  -%4
```

Or conditionally:

```
if %4 ^[^-]
 amount  -%4
```

### Setting currency/commodity

```
# Fixed currency
currency $

# From CSV field
fields date, description, cur, amount
currency1 %cur
```

Or append to amount:

```
amount %4 USD
```

### Amount decimal places

Amount precision in CSV rules affects transaction balancing precision. If your bank
CSV has 2 decimal places, hledger checks balance to 2 decimal places.

## Advanced features

### Rapid feedback workflow

```
# Preview import without writing
hledger import bank.csv --dry-run

# Preview only uncategorized
hledger import --dry-run bank.csv | hledger -f- -I print unknown

# Watch for changes during rule development
watchexec -- "hledger import --dry-run bank.csv | hledger -f- -I print unknown"
```

### How CSV rules are evaluated

For each CSV record:

1. All directives (`skip`, `fields`, etc.) are applied in order
2. All `if` blocks are tested top to bottom; all matching blocks apply (last wins for same field)
3. Field assignments outside `if` blocks always apply
4. `%fieldname` interpolation uses original CSV values

### Well-factored rules

Split rules across files for reuse:

```
# bank-checking.csv.rules (account-specific)
source bank-checking*.csv
skip 1
fields date, description, amount
date-format %m/%d/%Y
account1 assets:bank:checking
include categorisation.rules

# categorisation.rules (shared)
if walmart|costco
 account2  expenses:food:groceries
if netflix|hulu
 account2  expenses:entertainment
```

## Complete examples

### Simple bank CSV

CSV:
```
Date,Description,Amount,Balance
01/15/2024,GROCERY STORE,-45.67,1234.56
01/16/2024,DIRECT DEPOSIT,3000.00,4234.56
```

Rules:
```
skip 1
fields date, description, amount, balance
date-format %m/%d/%Y
currency $
account1 assets:bank:checking
account2 expenses:unknown

if DIRECT DEPOSIT|PAYROLL
 account2  revenues:salary

if GROCERY|FOOD|SAFEWAY
 account2  expenses:food:groceries
```

### Credit card with separate debit/credit columns

CSV:
```
Trans Date,Post Date,Description,Category,Type,Amount
01/15/2024,01/16/2024,RESTAURANT XYZ,Food & Drink,Sale,-32.50
01/20/2024,01/20/2024,PAYMENT RECEIVED,Payment,Payment,500.00
```

Rules:
```
skip 1
fields date, date2, description, category, type, amount
date-format %m/%d/%Y
currency $
account1 liabilities:credit-card

if %type Payment
 account2  assets:bank:checking

if %category Food & Drink
 account2  expenses:food:dining

if %category Groceries
 account2  expenses:food:groceries
```

### PayPal with complex fields

```
skip 1
fields date, time, timezone, name, type, status, currency, gross, fee, net, , , txnid, title

date-format %m/%d/%Y
account1 assets:online:paypal
description %name | %title

# Ignore some events
if %type Temporary Hold|Currency Conversion|Authorization
 skip

# Set amounts
amount1 %net
amount2 -%gross

# Fee posting
if %fee [1-9]
 amount3 %fee
 account3 expenses:fees:paypal

# Categorize
if %type Payment|Purchase
 account2  expenses:unknown

if %type Transfer
 account2  assets:bank:checking
```

### Amazon orders

```
skip 1
fields date, orderid, title, category, , amzamount, , amztax
date-format %Y-%m-%d
code %orderid
description Amazon | %title
account1 liabilities:credit-card
account2 expenses:shopping

if %category Books
 account2  expenses:education:books

if %amztax [1-9]
 account3  expenses:tax:sales
 amount3   %amztax
 comment3  tax on: %title
```
