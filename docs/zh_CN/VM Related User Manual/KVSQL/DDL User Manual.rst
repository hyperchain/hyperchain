.. _DDL-User-Manual:

DDL语句使用手册
^^^^^^^^^^^^^^^^^^^^

CREATE
============

 ::

    "CREATE" OptTemporary "TABLE" IfNotExists TableName
                TableElementListOpt
                CreateTableOptionListOpt
                PartitionOpt DuplicateOpt AsOpt CreateTableSelectOpt

    "CREATE" OptTemporary "TABLE" IfNotExists TableName
                LikeTableWithOrWithoutParen

    OptTemporary:{
        ""	/* empty */
    }

    IfNotExists:{
        ""
        |	"IF" "NOT" "EXISTS"
    }

    TableName:{
            Identifier
    }

    TableElementListOpt:{
        ""	/* empty */
        |	'(' TableElementList ')'
    }

    TableElementList:{
            TableElement
        |	TableElementList ',' TableElement
    }

    TableElement:{
            ColumnDef
        |	Constraint
    }

    ColumnDef:{
            ColumnName Type ColumnOptionListOpt
        |	ColumnName "SERIAL" ColumnOptionListOpt
    }

    ColumnName:{
            Identifier
    }

    ColumnOptionListOpt:
        "" | ColumnOptionList

    ColumnOptionList:
        ColumnOption | ColumnOptionList ColumnOption

    ColumnOption:
            "NOT" "NULL"
        |	"NULL"
        |	"AUTO_INCREMENT"
        |	PrimaryOpt "KEY"
        |	"UNIQUE" | "UNIQUE" "KEY"
        |	"DEFAULT" DefaultValueExpr
        |	"SERIAL" "DEFAULT" "VALUE"
        |	"COMMENT" stringLit

    Constraint:
        ConstraintKeywordOpt ConstraintElem

    ConstraintKeywordOpt:
        "" |	"CONSTRAINT" |	"CONSTRAINT" Symbol(Identifier)

    ConstraintElem:
            "PRIMARY" "KEY" IndexNameAndTypeOpt '(' IndexPartSpecificationList ')' IndexOptionList
        |	KeyOrIndex IndexNameAndTypeOpt '(' IndexPartSpecificationList ')' IndexOptionList
        |	"UNIQUE" KeyOrIndexOpt IndexNameAndTypeOpt '(' IndexPartSpecificationList ')' IndexOptionList

    IndexNameAndTypeOpt:
        IndexName

    IndexPartSpecificationList:
        IndexPartSpecification | IndexPartSpecificationList ',' IndexPartSpecification

    IndexPartSpecification:
        ColumnName OptFieldLen Order

    IndexOptionList:
        "" | IndexOptionList IndexOption

    IndexOption:
    |	"COMMENT" stringLit

    KeyOrIndex:
        "KEY" |	"INDEX"

    CreateTableOptionListOpt:
        "" |	TableOptionList

    TableOptionList:
        TableOption |	TableOptionList TableOption |	TableOptionList ',' TableOption

    TableOption:
        "AUTO_INCREMENT" EqOpt LengthNum
    |	"UNION" EqOpt '(' TableNameListOpt ')'

    PartitionOpt:
        ""

    DuplicateOpt:
        ""

    AsOpt:
        "" | "AS"

    CreateTableSelectOpt:
        ""
