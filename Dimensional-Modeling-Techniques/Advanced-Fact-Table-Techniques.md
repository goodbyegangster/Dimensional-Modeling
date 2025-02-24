# Advanced Fact Table Techniques

## Fact Table Surrogate Keys

サロゲートキーは、ほぼすべてのディメンション・テーブルの主キーを実装するために使用されます。加えて、単一列のサロゲート・ファクトキーも有用ですが、必須ではありません。ファクトテーブルのサロゲートキーは、どのディメンションにも関連付けられておらず、ETL やロードプロセス中に順次割り当てられます。このキーは以下の目的で使用されます

- 1. ファクトテーブルの単一列の主キーとして機能する
- 2. ETL の目的で、複数のディメンションとのナビゲートを必要とせず、ファクトテーブル行を即座に識別する
- 3. 中断されたロードプロセスを取り消したり再開したりする
- 4. ファクトテーブルの更新操作をリスクの少ない挿入と削除に分解する

> Surrogate keys are used to implement the primary keys of almost all dimension tables. In addition, single column surrogate fact keys can be useful, albeit not required. Fact table surrogate keys, which are not associated with any dimension, are assigned sequentially during the ETL load process and are used 1) as the single column primary key of the fact table; 2) to serve as an immediate identifier of a fact table row without navigating multiple dimensions for ETL purposes; 3) to allow an interrupted load process to either back out or resume; 4) to allow fact table update operations to be decomposed into less risky inserts plus deletes.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/fact-surrogate-key/

## Centipede Fact Tables

一部の設計者は、多対一階層の各レベル（例：日付ディメンション、月ディメンション、四半期ディメンション、年ディメンション）に対して個別の正規化されたディメンションを作成し、それらすべての外部キーをファクトテーブルに含めます。この結果、階層的に関連するディメンションが多数存在する `ムカデ型ファクトテーブル` が生じます。ムカデ型ファクトテーブルは避けるべきです。

これらの固定深度で多対一階層となるディメンションは、例として挙げた日付のように、ユニークな最小粒度に統合されるべきです。また、ムカデ型ファクトテーブルは、設計者が低カーディナリティの個別のディメンション・テーブルに多数の外部キーを埋め込む代わりに、ジャンク・ディメンションを作成しない場合にも発生します。

> Some designers create separate normalized dimensions for each level of a many-to- one hierarchy, such as a date dimension, month dimension, quarter dimension, and year dimension, and then include all these foreign keys in a fact table. This results in a centipede fact table with dozens of hierarchically related dimensions. Centipede fact tables should be avoided. All these fixed depth, many-to-one hierarchically related dimensions should be collapsed back to their unique lowest grains, such as the date for the example mentioned. Centipede fact tables also result when designers embed numerous foreign keys to individual low-cardinality dimension tables rather than creating a junk dimension.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/centipede-fact-table/

## Numeric Values as Attributes or Facts

設計者は、数値がファクトまたはディメンション属性のどちらに分類されるか、明確でないケースに直面することがあります。典型的な例として、製品の標準リスト価格があります。数値が主に計算目的で使用される場合、それはファクトテーブルに属する可能性が高いです。一方、安定した数値が主にフィルタリングやグループ化に使用される場合、それはディメンション属性として扱うべきです。この場合、離散的な数値に値帯域属性（例：$0-50）を補足することができます。場合によっては、数値をファクトとディメンション属性の両方としてモデル化することが有用です。例えば、定量的なオンタイム配達メトリックと定性的なテキスト記述子の両方を持つ場合です。

> Designers sometimes encounter numeric values that don’t clearly fall into either the fact or dimension attribute categories. A classic example is a product’s standard list price. If the numeric value is used primarily for calculation purposes, it likely belongs in the fact table. If a stable numeric value is used predominantly for filtering and grouping, it should be treated as a dimension attribute; the discrete numeric values can be supplemented with value band attributes (such as $0-50). In some cases, it is useful to model the numeric value as both a fact and dimension attribute, such as a quantitative on-time delivery metric and qualitative textual descriptor.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/numeric%20-attribute-fact/

## Lag/Duration Facts

