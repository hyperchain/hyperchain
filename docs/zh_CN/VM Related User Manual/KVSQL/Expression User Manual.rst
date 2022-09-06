.. _Expression-User-Manual:

Expression 使用手册
^^^^^^^^^^^^^^^^^^^^^^^^

buildin Function映射表
===========================

    mysql中支持内置函数，内置函数在词法分析阶段就会被解析成特定的终结符类型，下面给出内置函数到终结符类型的映射关系表。

    ============ ==================
    函数名       终结符类型
    ============ ==================
    ADDDATE      builtinAddDate
    BIT_AND      builtinBitAnd
    BIT_OR       builtinBitOr
    BIT_XOR      builtinBitXor
    CAST         builtinCast
    COUNT        builtinCount
    CURDATE      builtinCurDate
    CURTIME      builtinCurTime
    DATE_ADD     builtinDateAdd
    DATE_SUB     builtinDateSub
    EXTRACT      builtinExtract
    GROUP_CONCAT builtinGroupConcat
    MAX          builtinMax
    MID          builtinSubstring
    MIN          builtinMin
    NOW          builtinNow
    POSITION     builtinPosition
    SESSION_USER builtinUser
    STD          builtinStddevPop
    STDDEV       builtinStddevPop
    STDDEV_POP   builtinStddevPop
    STDDEV_SAMP  builtinStddevSamp
    SUBDATE      builtinSubDate
    SUBSTR       builtinSubstring
    SUBSTRING    builtinSubstring
    SUM          builtinSum
    SYSDATE      builtinSysDate
    SYSTEM_USER  builtinUser
    TRIM         builtinTrim
    VARIANCE     builtinVarPop
    VAR_POP      builtinVarPop
    VAR_SAMP     builtinVarSamp
    ============ ==================

    ::

    %{%}

    %token	<ident>
        pipes              "||"

        singleAtIdentifier "identifier with single leading at"
            doubleAtIdentifier "identifier with double leading at"

        binaryType        "BINARY"

    %token  <item>
        assignmentEq ":="

        /* The following tokens belong to ReservedKeyword. Notice: make sure these tokens are contained in ReservedKeyword. */
        falseKwd          "FALSE"
        trueKwd           "TRUE"

        /* The following tokens belong to UnReservedKeyword. Notice: make sure these tokens are contained in UnReservedKeyword. */
        pipesAsOr
        weightString          "WEIGHT_STRING"

        /* The following tokens belong to TiDBKeyword. Notice: make sure these tokens are contained in TiDBKeyword. */
        builtinAddDate
        builtinBitAnd
        builtinBitOr
        builtinBitXor
        builtinCast
        builtinCount
        builtinCurDate
        builtinCurTime
        builtinDateAdd
        builtinDateSub
        builtinExtract
        builtinGroupConcat
        builtinMax
        builtinMin
        builtinNow
        builtinPosition
        builtinSubDate
        builtinSubstring
        builtinSum
        builtinSysDate
        builtinStddevPop
        builtinStddevSamp
        builtinTrim
        builtinUser
        builtinVarPop
        builtinVarSamp

    %token	<item>
        jss          "->"
        juss         "->>"
    %%

    Expression:
        singleAtIdentifier assignmentEq Expression
    |	Expression logOr Expression
    |	Expression "XOR" Expression
    |	Expression logAnd Expression
    |	"NOT" Expression
    |	BoolPri IsOrNotOp trueKwd
    |	BoolPri IsOrNotOp falseKwd
    |	BoolPri

    BitExpr:
        BitExpr '|' BitExpr
    |	BitExpr '&' BitExpr
    |	BitExpr "<<" BitExpr
    |	BitExpr ">>" BitExpr
    |	BitExpr '+' BitExpr
    |	BitExpr '-' BitExpr
    |	BitExpr '*' BitExpr
    |	BitExpr '/' BitExpr
    |	BitExpr '%' BitExpr
    |	BitExpr "DIV" BitExpr
    |	BitExpr "MOD" BitExpr
    |	BitExpr '^' BitExpr
    |	SimpleExpr

    SimpleExpr:
        SimpleIdent
    |	FunctionCallKeyword
    |	FunctionCallNonKeyword
    |	FunctionCallGeneric
    |	Literal
    |	SumExpr
    |	'!' SimpleExpr
    |	'~' SimpleExpr
    |	'-' SimpleExpr
    |	'+' SimpleExpr
    |	SimpleExpr pipes SimpleExpr
    |	not2 SimpleExpr
    |	SubSelect
    |	'(' Expression ')'
    |	'(' ExpressionList ',' Expression ')'
    |	"ROW" '(' ExpressionList ',' Expression ')'
    |	"EXISTS" SubSelect
    |	"BINARY" SimpleExpr
    |	builtinCast '(' Expression "AS" CastType ')'
    |	"CASE" ExpressionOpt WhenClauseList ElseOpt "END"
    |	"CONVERT" '(' Expression ',' CastType ')'

    SimpleIdent:
        Identifier
    |	Identifier '.' Identifier
    |	'.' Identifier '.' Identifier

    BoolPri:
        BoolPri IsOrNotOp "NULL"
    |	BoolPri CompareOp PredicateExpr
    |	BoolPri CompareOp AnyOrAll SubSelect
    |	PredicateExpr

    PredicateExpr:
        BitExpr InOrNotOp '(' ExpressionList ')'
    |	BitExpr InOrNotOp SubSelect
    |	BitExpr BetweenOrNotOp BitExpr "AND" PredicateExpr
    |	BitExpr LikeOrNotOp SimpleExpr LikeEscapeOpt
    |	BitExpr RegexpOrNotOp SimpleExpr
    |	BitExpr

    InOrNotOp:
        "IN"
    |	"NOT" "IN"

    ExpressionList:
        Expression
    |	ExpressionList ',' Expression

    //FulltextSearchModifierOpt:
    //	/* empty */

    ColumnNameList:
        ColumnName
    |	ColumnNameList ',' ColumnName

    ColumnName:
        Identifier
    |	Identifier '.' Identifier

    IsOrNotOp:
        "IS"
    |	"IS" "NOT"

    BetweenOrNotOp:
        "BETWEEN"
    |	"NOT" "BETWEEN"

    LikeOrNotOp:
        "LIKE"
    |	"NOT" "LIKE"

    LikeEscapeOpt:
        /* empty */
    |	"ESCAPE" stringLit

    RegexpOrNotOp:
        RegexpSym
    |	"NOT" RegexpSym

    RegexpSym:
        "REGEXP"
    |	"RLIKE"

    logOr:
        pipesAsOr
    |	"OR"

    CompareOp:
        ">="
    |	'>'
    |	"<="
    |	'<'
    |	"!="
    |	"<>"
    |	"="
    |	"<=>"  // NullEQ

    AnyOrAll:
        "ANY"
    |	"SOME"
    |	"ALL"

    SubSelect:
        '(' SelectStmt ')' // please see DQL
    |	'(' SetOprStmt ')' // please see DQL

    FunctionCallKeyword:
        FunctionNameConflict '(' ExpressionListOpt ')'
    |	FunctionNameOptionalBraces OptionalBraces
    |	builtinCurDate '(' ')'
    |	FunctionNameDatetimePrecision FuncDatetimePrec
    |	"MOD" '(' BitExpr ',' BitExpr ')'


    FunctionCallNonKeyword:
        builtinCurTime '(' FuncDatetimePrecListOpt ')'
    |	builtinTrim '(' Expression ')'
    |	builtinTrim '(' Expression "FROM" Expression ')'
    |	builtinTrim '(' TrimDirection "FROM" Expression ')'
    |	builtinTrim '(' TrimDirection Expression "FROM" Expression ')'

    FunctionCallGeneric:
        identifier '(' ExpressionListOpt ')'
    |	Identifier '.' Identifier '(' ExpressionListOpt ')'

    ExpressionListOpt:
        /* empty */
    |	ExpressionList

    FunctionNameConflict:
    |	"IF"

    FunctionNameOptionalBraces:
    |	"CURRENT_DATE"
    |	"UTC_DATE"

    FunctionNameDatetimePrecision:
        "CURRENT_TIME"
    |	"CURRENT_TIMESTAMP"
    |	"UTC_TIME"
    |	"UTC_TIMESTAMP"

    OptionalBraces:
        /* empty */
    |	'(' ')'

    FuncDatetimePrec:
        /* empty */
    |	'(' ')'
    |	'(' intLit ')'

    StringName:
        stringLit
    |	Identifier

    FuncDatetimePrecListOpt:
        /* empty */
    |	FuncDatetimePrecList

    FuncDatetimePrecList:
        intLit

    TableName:
        Identifier
    |	Identifier '.' Identifier

    SignedNum:
        Int64Num
    |	'+' Int64Num
    |	'-' NUM

    Int64Num:
        NUM

    NUM:
        intLit

    Literal:
        "FALSE"
    |	"NULL"
    |	"TRUE"
    |	floatLit
    |	decLit
    |	intLit
    |	StringLiteral
    |	"UNDERSCORE_CHARSET" stringLit
    |	hexLit
    |	bitLit

    StringLiteral:
        stringLit
    |	StringLiteral stringLit

    SumExpr:
        "AVG" '(' BuggyDefaultFalseDistinctOpt Expression ')'
    |	builtinBitAnd '(' Expression ')'
    |	builtinBitAnd '(' "ALL" Expression ')'
    |	builtinBitOr '(' Expression ')'
    |	builtinBitOr '(' "ALL" Expression ')'
    |	builtinBitXor '(' Expression ')'
    |	builtinBitXor '(' "ALL" Expression ')'
    |	builtinCount '(' DistinctKwd ExpressionList ')'
    |	builtinCount '(' "ALL" Expression ')'
    |	builtinCount '(' Expression ')'
    |	builtinCount '(' '*' ')'
    |	builtinGroupConcat '(' BuggyDefaultFalseDistinctOpt ExpressionList OrderByOptional OptGConcatSeparator ')'
    |	builtinMax '(' BuggyDefaultFalseDistinctOpt Expression ')'
    |	builtinMin '(' BuggyDefaultFalseDistinctOpt Expression ')'
    |	builtinSum '(' BuggyDefaultFalseDistinctOpt Expression ')'
    |	builtinStddevPop '(' BuggyDefaultFalseDistinctOpt Expression ')'
    |	builtinStddevSamp '(' BuggyDefaultFalseDistinctOpt Expression ')'
    |	builtinVarPop '(' BuggyDefaultFalseDistinctOpt Expression ')'
    |	builtinVarSamp '(' BuggyDefaultFalseDistinctOpt Expression ')'

    BuggyDefaultFalseDistinctOpt:
        DefaultFalseDistinctOpt
    |	DistinctKwd "ALL"

    DefaultFalseDistinctOpt:
        /* empty */
    |	DistinctOpt

    DistinctOpt:
        "ALL"
    |	DistinctKwd

    DistinctKwd:
        "DISTINCT"
    |	"DISTINCTROW"

    OrderByOptional:
        /* empty */
    |	OrderBy

    OrderBy:
        "ORDER" "BY" ByList

    OrderBy:
        "ORDER" "BY" ByList

    ByList:
        ByItem
    |	ByList ',' ByItem

    ByItem:
        Expression Order

    Order:
        /* EMPTY */
    |	"ASC"
    |	"DESC"

    OptGConcatSeparator:
        /* empty */
    |	"SEPARATOR" stringLit

    CastType:
        "BINARY" OptFieldLen
    |	Char OptFieldLen OptBinary
    |	"DATE"
    |	"DATETIME" OptFieldLen
    |	"DECIMAL" FloatOpt
    |	"TIME" OptFieldLen
    |	"SIGNED" OptInteger
    |	"UNSIGNED" OptInteger
    //|	"JSON"
    |	"DOUBLE"
    |	"FLOAT" FloatOpt
    |	"REAL"

    OptFieldLen:
        /* empty */
    |	FieldLen

    FieldLen:
        '(' LengthNum ')'

    LengthNum:
        NUM

    NUM:
        intLit

    OptBinary:
        /* empty */


    FloatOpt:
        /* empty */
    |	FieldLen
    |	Precision

    Precision:
        '(' LengthNum ',' LengthNum ')'

    OptInteger:
        /* empty */
    |	"INTEGER"
    |	"INT"

    ExpressionOpt:
        /* empty */
    |	Expression

    WhenClauseList:
        WhenClause
    |	WhenClauseList WhenClause

    WhenClause:
        "WHEN" Expression "THEN" Expression

    ElseOpt:
        /* empty */
    |	"ELSE" Expression

    %%



