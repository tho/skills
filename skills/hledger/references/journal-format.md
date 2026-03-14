# Journal Format Reference

## Table of Contents

- [Transaction syntax](#transaction-syntax)
- [Postings](#postings)
- [Amounts and commodities](#amounts-and-commodities)
- [Costs](#costs)
- [Balance assertions](#balance-assertions)
- [Tags](#tags)
- [Directives](#directives)
- [Periodic transactions](#periodic-transactions)
- [Auto postings](#auto-postings)

## Transaction syntax

```
DATE [!|*] [(CODE)] [DESCRIPTION]  [; COMMENT]
    [; COMMENT]
    ACCOUNT1  AMOUNT  [; COMMENT]
    ACCOUNT2  [AMOUNT]  [; COMMENT]
```

- Date: `YYYY-MM-DD` or `YYYY/MM/DD` or `YYYY.MM.DD` (year optional, inferred)
- Status: (empty) = unmarked, `!` = pending, `*` = cleared
- Code: optional, in parentheses, e.g. check number
- Description: optional. Use `PAYEE | NOTE` for separate payee/note fields
- Must have 2+ postings that sum to zero (one amount can be omitted)
- **Two or more spaces** required between account name and amount

Example:

```
2024-01-15 * (1234) Whole Foods | groceries  ; trip: january
    expenses:food:groceries     $45.67
    assets:bank:checking
```

## Postings

Each posting is indented (2+ spaces) and has:

```
    [!|*] ACCOUNT  [AMOUNT] [= BALANCE] [; COMMENT]
```

- Account names: any text with `:` hierarchy separator
- One amount per transaction can be omitted (inferred)
- Positive = debit (added to account), negative = credit (removed)

## Amounts and commodities

```
$1           # symbol left, no space
4000 AAPL    # symbol right, with space
3 "green apples"  # quoted commodity names (if non-letters)
-$1 or $-1   # negative amounts
1,000.00     # digit group marks
1.000,00     # European style (declare decimal-mark)
1E3          # scientific notation
```

Commodity display style is inferred from usage or set with `commodity` directive.

## Costs

Record exchange rates on postings:

```
# Per-unit cost
assets:euros  100 EUR @ $1.35

# Total cost
assets:euros  100 EUR @@ $135

# Implicit cost (inferred from balancing, not recommended)
assets:euros   100 EUR
assets:dollars  $-135
```

Report at cost with `-B`/`--cost`.

## Balance assertions

Assert account balance after a posting:

```
assets:checking  $0 = $500       # single commodity
assets:checking  $0 == $500      # sole commodity (no others)
assets           $0 =* $1000     # inclusive of subaccounts
assets           $0 ==* $1000    # sole + inclusive
```

Disable with `-I`/`--ignore-assertions`.

## Tags

Tags go in comments, as `tagname:` or `tagname: value`:

```
2024-01-15 groceries  ; trip: january, category: food
    expenses:food  $10  ; receipt: yes
    assets:checking
```

- Transaction tags propagate to postings (and vice versa)
- Account tags (in `account` directive) propagate to postings
- Query with `tag:NAME` or `tag:NAME=VALUE`
- Special tags: `date:` (posting date override), `type:` (account type)

## Directives

### `account`

Declare accounts for error checking, display order, and type:

```
account assets             ; type: A
account assets:bank        ; type: C
account liabilities        ; type: L
account equity             ; type: E
account revenues           ; type: R
account expenses           ; type: X
account equity:conversion  ; type: V
```

Types: `A` (Asset), `L` (Liability), `E` (Equity), `R` (Revenue), `X` (Expense),
`C` (Cash, subtype of A), `V` (Conversion, subtype of E).

### `commodity`

Declare commodity display style and enable error checking:

```
commodity $0.00
commodity 1.000,00 EUR
commodity 1000.00000000 BTC
```

### `decimal-mark`

Set decimal mark for amount parsing: `decimal-mark .` or `decimal-mark ,`.

### `payee`

Declare valid payee names: `payee Whole Foods`.

### `tag`

Declare valid tag names: `tag trip`.

### `P` (market price)

Declare market prices for value reporting:

```
P 2024-03-01 EUR $1.08
P 2024-03-01 AAPL $179.00
```

### `include`

Include another file: `include 2024-opening.journal`.

### `alias`

Rewrite account names:

```
alias checking = assets:bank:wells fargo:checking
alias /^expenses:food$/= expenses:food:other
```

### `D` (default commodity)

Set default commodity for bare numbers: `D $1000.00`.

### `Y` (default year)

Set default year for yearless dates: `Y 2024`.

### `comment` / `end comment`

Block comment:

```
comment
These lines are all ignored.
end comment
```

## Periodic transactions

Generate recurring transactions (for `--forecast`) or budget goals (for `--budget`):

```
~ monthly  set budget goals  ; <- 2+ spaces before description
    (expenses:rent)      $1000
    (expenses:food)       $500

~ every 15th day
    assets:checking       $-500
    expenses:rent          $500
```

Period expressions: `monthly`, `weekly`, `quarterly`, `yearly`, `every 2 weeks`,
`every 1st day`, `every 15th day of month`, `every Tue`, etc.

## Auto postings

Generate extra postings on matching transactions (with `--auto`):

```
= revenues:consulting
    liabilities:tax  *0.25
    expenses:tax    *-0.25
```

`*` means "multiply by the matched posting's amount".
