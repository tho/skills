---
name: hledger
description: Use when working with hledger journal files (.journal, .j), CSV import rules (.csv.rules), hledger CLI commands, or when the user asks about plain text accounting, transaction entry, CSV import/categorization, balance reports, income statements, balance sheets, cashflow, account reconciliation, or financial data analysis with hledger.
---

# hledger

## Core Principles

- hledger uses double-entry bookkeeping: every transaction's postings must sum to zero
- Account hierarchy uses `:` separator (e.g. `assets:bank:checking`)
- **Two or more spaces** required between account name and amount in postings
- Commodity/currency symbol can go left (`$100`) or right (`100 EUR`) of the quantity
- Positive amounts = debit (added to account), negative = credit (removed)
- Five standard account types: `assets`, `liabilities`, `equity`, `revenues`, `expenses`

## Task Workflows

### 1. Creating CSV Import Rules

Read [references/csv-rules.md](references/csv-rules.md) for complete syntax and examples.

Workflow:

1. Examine the CSV file to understand column structure, date format, and amount conventions
2. Create a `.csv.rules` file (named `CSVFILE.csv.rules` by convention)
3. Set `skip`, `fields`, `date-format`, `currency`, and `account1`
4. Preview with: `hledger print -f bank.csv` or `hledger import --dry-run bank.csv`
5. Add `if` blocks to categorize transactions by description/payee
6. Iterate: `hledger import --dry-run bank.csv | hledger -f- -I print unknown`
7. Import: `hledger import bank.csv`

Key patterns:

```
# Minimal rules file
skip 1
fields date, description, amount
date-format %m/%d/%Y
currency $
account1 assets:bank:checking
account2 expenses:unknown

# Categorization
if walmart|costco|safeway
 account2  expenses:food:groceries

if netflix|spotify|hulu
 account2  expenses:entertainment:subscriptions

# If table (compact categorization)
if,account2
rent,expenses:housing:rent
electric,expenses:utilities:electric
water,expenses:utilities:water
```

### 2. Validating Transactions

Read [references/commands.md](references/commands.md) for the `check` command and query syntax.

```bash
# Basic validation (always runs: parseable, balanced, assertions)
hledger check

# Strict mode (also checks accounts, commodities are declared)
hledger check -s

# Additional checks
hledger check ordereddates        # dates in chronological order
hledger check recentassertions    # recent balance assertions exist

# Find undeclared accounts and generate directives
hledger accounts --undeclared --directives >> $LEDGER_FILE

# Preview imported transactions before committing
hledger import --dry-run bank.csv
hledger import --dry-run bank.csv | hledger -f- -I print unknown
```

Best practices for robust journals:
- Declare all accounts with `account` directives and `type:` tags
- Declare commodities with `commodity` directives
- Use `hledger check -s` regularly
- Add balance assertions from bank statements: `hledger close --assert`
- Run `hledger check ordereddates` to catch date entry errors

### 3. Reports and Analysis

Read [references/commands.md](references/commands.md) for full command reference, queries, and period expressions.

```bash
# Financial statements
hledger bs                        # balance sheet (assets, liabilities)
hledger is                        # income statement (revenues, expenses)
hledger cf                        # cashflow (liquid assets only)

# Multi-period reports
hledger is -M                     # monthly income statement
hledger bs -Q                     # quarterly balance sheet
hledger bal -M expenses -S -t     # monthly expenses, sorted, tree view

# Account registers
hledger areg assets:checking      # checking account register
hledger reg expenses:food -M      # monthly food spending

# Analysis queries
hledger bal expenses -p thismonth          # this month's expenses
hledger bal expenses -p 'last 3 months' -M # last 3 months monthly
hledger bal -M --depth 2 expenses          # summarized expenses
hledger bal --pivot payee expenses         # spending by payee
hledger bal -% expenses -p thisyear        # expense percentages
hledger bal --budget -M                    # budget vs actual

# Export
hledger bs -O csv -o report.csv           # CSV export
hledger is -O html -o report.html         # HTML export
```

### 4. Journal File Setup

Read [references/journal-format.md](references/journal-format.md) for complete format reference.

Recommended structure for a new personal finance journal:

```
# ~/.hledger.journal or set LEDGER_FILE

# Declare account types
account assets             ; type: A
account assets:bank        ; type: C
account assets:cash        ; type: C
account liabilities        ; type: L
account equity             ; type: E
account revenues           ; type: R
account expenses           ; type: X

# Declare commodities
commodity $0.00
decimal-mark .

# Include yearly files or CSV rules
include 2024.journal
```

## References

- **[references/csv-rules.md](references/csv-rules.md)**: Complete CSV rules syntax, directives, field assignments, conditional rules, amount handling, and real-world examples (bank, credit card, PayPal, Amazon)
- **[references/journal-format.md](references/journal-format.md)**: Transaction syntax, postings, amounts, costs, balance assertions, tags, all directives, periodic transactions, auto postings
- **[references/commands.md](references/commands.md)**: All report commands (balance, register, print, bs, is, cf), import/check commands, query syntax, period expressions, common options
