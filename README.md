CPPPHP
======

Dummy C++ learning project of a PHP AST (abstract syntax tree).

The plan is to turn PHP code like this:

```php
<?php

function foo() {
  echo("Hello");
}

$a = 1;
foo();
```

To tokens like this:

```
T(PHP_START=<?php)
T(KEYWORD=function)
T(SPEC_NAME=foo)
T(PARENTHESIS_OPEN=()
T(PARENTHESIS_CLOSE=))
T(BRACKET_OPEN={)
T(KEYWORD=echo)
T(PARENTHESIS_OPEN=()
T(STRING=Hello)
T(PARENTHESIS_CLOSE=))
T(SEMICOLON=;)
T(BRACKET_CLOSE=})
T(VARIABLE=$a)
T(OP_ASSIGNMENT==)
T(NUMERIC=1)
T(SEMICOLON=;)
T(SPEC_NAME=foo)
T(PARENTHESIS_OPEN=()
T(PARENTHESIS_CLOSE=))
T(SEMICOLON=;)
```

Then make a (simplified) language grammar like this:

```
PROG: T(PHP_START) CODE
CODE: ( FUNC | STMT )*
FUNC: T(KEYWORD=function) T(SPEC_NAME) T(PARENTHESIS_OPEN) ARGF T(PARENTHESIS_CLOSE) T(BRACKET_OPEN) CODE T(BRACKET_CLOSE)
ARGF: ( T(VARIABLE) ( T(COMMA) T(VARIABLE) )* )*
STMT: ( ( ASSI | EXPR ) T(SEMICOLON) )*
ASSI: T(VARIABLE) T(OP_ASSIGNMENT) EXPR
EXPR: RVAL | FCAL | T(PARENTHESIS_OPEN) EXPR T(PARENTHESIS_CLOSE) | EXPR T(OP_PLUS) EXPR | EXPR T(OP_MINUS) EXPR | EXPR T(OP_MULTIPLY) EXPR | EXPR T(OP_DIVIDE) EXPR | EXPR T(OP_MODULO) EXPR
FCAL: T(SPEC_NAME) T(PARENTHESIS_OPEN) ARGC T(PARENTHESIS_CLOSE) | T(KEYWORD) T(PARENTHESIS_OPEN) ARGC T(PARENTHESIS_CLOSE)
RVAL: T(VARIABLE) | T(STRING) | T(NUMERIC)
ARGC: ( EXPR ( T(COMMA) EXPR )* )*
```

To turn to an AST rule tree like this:

```
ARGC
  OR GROUP:
    GROUP:
      OR GROUP (*):
        GROUP:
          DEFINITION: EXPR
          OR GROUP (*):
            GROUP:
              TOKEN: T(COMMA=?)
              DEFINITION: EXPR

ARGF
  OR GROUP:
    GROUP:
      OR GROUP (*):
        GROUP:
          TOKEN: T(VARIABLE=?)
          OR GROUP (*):
            GROUP:
              TOKEN: T(COMMA=?)
              TOKEN: T(VARIABLE=?)

ASSI
  OR GROUP:
    GROUP:
      TOKEN: T(VARIABLE=?)
      TOKEN: T(OP_ASSIGNMENT=?)
      DEFINITION: EXPR

CODE
  OR GROUP:
    GROUP:
      OR GROUP (*):
        GROUP:
          DEFINITION: FUNC
        GROUP:
          DEFINITION: STMT

EXPR
  OR GROUP:
    GROUP:
      DEFINITION: RVAL
    GROUP:
      DEFINITION: FCAL
    GROUP:
      TOKEN: T(PARENTHESIS_OPEN=?)
      DEFINITION: EXPR
      TOKEN: T(PARENTHESIS_CLOSE=?)
    GROUP:
      DEFINITION: EXPR
      TOKEN: T(OP_PLUS=?)
      DEFINITION: EXPR
    GROUP:
      DEFINITION: EXPR
      TOKEN: T(OP_MINUS=?)
      DEFINITION: EXPR
    GROUP:
      DEFINITION: EXPR
      TOKEN: T(OP_MULTIPLY=?)
      DEFINITION: EXPR
    GROUP:
      DEFINITION: EXPR
      TOKEN: T(OP_DIVIDE=?)
      DEFINITION: EXPR
    GROUP:
      DEFINITION: EXPR
      TOKEN: T(OP_MODULO=?)
      DEFINITION: EXPR

FCAL
  OR GROUP:
    GROUP:
      TOKEN: T(SPEC_NAME=?)
      TOKEN: T(PARENTHESIS_OPEN=?)
      DEFINITION: ARGC
      TOKEN: T(PARENTHESIS_CLOSE=?)
    GROUP:
      TOKEN: T(KEYWORD=?)
      TOKEN: T(PARENTHESIS_OPEN=?)
      DEFINITION: ARGC
      TOKEN: T(PARENTHESIS_CLOSE=?)

FUNC
  OR GROUP:
    GROUP:
      TOKEN: T(KEYWORD=function)
      TOKEN: T(SPEC_NAME=?)
      TOKEN: T(PARENTHESIS_OPEN=?)
      DEFINITION: ARGF
      TOKEN: T(PARENTHESIS_CLOSE=?)
      TOKEN: T(BRACKET_OPEN=?)
      DEFINITION: CODE
      TOKEN: T(BRACKET_CLOSE=?)

PROG
  OR GROUP:
    GROUP:
      TOKEN: T(PHP_START=?)
      DEFINITION: CODE

RVAL
  OR GROUP:
    GROUP:
      TOKEN: T(VARIABLE=?)
    GROUP:
      TOKEN: T(STRING=?)
    GROUP:
      TOKEN: T(NUMERIC=?)

STMT
  OR GROUP:
    GROUP:
      OR GROUP (*):
        GROUP:
          OR GROUP:
            GROUP:
              DEFINITION: ASSI
            GROUP:
              DEFINITION: EXPR
          TOKEN: T(SEMICOLON=?)
```

So you can build up the AST from the tokens:

```
or
  concat
    T(PHP_START=<?php)
    or
      concat
        or
          concat
            or
              concat
                T(KEYWORD=function)
                T(SPEC_NAME=foo)
                T(PARENTHESIS_OPEN=()
                or
                  concat
                    or
                T(PARENTHESIS_CLOSE=))
                T(BRACKET_OPEN={)
                or
                  concat
                    or
                      concat
                        or
                          concat
                            or
                              concat
                                or
                                  concat
                                    or
                                      concat
                                        or
                                          concat
                                            T(KEYWORD=echo)
                                            T(PARENTHESIS_OPEN=()
                                            or
                                              concat
                                                or
                                                  concat
                                                    or
                                                      concat
                                                        or
                                                          concat
                                                            T(STRING=Hello)
                                                    or
                                            T(PARENTHESIS_CLOSE=))
                                T(SEMICOLON=;)
                      concat
                        or
                          concat
                            or
                T(BRACKET_CLOSE=})
          concat
            or
              concat
                or
                  concat
                    or
                      concat
                        or
                          concat
                            T(VARIABLE=$a)
                            T(OP_ASSIGNMENT==)
                            or
                              concat
                                or
                                  concat
                                    T(NUMERIC=1)
                    T(SEMICOLON=;)
                  concat
                    or
                      concat
                        or
                          concat
                            or
                              concat
                                T(SPEC_NAME=foo)
                                T(PARENTHESIS_OPEN=()
                                or
                                  concat
                                    or
                                T(PARENTHESIS_CLOSE=))
                    T(SEMICOLON=;)
```