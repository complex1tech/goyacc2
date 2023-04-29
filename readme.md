Goyacc2
=======

Goyacc2 is a version of yacc for Go. It is written in Go and generates parsers written in Go.
It is a fork of standard [Goyacc](https://pkg.go.dev/golang.org/x/tools/cmd/goyacc) with additional
features.

Additional features:
- Includes via `// include: filename`.
- Extended `%type <field.type>` syntax with field names and types.

## Includes
Use `// include: filename.y` to include a file in a grammar file.

For example:
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

For example:
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

Goyacc2 extends this functionaly and adds support for `field.type` syntax, and
`$F` to access the field name, and `$T` to access the field type.

For example:
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
    { $F = ($T)($1) }
```

This rule will generate:
```go
yyVAL.val = (*Expr)(yyDollar[1].val.(*Expr))
```

© 2022 Ivan Korobkov