蓄積スナップショット・ファクトテーブルは、複数のプロセス・マイルストーンをキャプチャします。それぞれに日付の外部キーや、場合によってはタイムスタンプが含まれます。ビジネスユーザーは、これらのマイルストーン間の遅延や期間を分析したいと考えることがよくあります。これらの遅延は単に日付の差である場合もあれば、より複雑なビジネスルールに基づく場合もあります。パイプラインに数十のステップがある場合、数百の遅延が存在する可能性があります。タイムスタンプや日付ディメンション外部キーを基に遅延を計算する代わり、プロセスの開始点に対して測定された各ステップの遅延を 1 つだけファクトテーブルに格納します。その後、2 つのステップ間の遅延は、ファクトテーブルに格納された 2 つの遅延の単純な引き算として計算できます。

> Accumulating snapshot fact tables capture multiple process milestones, each with a date foreign key and possibly a date/time stamp. Business users often want to analyze the lags or durations between these milestones; sometimes these lags are just the differences between dates, but other times the lags are based on more complicated business rules. If there are dozens of steps in a pipeline, there could be hundreds of possible lags. Rather than forcing the user’s query to calculate each possible lag from the date/time stamps or date dimension foreign keys, just one time lag can be stored for each step measured against the process’s start point. Then every possible lag between two steps can be calculated as a simple subtraction between the two lags stored in the fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/lag-duration-fact/

## Header/Line Fact Tables

オペレーショントランザクションシステムは、複数のトランザクションラインに関連付けられたトランザクションヘッダー行で構成されることがよくあります。ヘッダー/ラインスキーマ（親/子スキーマとも呼ばれる）では、すべてのヘッダーレベルのディメンション外部キーと退化ディメンションをラインレベルのファクトテーブルに含めるべきです。

> Operational transaction systems often consist of a transaction header row that’s associated with multiple transaction lines. With header/line schemas (also known as parent/child schemas), all the header-level dimension foreign keys and degenerate dimensions should be included on the line-level fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/header-line-fact-table/

## Allocated Facts

ヘッダー/ラインのトランザクションデータでは、異なる粒度のファクト（例：ヘッダーの運送料）に遭遇することがよくあります。割り当てられたファクトをすべてのディメンションでスライスおよびロールアップできるようにするため、ビジネスルールに基づいてヘッダーファクトをラインレベルに割り当てるべきです。多くの場合、ヘッダーレベルのファクトテーブルを作成する必要はありません。ただし、この集計がクエリパフォーマンスの向上をもたらす場合を除きます。

> It is quite common in header/line transaction data to encounter facts of differing granularity, such as a header freight charge. You should strive to allocate the header facts down to the line level based on rules provided by the business, so the allocated facts can be sliced and rolled up by all the dimensions. In many cases, you can avoid creating a header-level fact table, unless this aggregation delivers query performance advantages.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/allocated-fact/

## Profit and Loss Fact Tables Using Allocations

損益の完全な方程式を明らかにするファクトテーブルは、エンタープライズ DW/BI システムの中で最も強力な成果物の 1 つです。損益の方程式は、(収益) - (コスト) = (利益) です。理想的には、ファクトテーブルは原子レベルな収益トランザクションの粒度を基に損益方程式を実装し、多くのコスト要素を含ませます。これらテーブルが原子粒度であるため、顧客収益性、製品収益性、プロモーション収益性、チャネル収益性など、さまざまなロールアップが可能です。しかし、これらのファクトテーブルを構築するのは困難です。なぜなら、コスト要素を元のソースからファクトテーブルの粒度に割り当てる必要があるからです。この割り当てステップは、しばしば主要な ETL サブシステムであり、政治的に敏感なステップであり、高レベルの経営陣の支援が必要です。このため、損益ファクトテーブルは、通常 DW/BI プログラムの初期実装段階では取り組まれません。

> Fact tables that expose the full equation of profit are among the most powerful deliverables of an enterprise DW/BI system. The equation of profit is (revenue) – (costs) = (profit). Fact tables ideally implement the profit equation at the grain of the atomic revenue transaction and contain many components of cost. Because these tables are at the atomic grain, numerous rollups are possible, including customer profitability, product profitability, promotion profitability, channel profitability, and others. However, these fact tables are difficult to build because the cost components must be allocated from their original sources to the fact table’s grain. This allocation step is often a major ETL subsystem and is a politically charged step that requires high- level executive support. For these reasons, proﬁt and loss fact tables are typically not tackled during the early implementation phases of a DW/BI program.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/profit-loss-fact-table/

## Multiple Currency Facts

