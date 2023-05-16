Goyacc2
=======

Goyacc2 is a version of yacc for Go. It is written in Go and generates parsers written in Go.
It is a fork of standard [Goyacc](https://pkg.go.dev/golang.org/x/tools/cmd/goyacc) with additional
features.

Additional features:
- Includes via `// include: filename`.
- Extended `%type <field:type> rule` syntax with field names and types.
- Table-like `%type rule <field:type>` syntax.
- Checks for duplicate rules.

## Includes
Use `// include: filename.y` to include a file in a grammar file.
Starting/ending `%%` are automatically trimmed from the included files.
Some editors need `%%` to correctly highlight `*.y` file syntax.

```
// include: stmt.y
// include: stmt_alter_database.y
// include: stmt_create_table.y
```

## Extended fields
Extended fields allow to use a single `value any` field in `union` but access and assign
it in a type-safe way.

Standard `goyacc` associates rules with fields in `yySymType` union struct via `%type <name>`
definitions. Even though definitions are called `types` they are actually field names.
Later, it allows to access and assign these fields in a type-safe manner 
via `$1` and `$$` syntax.

```
// yySymType
%union {
    bval  bool
    ival  int
    str   string
}

// Types
%type <bval>
    ok

%type <str>
    database_name
    table_name

// Rules
table_name:
    IDENT
    { $$ = $1 }
```

Goyacc2 extends this functionaly and adds support for `field:type` syntax, 
and `$T` to access the field type.

```
// yySymType
%union {
    val any
}

// Types
%type <val:int>
    int

%type <val:*Expr>
    expr

// Rules
expr:
    some_expr
    { $$ = $1 }
    some_expr_with_type
    { $$ = ($T)($1) }
```

These rules will generate:
```go
yyVAL.val = cast[*Expr](yyDollar[1].val)
// and
yyVAL.val = (*Expr)(cast[*Expr](yyDollar[1].val)) // Type check on assignment
```

## Table-like type syntax
Goyacc2 supports an inverted type syntax when a rule is followed by its type.
It allows to organizes types in a table sorted by rules.

```
%type Bit                       <type_name>
%type BitWithLength             <type_name>
%type BitWithoutLength          <type_name>
%type Character                 <type_name>
%type CharacterWithLength       <type_name>
%type CharacterWithoutLength    <type_name>
%type ColConstraint             <constraint>
%type ColConstraintElem         <constraint>
%type ColId                     <string>
%type ColLabel                  <string>
```

© 2023 Ivan Korobkov
