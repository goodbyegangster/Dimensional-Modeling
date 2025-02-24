# Basic Dimension Table Techniques <!-- omit in toc -->

- [Dimension Table Structure](#dimension-table-structure)
- [Dimension Surrogate Keys](#dimension-surrogate-keys)
- [Natural, durable and supernatural keys](#natural-durable-and-supernatural-keys)
- [Drilling Down](#drilling-down)
- [Degenerate dimensions](#degenerate-dimensions)
- [Denormalized flattened dimensions](#denormalized-flattened-dimensions)
- [Multiple Hierarchies in Dimensions](#multiple-hierarchies-in-dimensions)
- [Flags and Indicators as Textual Dimension Attributes](#flags-and-indicators-as-textual-dimension-attributes)
- [Null Attributes in Dimensions](#null-attributes-in-dimensions)
- [Calendar Date Dimensions](#calendar-date-dimensions)
- [Role-Playing Dimensions](#role-playing-dimensions)
- [Junk Dimensions](#junk-dimensions)
- [Snowflaked Dimensions](#snowflaked-dimensions)
- [Outrigger Dimensions](#outrigger-dimensions)

## Dimension Table Structure

すべてのディメンションテーブルには、単一の主キー列があります。この主キーは、関連するファクトテーブルに外部キーとして埋め込まれ、そのディメンション行の記述的なコンテキストが、そのファクトテーブル行に対して正確に一致するようになっています。ディメンションテーブルは通常、幅広く、フラットで正規化されていないテーブルであり、低いカーディナリティ（値の種類が少ない）のテキスト属性を多く持っています。運用上のコードやインジケーターも属性として扱うことができますが、最も効果的なディメンション属性は、詳細な説明で埋められたものです。ディメンションテーブルの属性は、クエリや BI（ビジネスインテリジェンス）アプリケーションからの制約やグループ化指定の主な対象となります。レポート上の説明ラベルは、通常、ディメンション属性のドメイン値に基づいています。

> Every dimension table has a single primary key column. This primary key is embedded as a foreign key in any associated fact table where the dimension row’s descriptive context is exactly correct for that fact table row. Dimension tables are usually wide, ﬂat denormalized tables with many low-cardinality text attributes. While operational codes and indicators can be treated as attributes, the most powerful dimension attributes are populated with verbose descriptions. Dimension table attributes are the primary target of constraints and grouping specifications from queries and BI applications. The descriptive labels on reports are typically dimension attribute domain values.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-table-structure/

## Dimension Surrogate Keys

ディメンションテーブルは、1 つの列が一意の主キーとして機能するように設計されています。この主キーは、運用システムのナチュラルキー（自然キー）を使用することはできません。なぜなら、時間の経過に伴う変更を追跡する場合、1 つのナチュラルキーに対して複数のディメンション行が存在する可能性があるからです。さらに、ディメンションのナチュラルキーは複数のソースシステムによって生成される場合があり、それらナチュラルキーの互換性がなかったり、適切に管理されていなかったりすることもあります。そのため、データウェアハウス/BI（DW/BI）システムは、すべてのディメンションの主キーを管理する責任を負う必要があります。明示的なナチュラルキーや、ナチュラルキーに日付を付加したものを使用するのではなく、すべてのディメンションに対して匿名の整数型の主キー（サロゲートキー）を作成するべきです。これらのディメンション・サロゲートキーは単純な整数で、新しいキーが必要になるたびに 1 から始まる連番として割り当てられます。ただし、日付ディメンションはサロゲートキーのルールから除外されます。このディメンションは非常に予測可能で安定しているため、より意味のある主キーを使用することができます。

> A dimension table is designed with one column serving as a unique primary key. This primary key cannot be the operational system’s natural key because there will be multiple dimension rows for that natural key when changes are tracked over time. In addition, natural keys for a dimension may be created by more than one source system, and these natural keys may be incompatible or poorly administered. The DW/BI system needs to claim control of the primary keys of all dimensions; rather than using explicit natural keys or natural keys with appended dates, you should create anonymous integer primary keys for every dimension. These dimension surrogate keys are simple integers, assigned in sequence, starting with the value 1, every time a new key is needed. The date dimension is exempt from the surrogate key rule; this highly predictable and stable dimension can use a more meaningful primary key.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-surrogate-key/

## Natural, durable and supernatural keys

運用ソースシステムによって作成されたナチュラルキー（自然キー）は、DW/BI システムの管理外にあるビジネスルールに従います。

例えば、従業員番号（ナチュラルキー）は、従業員が退職し、その後再雇用された場合に変更されることがあります。しかし、データウェアハウスがその従業員に対して一意のキーを持ちたい場合、このような状況でも変更されない永続的な新しいキーを作成する必要があります。このキーは「耐久性のある超自然キー（durable supernatural key）」と呼ばれることがあります。最良な耐久性のあるキーは、元のビジネスプロセスに依存しない形式を持ち、したがって 1 から始まる連番として割り当てられる単純な整数であるべきです。従業員のプロファイルが時間とともに変化するにつれて、複数のサロゲートキーがその従業員に関連付けられる可能性がありますが、耐久性のあるキーは決して変更されません。

> Natural keys created by operational source systems are subject to business rules outside the control of the DW/BI system. For instance, an employee number (natural key) may be changed if the employee resigns and then is rehired. When the data warehouse wants to have a single key for that employee, a new durable key must be created that is persistent and does not change in this situation. This key is sometimes referred to as a durable supernatural key. The best durable keys have a format that is independent of the original business process and thus should be simple integers assigned in sequence beginning with 1. While multiple surrogate keys may be associated with an employee over time as their profile changes, the durable key never changes.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/natural-durable-supernatural-key/

## Drilling Down

ドリルダウンは、ビジネスユーザーがデータを分析する際の最も基本的な方法です。ドリルダウンとは、単純に既存のクエリに行ヘッダーを追加することを意味します。この新しい行ヘッダーは、SQL クエリの GROUP BY 句に追加されるディメンション属性です。この属性は、クエリ内でファクトテーブルに関連付けられた任意のディメンションから取得することができます。ドリルダウンを行うため、あらかじめ定義された階層やドリルダウンパスを設定する必要はありません。

> Drilling down is the most fundamental way data is analyzed by business users. Drilling down simply means adding a row header to an existing query; the new row header is a dimension attribute appended to the GROUP BY expression in an SQL query. The attribute can come from any dimension attached to the fact table in the query. Drilling down does not require the definition of predetermined hierarchies or drill-down paths.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/drilling-down/

## Degenerate dimensions

時には、主キー以外に何の内容も持たないディメンションが定義されることがあります。

例えば、請求書に複数の明細行がある場合、明細行のファクト行は請求書のすべての記述的なディメンション外部キーを引き継ぎ、請求書自体には固有の内容が残らないことがあります。しかし、請求書番号は明細行レベルのファクトテーブルにおいて有効なディメンションキーとして機能します。

このような退行ディメンション（degenerate dimension）は、関連するディメンションテーブルが存在しないことを明示的に認識した上で、ファクトテーブルに配置されます。退行ディメンションは、特にトランザクションファクトテーブルや蓄積スナップショットファクトテーブルで最も一般的です。

> Sometimes a dimension is defined that has no content except for its primary key. For example, when an invoice has multiple line items, the line item fact rows inherit all the descriptive dimension foreign keys of the invoice, and the invoice is left with no unique content. But the invoice number remains a valid dimension key for fact tables at the line item level. This degenerate dimension is placed in the fact table with the explicit acknowledgment that there is no associated dimension table. Degenerate dimensions are most common with transaction and accumulating snapshot fact tables.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/degenerate-dimension/

## Denormalized flattened dimensions

一般的に、ディメンショナルデザインを行う際には、長年の運用データベース設計によって培われた正規化の衝動を抑え、代わりに多対一の固定深度の階層を持つフラットなディメンション行である個別の属性として、非正規化する必要があります。ディメンションの非正規化は、dimensional modeling の 2 つの主要な目的である「シンプルさ」と「高速性」をサポートします。

> In general, dimensional designers must resist the normalization urges caused by years of operational database designs and instead denormalize the many-to-one fixed depth hierarchies into separate attributes on a flattened dimension row. Dimension denormalization supports dimensional modeling’s twin objectives of simplicity and speed.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/denormalized-flattened-dimension/

## Multiple Hierarchies in Dimensions

多くのディメンションは、複数のナチュラル階層を含んでいます。例えば、カレンダー日付ディメンションには、「日 -> 週 -> 会計期間」の階層や、「日 -> 月 -> 年」の階層が存在する場合があります。また、場所に関連するディメンションでは、複数の地理的階層を持つことがあります。これらすべての場合において、異なる階層は同じディメンションテーブル内で問題なく共存することができます。

> Many dimensions contain more than one natural hierarchy. For example, calendar date dimensions may have a day to week to fiscal period hierarchy, as well as a day to month to year hierarchy. Location intensive dimensions may have multiple geographic hierarchies. In all of these cases, the separate hierarchies can gracefully coexist in the same dimension table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/multiple-hierarchies/

## Flags and Indicators as Textual Dimension Attributes

難解な略語、真偽フラグ、運用指標などは、ディメンションテーブル内を独立して見たときに意味の分かる完全なテキスト形式の言葉で補完されるべきです。コード値の中に意味が埋め込まれている運用コードは、そのコードの各部分を分解し、それぞれを独自の記述的なディメンション属性として展開する必要があります。

> Cryptic abbreviations, true/false flags, and operational indicators should be supplemented in dimension tables with full text words that have meaning when independently viewed. Operational codes with embedded meaning within the code value should be broken down with each part of the code expanded into its own separate descriptive dimension attribute.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/flag-indicator-attribute/

## Null Attributes in Dimensions

ディメンション属性に null 値が発生するのは、特定のディメンション行が完全に埋められていない場合や、すべてのディメンション行に適用されない属性が存在する場合です。これらのケースでは、null 値の代わりに `Unknown（不明）` や `Not Applicable（該当なし）` といった説明的な文字列を代用することを推奨します。ディメンション属性に null 値を含めることは避けるべきです。なぜなら、異なるデータベースが null 値に対するグループ化や制約の処理を一貫して行わない場合があるからです。

> Null-valued dimension attributes result when a given dimension row has not been fully populated, or when there are attributes that are not applicable to all the dimension’s rows. In both cases, we recommend substituting a descriptive string, such as Unknown or Not Applicable in place of the null value. Nulls in dimension attributes should be avoided because different databases handle grouping and constraining on nulls inconsistently.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/null-dimension-attribute/

## Calendar Date Dimensions

カレンダー日付ディメンションは、事実上すべてのファクトテーブルに接続され、ファクトテーブルを馴染みのある日付、月、会計期間、カレンダー上の特別な日を通じてナビゲートできるようにします。例えば、イースターの日付を SQL で計算するのではなく、カレンダー日付ディメンションから参照する方が望ましいです。カレンダー日付ディメンションには通常、`週番号`、`月名`、`会計期間`、`国民の祝日` などの特性を記述する多くの属性が含まれています。

パーティショニングを容易にするために、日付ディメンションの主キーは、連番で割り当てられるサロゲートキーではなく、YYYYMMDD を表す整数のような、より意味のある形式にすることができます。ただし、日付ディメンションテーブルには、不明または未確定の日付を表す特別な行が必要です。スマートキー（日付キー）を使用する場合、フィルタリングやグループ化はスマートキーではなく、ディメンションテーブルの属性に基づいて行うべきです。

さらに精度が必要な場合、ファクトテーブルに独立した日付/タイムスタンプ列を追加することができます。この日付/タイムスタンプはディメンションテーブルへの外部キーではなく、独立した列として扱われます。もしビジネスユーザーが、時間帯のグループ化やシフト番号など、時刻に関連する属性で制約やグループ化を行う場合は、ファクトテーブルに別途「時刻ディメンション」の外部キーを追加する必要があります。

> Calendar date dimensions are attached to virtually every fact table to allow navigation of the fact table through familiar dates, months, fiscal periods, and special days on the calendar. You would never want to compute Easter in SQL, but rather want to look it up in the calendar date dimension. The calendar date dimension typically has many attributes describing characteristics such as week number, month name, fiscal period, and national holiday indicator. To facilitate partitioning, the primary key of a date dimension can be more meaningful, such as an integer representing YYYYMMDD, instead of a sequentially-assigned surrogate key. However, the date dimension table needs a special row to represent unknown or to-be-determined dates. If a smart date key is used, filtering and grouping should be based on the dimension table’s attributes, not the smart key.

> When further precision is needed, a separate date/time stamp can be added to the fact table. The date/time stamp is not a foreign key to a dimension table, but rather is a standalone column. If business users constrain or group on time-of-day attributes, such as day part grouping or shift number, then you would add a separate time-of-day dimension foreign key to the fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/calendar-date-dimension/

## Role-Playing Dimensions

1 つの物理的なディメンションは、ファクトテーブル内で複数回参照されることあります。それぞれの参照は、そのディメンションに対して論理的に異なる役割（ロール）となります。

例えば、ファクトテーブルには複数の日付が含まれる場合があり、それぞれが日付ディメンションへの外部キーとして表されます。このとき、各外部キーが日付ディメンションの独立したビューを参照することが重要です。これにより、各参照が互いに独立したものとなります。これらの独立したディメンション・ビュー（ユニークな属性列名を持つもの）は「ロール」と呼ばれます。

> A single physical dimension can be referenced multiple times in a fact table, with each reference linking to a logically distinct role for the dimension. For instance, a fact table can have several dates, each of which is represented by a foreign key to the date dimension. It is essential that each foreign key refers to a separate view of the date dimension so that the references are independent. These separate dimension views (with unique attribute column names) are called roles.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/role-playing-dimension/

## Junk Dimensions

トランザクション型のビジネスプロセスは、様々で雑多な低カーディナリティのフラグやインジケーターを生成することがよくあります。これらのフラグや属性ごとに個別のディメンションを作成するのではなく、それらを 1 つにまとめたジャンク・ディメンションを作成することができます。このディメンションは、スキーマ内で `トランザクション・プロファイル・ディメンション` とラベル付けされることが多いです。

ジャンク・ディメンションを作成する際、すべての属性の可能な値の直積（デカルト積）を含める必要はありません。代わりに、ソースデータ内で実際に発生する値の組み合わせだけを含めるようにするべきです。これにより、ディメンションのサイズを効率的に管理し、不要な組み合わせを排除することができます。

> Transactional business processes typically produce a number of miscellaneous, low-cardinality flags and indicators. Rather than making separate dimensions for each ﬂag and attribute, you can create a single junk dimension combining them together. This dimension, frequently labeled as a transaction profile dimension in a schema, does not need to be the Cartesian product of all the attributes’ possible values, but should only contain the combination of values that actually occur in the source data.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/junk-dimension/

## Snowflaked Dimensions

ディメンションテーブル内の階層的な関係が正規化されると、低カーディナリティ（値の種類が少ない）の属性が、属性キーを介して基底のディメンションテーブルに接続された二次テーブルとして現れます。このプロセスをディメンションテーブル内のすべての階層に対して繰り返すと、特徴的な多層構造が作られ、これを「スノーフレーク」と呼びます。スノーフレークは階層データを正確に表現しますが、スノーフレークは避けるべきです。なぜなら、スノーフレークはビジネスユーザーにとって理解しづらく、ナビゲートが難しいからです。また、クエリのパフォーマンスにも悪影響を与える可能性があります。

一方で、フラット化され非正規化されたディメンションテーブルは、スノーフレーク化されたディメンションとまったく同じ情報を含んでいます。そのため、スノーフレークを使用する必要はなく、非正規化された構造の方が実用的で効果的です。

> When a hierarchical relationship in a dimension table is normalized, low-cardinality attributes appear as secondary tables connected to the base dimension table by an attribute key. When this process is repeated with all the dimension table’s hierarchies, a characteristic multilevel structure is created that is called a snowﬂake. Although the snowﬂake represents hierarchical data accurately, you should avoid snowﬂakes because it is difficult for business users to understand and navigate snowﬂakes. They can also negatively impact query performance. A flattened denormalized dimension table contains exactly the same information as a snowﬂaked dimension.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/snowflake-dimension/

## Outrigger Dimensions

ディメンションは、別のディメンションテーブルを参照することができます。

例えば、銀行口座のディメンションが、口座が開設された日付を表す別のディメンションを参照する場合があります。このような二次的なディメンション参照は `アウトリガー・ディメンション (outrigger dimension)` と呼ばれます。

アウトリガー・ディメンションの使用は許容されますが、慎重に、そして必要最小限にとどめるべきです。ほとんどの場合、ディメンション間の相関関係はファクトテーブルに降格させるべきです。つまり、両方のディメンションをファクトテーブル内で個別の外部キーとして表現する方が適切です。これにより、データモデルがシンプルになり、クエリのパフォーマンスやメンテナンス性が向上します。

> A dimension can contain a reference to another dimension table. For instance, a bank account dimension can reference a separate dimension representing the date the account was opened. These secondary dimension references are called outrigger dimensions. Outrigger dimensions are permissible, but should be used sparingly. In most cases, the correlations between dimensions should be demoted to a fact table, where both dimensions are represented as separate foreign keys.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/outrigger-dimension/