複数通貨で財務トランザクションを記録するファクトテーブルには、行内のすべての財務ファクトに対して列のペアを含めるべきです。1 つの列には実際の通貨で表現されたトランザクションのファクトが含まれ、もう 1 つの列にはファクトテーブル全体で使用される単一の標準通貨で表現されたファクトが含まれます。標準通貨の値は、通貨変換のため承認されたビジネスルールに従った ETL プロセスが作成します。このファクトテーブルには、トランザクションでの実際の通貨を識別するため、通貨ディメンションも必要です。

> Fact tables that record financial transactions in multiple currencies should contain a pair of columns for every financial fact in the row. One column contains the fact expressed in the true currency of the transaction, and the other contains the same fact expressed in a single standard currency that is used throughout the fact table. The standard currency value is created in an ETL process according to an approved business rule for currency conversion. This fact table also must have a currency dimension to identify the transaction’s true currency.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/multiple-currencies/

## Year-to-Date Facts

ビジネスユーザーは、ファクトテーブルに年初来（YTD）値を要求することがよくあります。単一のリクエストに反論するのは難しいですが、YTD リクエストは簡単に「会計期間終了時の YTD」や「会計期間までの YTD」に変化する可能性があります。これらの多様なリクエストに対処するよりも信頼性が高くなる拡張方法は、YTD メトリクスをファクトテーブルに格納するのではなく、BI アプリケーションや OLAP キューブで計算することです。

> Business users often request year-to-date (YTD) values in a fact table. It is hard to argue against a single request, but YTD requests can easily morph into “YTD at the close of the fiscal period” or “fiscal period to date.” A more reliable, extensible way to handle these assorted requests is to calculate the YTD metrics in the BI applications or OLAP cube rather than storing YTD facts in the fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/year-to-date-fact/

## Multipass SQL to Avoid Fact-to-Fact Table Joins

BI アプリケーションは、ファクトテーブルの外部キーを介して 2 つのファクトテーブルを結合する SQL を発行してはなりません。このような結合の回答セットのカーディナリティをリレーショナルデータベースで制御することは不可能であり、BI ツールに誤った結果が返されます。例えば、2 つのファクトテーブルが顧客の製品出荷と返品を含む場合、これらのファクトテーブルを顧客および製品の外部キーを介して直接結合してはなりません。その代わりに、2 つのファクトテーブルをドリルアクロスする技法を使用します。この技法では、出荷と返品の回答セットを個別に作成し、共通の行ヘッダー属性値でソートマージして正しい結果を生成します。

> A BI application must never issue SQL that joins two fact tables together across the fact table’s foreign keys. It is impossible to control the cardinality of the answer set of such a join in a relational database, and incorrect results will be returned to the BI tool. For instance, if two fact tables contain customer’s product shipments and returns, these two fact tables must not be joined directly across the customer and product foreign keys. Instead, the technique of drilling across two fact tables should be used, where the answer sets from shipments and returns are separately created, and the results sort-merged on the common row header attribute values to produce the correct result.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/multipass-sql/

## Timespan Tracking in Fact Tables

ファクトテーブルには、トランザクション、定期スナップショット、蓄積スナップショットの 3 つの基本的な粒度があります。特定のケースでは、ファクトテーブルに行の有効開始日、行の有効終了日、現在の行インジケータを追加することが有用です。これは、SDC Type 2 で行うのと同様に、ファクト行が有効であった期間をキャプチャするためです。このパターンは珍しいですが、例えば、ゆっくり変化する在庫残高のようなシナリオに対応します。この場合、頻繁な定期スナップショットが各スナップショットで同一の行をロードすることになります。

> There are three basic fact table grains: transaction, periodic snapshot, and accumulating snapshot. In isolated cases, it is useful to add a row effective date, row expiration date, and current row indicator to the fact table, much like you do with type 2 slowly changing dimensions, to capture a timespan when the fact row was effective. Although an unusual pattern, this pattern addresses scenarios such as slowly changing inventory balances where a frequent periodic snapshot would load identical rows with each snapshot.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/timespan-fact-table/

## Late Arriving Facts

新しいファクト行の最新のディメンション・コンテキストが受信行と一致しない場合、そのファクト行は遅延到着と見なされます。これは、ファクト行が遅れて到着した場合に発生します。この場合、遅延到着の測定イベントが発生したときに有効であったディメンションキーを見つけるために、関連するディメンションを検索する必要があります。

> A fact row is late arriving if the most current dimensional context for new fact rows does not match the incoming row. This happens when the fact row is delayed. In this case, the relevant dimensions must be searched to ﬁnd the dimension keys that were effective when the late arriving measurement event occurred.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/late-arriving-fact/
