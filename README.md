# SQLGrep: Grep in MySQL database tables / fields

*If you do not know db schema (drank a lot yesterday, first day on new project or hacking alien starhip database)*

SQLGrep will examine db schema and search (`SELECT ... WHERE ...`) for specified text/number/regex/like (needle) in all fields of all tables.

## Install
`pip3 install sqlgrep`

## Examples
I use test db with just one table, so all found records will be in "libro" table.

~~~shell
# We want to know where price is stored and we know, at least one book costs 9.00
$ sqlgrep librodb 9 --limit 3
libro(id=9)/masterId 9
libro(id=9)/id 9
libro(id=30)/price 9.00

# LIKE search
$ sqlgrep librodb --like france% --limit 3
libro(id=944)/title FRANCESCO DI GIORGIO MARTINI: TEORIA E PRATICA PROPORZIONALE DA GIULIANOVA AI TRATTATI (con 5 appendici) - Montebello Mario - Demian edizioni - 1997 - Teramo
libro(id=3696)/title FRANCESCO BARACCA 1918-2008 - Varriale Paolo - Edizioni Rivista Aeronautica - 2008 - Roma
libro(id=4876)/title FRANCESCO CASORATI - Pansera Anty, Mantovani Giuseppe - Grafis - 1979 - Milano

# REGEXP search
$ sqlgrep  librodb --regex 'a{3}' --limit 3
libro(id=2841)/title AAARGH! - Halpenny Bruce Barrymore. - Casdec, - 1989
libro(id=13142)/title MERLUSSE - CIGALON. - Pagnol Marcel. - Faaasquelle éditeurs, - 1950
libro(id=24087)/title AAA! - Busi Aldo. - Bompiani, Assaggi, - 2010



~~~

## Speed
sqlgrep does one SQL SELECT ... WHERE query for each field in database. So, for db of 5 tables and 10 fields in each, there will be 50 queries (sending query to db is very simple and fast operation). All filtering are performed on database side (not in our slow python code), so it goes with maximal speed.

## Narrow your search and avoid false positives
Because of MySQL magic, sometimes empty/null values or other types are matched. To avoid it, use `--types TND` option. It will limit, which types of fields to examine. `T` is for all text fields (text, char, varchar), `N` for all numbers (decimal, int, smallint) and `D` for date and datetime. If you do not want to search in DATE/DATETIME fields, just use `--types TN`. Also, this will speed-up sqlgrep a little (if you look for price, most likely you do not need to search it in many large text fields).

Use `--tables Table1 Table2 Table3 ...` to search only in specific tables: 
~~~
sqlgrep --like --table libro -- librodb Artillery%`
~~~

## Database credentials
sqlgrep uses environment variables `MYSQL_HOST` (`--host`), `MYSQL_SOCKET` (`--socket`), `MYSQL_USER` (`-u`), `MYSQL_PASS` (`-p`).
By default, it tries to connect over FIFO socket (if it's found on system)

## Output options
`--all` - show full records (`SELECT * ...` result) for matching rows.

`--suppress` - do not print value of field, only table name, primary key value (if table has it) and field name, but not it's value. (To keep output short and clean)

`-v` / `--verbose` - verbose