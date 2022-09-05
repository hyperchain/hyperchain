.. _DML-User-Manual:

DML语句使用手册
^^^^^^^^^^^^^^^^^^^^

INSERT
==============

 ::

    InsertIntoStmt:
        "INSERT" TableOptimizerHints PriorityOpt IgnoreOptional IntoOpt TableName PartitionNameListOpt InsertValues OnDuplicateKeyUpdate

    TableOptimizerHints:
            ""

    PriorityOpt:
            ""

    IgnoreOptional:
            ""

    IntoOpt:
        {} |	"INTO"

    TableName:
            Identifier
        |	Identifier '.' Identifier

    PartitionNameListOpt:
        ""

    InsertValues:
            '(' ColumnNameListOpt ')' ValueSym ValuesList
        |	ValueSym ValuesList
        |	"SET" ColumnSetValueList

    OnDuplicateKeyUpdate:
            ""

    ColumnNameListOpt:
        "" |	ColumnNameList

    ColumnNameList:
        ColumnName | ColumnNameList ',' ColumnName

    ValueSym:
        "VALUE" |	"VALUES"

    ValuesList:
        RowValue | ValuesList ',' RowValue

    RowValue:
        '(' ValuesOpt ')'

    ValuesOpt:
        "" | Values

    Values:
        Values ',' ExprOrDefault | ExprOrDefault

    ExprOrDefault:
        Expression | "DEFAULT"

    ColumnSetValueList:
        "" |	ColumnSetValue |	ColumnSetValueList ',' ColumnSetValue

    ColumnSetValue:
        ColumnName eq('=') ExprOrDefault


UPDATE
==============

 ::

    UpdateStmt:
        "UPDATE" TableRef "SET" AssignmentList WhereClauseOptional OrderByOptional LimitClause
    |	"UPDATE" TableRefs "SET" AssignmentList WhereClauseOptional

    Empty:
        /* empty */

    TableName:
        Identifier
    |	Identifier '.' Identifier

    TableRefs:
        EscapedTableRef
    |	TableRefs ',' EscapedTableRef

    EscapedTableRef:
        TableRef
    |	'{' Identifier TableRef '}'

    TableRef:
        TableFactor
    |	JoinTable

    TableFactor:
        TableName TableAsNameOpt
    |	'(' SelectStmt ')' TableAsName
    |	'(' SetOprStmt ')' TableAsName
    |	'(' TableRefs ')'

    TableAsNameOpt:
        Empty
    |	TableAsName

    TableAsName:
        Identifier
    |	"AS" Identifier

    JoinTable:
        TableRef CrossOpt TableRef
    |	TableRef CrossOpt TableRef "ON" Expression
    |	TableRef CrossOpt TableRef "USING" '(' ColumnNameList ')'
    |	TableRef JoinType OuterOpt "JOIN" TableRef "ON" Expression
    |	TableRef JoinType OuterOpt "JOIN" TableRef "USING" '(' ColumnNameList ')'
    |	TableRef "NATURAL" "JOIN" TableRef
    |	TableRef "NATURAL" JoinType OuterOpt "JOIN" TableRef
    |	TableRef "STRAIGHT_JOIN" TableRef
    |	TableRef "STRAIGHT_JOIN" TableRef "ON" Expression

    CrossOpt:
        "JOIN"
    |	"CROSS" "JOIN"
    |	"INNER" "JOIN"

    JoinType:
        "LEFT"
    |	"RIGHT"

    OuterOpt:
        Empty
    |	"OUTER"

    ColumnName:
        Identifier
    |	Identifier '.' Identifier

    ColumnNameList:
        ColumnName
    |	ColumnNameList ',' ColumnName

    AssignmentList:
        Assignment
    |	AssignmentList ',' Assignment

    Assignment:
        ColumnName eq ExprOrDefault

    ExprOrDefault:
        Expression
    |	"DEFAULT"

    WhereClauseOptional:
        Empty
    |	WhereClause

    WhereClause:
        "WHERE" Expression

    OrderByOptional:
        Empty
    |	OrderBy


    OrderBy:
        "ORDER" "BY" ByList

    ByList:
        ByItem
    |	ByList ',' ByItem

    ByItem:
        Expression Order

    Order:
        Empty
    |	"ASC"
    |	"DESC"

    LimitClause:
        Empty
    |	"LIMIT" LimitOption

    LimitOption:
        LengthNum

    LengthNum:
        NUM

    NUM:
        intLit

    Identifier:
        identifier

    /**************查阅DQL、Expression************/


    Expression:

    SelectStmt:

    SetOprStmt:


DELETE
==============

 ::

    DeleteFromStmt:
        "DELETE" "FROM" TableName TableAsNameOpt WhereClauseOptional OrderByOptional LimitClause
    |	"DELETE" TableAliasRefList "FROM" TableRefs WhereClauseOptional
    |	"DELETE" "FROM" TableAliasRefList "USING" TableRefs WhereClauseOptional

    Empty:
    /* empty */

    TableName:
        Identifier
    |	Identifier '.' Identifier

    TableAsNameOpt:
        Empty
    |	TableAsName

    TableAsName:
        Identifier
    |	"AS" Identifier

    WhereClauseOptional:
        Empty
    |	WhereClause

    WhereClause:
        "WHERE" Expression

    OrderByOptional:
        Empty
    |	OrderBy


    OrderBy:
        "ORDER" "BY" ByList

    ByList:
        ByItem
    |	ByList ',' ByItem

    ByItem:
        Expression Order

    Order:
        Empty
    |	"ASC"
    |	"DESC"

    LimitClause:
        Empty
    |	"LIMIT" LimitOption

    LimitOption:
        LengthNum

    LengthNum:
        NUM

    NUM:
        intLit

    TableAliasRefList:
        TableNameOptWild
    |	TableAliasRefList ',' TableNameOptWild

    TableNameOptWild:
        Identifier OptWild
    |	Identifier '.' Identifier OptWild

    OptWild:
        Empty
    |	'.' '*'

    TableRefs:
        EscapedTableRef
    |	TableRefs ',' EscapedTableRef

    EscapedTableRef:
        TableRef
    |	'{' Identifier TableRef '}'

    TableRef:
        TableFactor
    |	JoinTable

    TableFactor:
        TableName TableAsNameOpt
    |	'(' SelectStmt ')' TableAsName
    |	'(' SetOprStmt ')' TableAsName
    |	'(' TableRefs ')'


    JoinTable:
        /* Use %prec to evaluate production TableRef before cross join */
        TableRef CrossOpt TableRef
    |	TableRef CrossOpt TableRef "ON" Expression
    |	TableRef CrossOpt TableRef "USING" '(' ColumnNameList ')'
    |	TableRef JoinType OuterOpt "JOIN" TableRef "ON" Expression
    |	TableRef JoinType OuterOpt "JOIN" TableRef "USING" '(' ColumnNameList ')'
    |	TableRef "NATURAL" "JOIN" TableRef
    |	TableRef "NATURAL" JoinType OuterOpt "JOIN" TableRef
    |	TableRef "STRAIGHT_JOIN" TableRef
    |	TableRef "STRAIGHT_JOIN" TableRef "ON" Expression

    CrossOpt:
        "JOIN"
    |	"CROSS" "JOIN"
    |	"INNER" "JOIN"

    JoinType:
        "LEFT"
    |	"RIGHT"

    OuterOpt:
        Empty
    |	"OUTER"

    ColumnName:
        Identifier
    |	Identifier '.' Identifier

    ColumnNameList:
        ColumnName
    |	ColumnNameList ',' ColumnName

