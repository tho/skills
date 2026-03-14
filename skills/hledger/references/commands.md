# Commands Reference

## Table of Contents

- [Report commands](#report-commands)
- [Financial statement commands](#financial-statement-commands)
- [Data entry and import](#data-entry-and-import)
- [Validation](#validation)
- [Other useful commands](#other-useful-commands)
- [Common options](#common-options)
- [Queries](#queries)
- [Period expressions](#period-expressions)

## Report commands

### `balance` (bal)

Flexible report showing accounts with numeric data:

```bash
hledger bal                       # all account balance changes
hledger bal expenses              # just expenses
hledger bal -M                    # monthly columns
hledger bal -M -t                 # monthly, tree mode
hledger bal -M expenses -S        # monthly expenses, sorted by amount
hledger bal -M --budget           # monthly budget report
hledger bal -H                    # historical (running) balances
hledger bal --depth 2             # limit account depth
hledger bal --layout wide         # multi-commodity on one line
hledger bal --layout bare         # symbols in separate column
hledger bal -% expenses           # as percentages
hledger bal --pivot payee         # group by payee instead of account
```

Key flags:
- `-l`/`--flat`: list mode (default), `-t`/`--tree`: tree mode
- `-H`/`--historical`: running balance from journal start
- `-S`/`--sort-amount`: sort by amount
- `-N`/`--no-total`: hide total row
- `-E`/`--empty`: show zero-balance accounts
- `--budget`: show actual vs budget from periodic transactions
- `-D`/`-W`/`-M`/`-Q`/`-Y`: daily/weekly/monthly/quarterly/yearly columns
- `-T`/`--row-total`, `-A`/`--average`: add total/average column
- `-r`/`--related`: show counterpart accounts
- `--transpose`: swap rows and columns
- `-O csv`/`-O html`/`-O json`: output format

### `register` (reg)

Show individual postings as a register:

```bash
hledger reg                       # all postings
hledger reg expenses:food         # food expenses
hledger reg -M expenses           # monthly totals
hledger reg -H checking           # running balance of checking
hledger reg -w 120                # wider output
hledger reg -p 'this month'       # current month only
```

Shows date, description, account, amount, running total. Useful for reconciliation.

### `aregister` (areg)

Show transactions affecting one account, like a bank statement:

```bash
hledger areg assets:checking              # checking register
hledger areg assets:checking -p thismonth  # this month
```

First argument is the account (regex). Shows one line per transaction with running balance.

### `print`

Show full journal entries:

```bash
hledger print                     # all transactions
hledger print -x                  # explicit (show all amounts)
hledger print date:thismonth      # this month
hledger print -m "grocery"        # fuzzy match one recent transaction
hledger print --new               # only new since last run
hledger print -O csv              # CSV output
hledger print -O beancount        # Beancount format
```

Output is valid journal format, can be piped to another hledger command:

```bash
hledger print assets:cash | hledger -f- -I reg expenses:food
```

## Financial statement commands

### `balancesheet` (bs)

Assets and liabilities with historical balances:

```bash
hledger bs                        # balance sheet
hledger bs -M                     # monthly
hledger bs --flat                 # flat list
hledger bs -O csv                 # CSV output
```

Shows accounts with type A/C (assets/cash) and L (liabilities). Sign is normalized
(positive = what you'd expect).

### `balancesheetequity` (bse)

Like `bs` but also shows equity. Useful to check accounting equation (A+L+E = 0).

### `incomestatement` (is)

Revenue and expenses:

```bash
hledger is                        # income statement
hledger is -M                     # monthly
hledger is -S                     # sorted by amount
hledger is --depth 2              # summarized
```

Shows accounts with type R (revenue) and X (expense).

### `cashflow` (cf)

Cash inflows and outflows (liquid asset accounts only):

```bash
hledger cf                        # cashflow statement
hledger cf -M                     # monthly
```

Shows accounts with type C (cash).

## Data entry and import

### `import`

Import new transactions from CSV/other files:

```bash
hledger import bank.csv                    # import new records
hledger import bank.csv --dry-run          # preview without writing
hledger import *.csv                       # import multiple files
hledger import bank.csv.rules              # import via rules file
hledger import --catchup bank.csv          # mark all as imported
```

Uses `.latest.FILENAME` files for overlap detection. Preview uncategorized:

```bash
hledger import --dry-run bank.csv | hledger -f- -I print unknown
```

### `add`

Interactive transaction entry at the terminal:

```bash
hledger add
hledger add today 'whole foods' expenses:food '$45'
```

## Validation

### `check`

Run correctness checks:

```bash
hledger check                              # basic checks (parseable, balanced, assertions)
hledger check -s                           # basic + strict (accounts, commodities declared)
hledger check ordereddates                 # dates in order
hledger check payees                       # all payees declared
hledger check tags                         # all tags declared
hledger check recentassertions             # recent balance assertions exist
hledger check uniqueleafnames              # no duplicate leaf account names
```

#### Basic checks (always run)
- `parseable`: valid syntax, files readable
- `autobalanced`: transactions balance
- `assertions`: balance assertions pass

#### Strict checks (`-s`/`--strict`)
- `balanced`: no implicit commodity conversions
- `commodities`: all commodity symbols declared
- `accounts`: all account names declared

### `diff`

Compare an account across two files:

```bash
hledger diff -f journal.journal -f bank.csv assets:checking
```

## Other useful commands

### `accounts`

```bash
hledger accounts                  # all accounts
hledger accounts --used           # only those with transactions
hledger accounts --undeclared     # used but not declared
hledger accounts --tree           # tree view
hledger accounts --types          # show account types
hledger accounts --directives     # output as account directives
```

### `stats`

```bash
hledger stats                     # journal summary
hledger stats -v                  # verbose (show file paths, commodities)
```

### `descriptions`, `payees`, `notes`, `tags`, `commodities`, `prices`

List unique values:

```bash
hledger descriptions
hledger payees --undeclared
hledger tags --values
hledger commodities --undeclared
hledger prices --infer-market-prices --show-reverse
```

### `close`

Generate closing/opening transactions for year-end:

```bash
hledger close --retain -e 2025          # retain earnings (close R, X to equity)
hledger close --migrate -e 2025         # migrate balances to new file
hledger close --assert                  # generate balance assertions
```

## Common options

| Option | Purpose |
|--------|---------|
| `-f FILE` | Read from FILE |
| `-b DATE` | Begin date |
| `-e DATE` | End date (exclusive) |
| `-p PERIOD` | Period expression |
| `-D`/`-W`/`-M`/`-Q`/`-Y` | Daily/weekly/monthly/quarterly/yearly |
| `-B`/`--cost` | Convert to cost |
| `-V`/`--market` | Convert to market value |
| `-X COMM` | Convert to specific commodity |
| `--depth N` or `-N` | Limit account depth |
| `-s`/`--strict` | Enable strict checks |
| `-I`/`--ignore-assertions` | Skip balance assertions |
| `--auto` | Apply auto posting rules |
| `--forecast` | Generate forecast transactions |
| `--infer-costs` | Infer costs from equity conversion postings |
| `--infer-equity` | Generate equity postings from costs |
| `-O FMT` | Output format (txt, csv, tsv, html, json) |
| `-o FILE` | Output to file |
| `--color=n` | Disable color |

## Queries

Filter commands with query arguments. Default is account name matching (case-insensitive regex):

```bash
hledger bal food                  # accounts containing "food"
hledger bal '^expenses\b'         # accounts starting with "expenses"
hledger bal 'food|dining'         # food or dining
```

### Query types

| Query | Matches |
|-------|---------|
| `acct:REGEX` | Account names (default, `acct:` optional) |
| `desc:REGEX` | Transaction descriptions |
| `payee:REGEX` | Payee (left of `\|` in description) |
| `note:REGEX` | Note (right of `\|` in description) |
| `date:PERIOD` | Dates within period |
| `amt:N` or `amt:'>N'` | Posting amounts |
| `cur:REGEX` | Commodity/currency symbol |
| `tag:NAME[=VAL]` | Tag name and optional value |
| `status:` / `status:!` / `status:*` | Unmarked / pending / cleared |
| `code:REGEX` | Transaction code |
| `depth:N` | Account depth limit |
| `type:CODES` | Account type (`ALERXCV`) |
| `real:` / `real:0` | Real / virtual postings |
| `not:QUERY` | Negate any query |

### Boolean queries

```bash
hledger print expr:'date:2024 and (desc:amazon or desc:amzn)'
hledger print expr:'food and not dining'
```

Operators: `AND`, `OR`, `NOT` (case-insensitive), parentheses for grouping.

## Period expressions

Used with `-p` option:

```bash
-p 2024                           # year 2024
-p "2024/1"                       # January 2024
-p "this month"                   # current month
-p "last quarter"                 # previous quarter
-p "from 2024/1/1 to 2024/4/1"   # Q1 2024
-p "monthly in 2024"              # monthly periods in 2024
-p "weekly from 2024/1/1"         # weekly from date
-p "bimonthly"                    # every 2 months
-p "every 15th day"               # by the 15th of each month
-p "every Tue"                    # weekly on Tuesday
```

### Smart dates

Used in `-b`, `-e`, `-p`, and `date:` queries:

```
2024, 2024-01, 2024-01-15        # specific dates
today, yesterday, tomorrow        # relative
this/last/next day/week/month/quarter/year
3 months ago, in 2 weeks
q1, q2, 2024q3                   # quarters
jan, feb, october                 # months
```

End dates are always exclusive.
