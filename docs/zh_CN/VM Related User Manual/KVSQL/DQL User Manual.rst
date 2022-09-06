.. _DQL-User-Manual:

DQL语句使用手册
^^^^^^^^^^^^^^^^^^^^

Select
==========

 ::

    SelectStmt:
        SelectStmtBasic OrderByOptional SelectStmtLimit
    |	SelectStmtFromDualTable OrderByOptional SelectStmtLimit
    |	SelectStmtFromTable OrderByOptional SelectStmtLimit

    SelectStmtFromTable:
        SelectStmtBasic "FROM" TableRefsClause WhereClauseOptional SelectStmtGroup HavingClause

    SelectStmtBasic:
        "SELECT" SelectStmtOpts SelectStmtFieldList

    SelectStmtOpts:
        DefaultFalseDistinctOpt SelectStmtStraightJoin

    DefaultFalseDistinctOpt:
        /* empty */ // Default false
    |	DistinctOpt

    DefaultTrueDistinctOpt:
        /* empty */ // Default true
    |	DistinctOpt

    DistinctOpt:
        "ALL"
    |	DistinctKwd

    DistinctKwd:
        "DISTINCT"
    |	"DISTINCTROW"

    SelectStmtStraightJoin:
        /* empty */
    |	"STRAIGHT_JOIN"

    SelectStmtFieldList:
        FieldList

    FieldList:
        Field
    |	FieldList ',' Field

    Field:
        '*'
    |	Identifier '.' '*'
    |	Expression

    OrderByOptional:
        /* empty */
    |	OrderBy

    OrderBy:
        "ORDER" "BY" ByList

    ByList:
        ByItem
    |	ByList ',' ByItem

    ByItem:
        Expression Order

    Order:
        /* empty */
    |	"ASC"
    |	"DESC"

    SelectStmtLimit:
        /* empty */
    |	"LIMIT" LimitOption
    |	"LIMIT" LimitOption ',' LimitOption
    |	"LIMIT" LimitOption "OFFSET" LimitOption

    LimitOption:
        LengthNum

    LengthNum:
        NUM

    NUM:
        intLit

    SelectStmtFromDualTable:
        SelectStmtBasic FromDual WhereClauseOptional

    FromDual:
        "FROM" "DUAL"

    WhereClauseOptional:
        /* empty */
    |	WhereClause

    WhereClause:
        "WHERE" Expression

    TableRefsClause:
        TableRefs

    TableRefs:
        EscapedTableRef
    |	TableRefs ',' EscapedTableRef

    EscapedTableRef:
        TableRef

    TableRef:
        TableFactor
    |	JoinTable

    TableFactor:
        TableName TableAsNameOpt
    |	'(' SelectStmt ')' TableAsName
    |	'(' SetOprStmt ')' TableAsName
    |	'(' TableRefs ')'

    TableName:
        Identifier

    TableAsNameOpt:
        /* empty */
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

    ColumnNameList:
        ColumnName
    |	ColumnNameList ',' ColumnName

    ColumnName:
        Identifier
    |	Identifier '.' Identifier

    JoinType:
        "LEFT"
    |	"RIGHT"

    OuterOpt:
        /* empty */
    |	"OUTER"

    SetOprStmt:
        SetOprClauseList SetOpr SelectStmtBasic OrderByOptional SelectStmtLimit
    |	SetOprClauseList SetOpr SelectStmtFromDualTable OrderByOptional SelectStmtLimit
    |	SetOprClauseList SetOpr SelectStmtFromTable OrderByOptional SelectStmtLimit
    |	SetOprClauseList SetOpr '(' SelectStmt ')' OrderByOptional SelectStmtLimit

    SetOprClauseList:
        SetOprSelect
    |	SetOprClauseList SetOpr SetOprSelect

    SetOprSelect:
        SelectStmt
    |	'(' SelectStmt ')'

    SetOpr:
        "UNION" UnionOpt

    UnionOpt:
        DefaultTrueDistinctOpt

    SelectStmtGroup:
        /* empty */
    |	GroupByClause

    GroupByClause:
        "GROUP" "BY" ByList

    HavingClause:
        /* empty */
    |	"HAVING" Expression




