.. _Data-Type-User-Manual:

数据类型Type使用手册
^^^^^^^^^^^^^^^^^^^^^^^

 ::

    Type:{
            NumericType |	StringType | DateAndTimeType
    }

    NumericType:{
            IntegerType OptFieldLen FieldOpts
        |	BooleanType FieldOpts
        |	FixedPointType FloatOpt FieldOpts
        |	FloatingPointType FloatOpt FieldOpts
        |	BitValueType OptFieldLen
    }

    IntegerType:{
            "TINYINT" |	"SMALLINT" |	"MEDIUMINT"
        |	"INT" |	"INT1" |	"INT2" |	"INT3" |	"INT4" |	"INT8"
        |	"INTEGER" |	"BIGINT"
    }

    BooleanType:{
        "BOOL" |	"BOOLEAN"
    }

    FixedPointType:{
        "DECIMAL" |	"NUMERIC" |	"FIXED"
    }

    FloatingPointType:{
        "FLOAT" |	"REAL" |	"DOUBLE" |	"DOUBLE" "PRECISION"
    }

    BitValueType:{
        "BIT"
    }

    StringType:{
            Char FieldLen OptBinary
        |	Char OptBinary
        |	NChar FieldLen OptBinary
        |	NChar OptBinary
        |	Varchar FieldLen OptBinary
        |	NVarchar FieldLen OptBinary
        |	"BINARY" OptFieldLen
        |	"VARBINARY" FieldLen
        |	BlobType
        |	TextType OptCharsetWithOptBinary
        |	"ENUM" '(' StringList ')' OptCharset
        |	"SET" '(' StringList ')' OptCharset
        |	"LONG" Varchar OptCharsetWithOptBinary
        |	"LONG" OptCharsetWithOptBinary
    }

    Char:{
        "CHARACTER" |	"CHAR"
    }

    NChar:{
            "NCHAR"
        |	"NATIONAL" "CHARACTER"
        |	"NATIONAL" "CHAR"
    }

    Varchar:{
            "CHARACTER" "VARYING"
        |	"CHAR" "VARYING"
        |	"VARCHAR"
        |	"VARCHARACTER"
    }

    NVarchar:{
            "NATIONAL" "VARCHAR"
        |	"NATIONAL" "VARCHARACTER"
        |	"NVARCHAR"
        |	"NCHAR" "VARCHAR"
        |	"NCHAR" "VARCHARACTER"
        |	"NATIONAL" "CHARACTER" "VARYING"
        |	"NATIONAL" "CHAR" "VARYING"
        |	"NCHAR" "VARYING"
    }

    BlobType:
            "TINYBLOB"
        |	"BLOB" OptFieldLen
        |	"MEDIUMBLOB" | "LONGBLOB" |	"LONG" "VARBINARY"

    TextType:
        "TINYTEXT" |	"TEXT" OptFieldLen |	"MEDIUMTEXT" |	"LONGTEXT"

    DateAndTimeType:
            "DATE"
        |	"DATETIME" OptFieldLen
        |	"TIMESTAMP" OptFieldLen
        |	"TIME" OptFieldLen
        |	Year OptFieldLen FieldOpts

    Year:
        "YEAR"

    OptFieldLen:{
            ""
        |	FieldLen
    }

    FloatOpt:{
            ""
        |	FieldLen
        |	Precision
    }

    FieldLen:{
        '(' LengthNum ')'
    }

    Precision:{
        '(' LengthNum ',' LengthNum ')'
    }

    FieldOpts:{
            ""
        |	FieldOpts FieldOpt
    }

    FieldOpt:{
        "UNSIGNED" |	"SIGNED" | "ZEROFILL"
    }

    OptBinary:{
            ""
        |	"BINARY" OptCharset
        |	CharsetKw CharsetName OptBinMod
    }

    OptCharset:{
            ""
        |	CharsetKw CharsetName
    }

    CharsetKw:
        "CHARACTER" "SET"
    |	"CHARSET"
    |	"CHAR" "SET"

    CharsetName:
        StringName
    |	"BINARY"

    OptBinMod:
        ""
    |	"BINARY"

    StringList:
            stringLit
        |	StringList ',' stringLit

    OptCharsetWithOptBinary:
            OptBinary
        |	"ASCII" |	"UNICODE" |	"BYTE"