# PL/pgSQL lint

A PostgreSQL's plpgsql interpreter uses a two step checking. First step is syntax checking when
function is validated - it is done on function's creation time or when function is executed
first time in a session. Second step - a deeper checks of embedded SQL and expressions are done 
in runtime when SQL or expression is evaluated first time in a session. This step is slower and
this technique eliminates checking of SQL or expressions that are never evaluated (but some errors
can be found too late).

plpgsql_lint ensures a deep validation of all embedded SQL and expressions (not only evaluated)
every time when function is started.


## Installation
 * copy the source code to PostgreSQL's source code tree (8.4.x, 9.0.x, 9.1.x, [9.2.x]) - to _contrib_ directory
 * compile it there and install it - _make; make install_

## Usage

    postgres=# load 'plpgsql';
    LOAD
    postgres=# load 'plpgsql_lint';
    LOAD
    postgres=# CREATE TABLE t1(a int, b int);
    CREATE TABLE

    postgres=# CREATE OR REPLACE FUNCTION public.f1()
               RETURNS void
               LANGUAGE plpgsql
               AS $function$
               DECLARE r record;
               BEGIN
                 FOR r IN SELECT * FROM t1
                 LOOP
                   RAISE NOTICE '%', r.c; -- there is bug - table t1 missing "c" column
                 END LOOP;
               END;
               $function$;
    CREATE FUNCTION

    postgres=# select f1();
    ERROR:  record "r" has no field "c"
    CONTEXT:  SQL statement "SELECT r.c"
    PL/pgSQL function "f1" line 5 at RAISE

The function f1() can be executed successfully without active plpgsql_lint, because table t1
is empty and RAISE statement will never be executed. When plpgsql_lint is active, then this
badly written expression is identified.

This module can be deactivated by setting

    SET plpgsql.enable_lint TO false;

## Limits

plpgsql_lint should find all errors on really static code. When developer uses some
PLpgSQL's dynamic features like dynamic SQL or record data type, then false positives are
possible. These should be rare - in well written code - and then the affected function
should be redesigned or plpgsql_lint should be disabled for this function.


    CREATE OR REPLACE FUNCTION f1()
    RETURNS void AS $$
    DECLARE r record;
    BEGIN
      FOR r IN EXECUTE 'SELECT * FROM t1'
      LOOP
        RAISE NOTICE '%', r.c;
      END LOOP;
    END;
    $$ LANGUAGE plpgsql SET plpgsql.enable_lint TO false;

_A usage of plpgsql_lint adds a small overhead and you should use it only in develop or preprod environments._

### Dynamic SQL

This module doesn't check queries that are assembled in runtime. It is not possible
to identify result of dynamic queries - so plpgsql_cannot to set correct type to record
variables and cannot to check a dependent SQLs and expressions. Don't use record variable
as target for dynamic queries or disable plpgsql_lint for functions that use a dynamic
queries.

<<<<<<< HEAD
=======
### PL/pgSQL statements

plpgsql_lint checks only embedded SQL and expressions. It doesn't check a PL/pgSQL
statements. There are a few bugs that are not identified by syntax checking and plpgsql_lint
too:

    RAISE NOTICE '% %', var1, var2, var3; -- three variables instead two
                                          -- plpgsql_lint is blind in this case :(

>>>>>>> 007cfe2f8998aa59a59a47f8828f6eed0590b399
## Licence

Copyright (c) Pavel Stehule (pavel.stehule@gmail.com)

 Permission is hereby granted, free of charge, to any person obtaining a copy
 of this software and associated documentation files (the "Software"), to deal
 in the Software without restriction, including without limitation the rights
 to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 copies of the Software, and to permit persons to whom the Software is
 furnished to do so, subject to the following conditions:

 The above copyright notice and this permission notice shall be included in
 all copies or substantial portions of the Software.

 THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 THE SOFTWARE.

## Note

If you like it, send a postcard to address

    Pavel Stehule
    Skalice 12
    256 01 Benesov u Prahy
    Czech Republic
