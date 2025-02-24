# Advanced Dimension Table Techniques

## Dimension-to-Dimension Table Joins

ディメンションは他のディメンションを参照することがあります。これらの関係はアウトリガーディメンション（outrigger dimensions）を用いてモデル化することができますが、場合によっては、基底ディメンション（base dimension）に、アウトリガーディメンションへの外部キーが存在することで、基底ディメンションが爆発的に増加する可能性があります。これは、アウトリガーディメンションでタイプ 2 の変更が発生すると、それに対応する基底ディメンションでもタイプ 2 の処理が必要になるためです。このような爆発的な増加は、ディメンション間の関連性を低下させることで回避できる場合があります。

具体的には、アウトリガーディメンションの外部キーを基底ディメンションではなく、ファクトテーブルに配置する方法です。この方法では、ディメンション間の関連性を発見するにはファクトテーブルを辿る必要がありますが、特にファクトテーブルが定期スナップショットであり、すべてのディメンションのキーが各報告期間に確実に存在する場合には、許容されることが多いです。

> Dimensions can contain references to other dimensions. Although these relationships can be modeled with outrigger dimensions, in some cases, the existence of a foreign key to the outrigger dimension in the base dimension can result in explosive growth of the base dimension because type 2 changes in the outrigger force corresponding type 2 processing in the base dimension. This explosive growth can often be avoided if you demote the correlation between dimensions by placing the foreign key of the outrigger in the fact table rather than in the base dimension. This means the correlation between the dimensions can be discovered only by traversing the fact table, but this may be acceptable, especially if the fact table is a periodic snapshot where all the keys for all the dimensions are guaranteed to be present for each reporting period.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimension-to-dimension-join/

## Multivalued Dimensions and Bridge Tables

古典的なディメンショナルスキーマでは、ファクトテーブルに接続される各ディメンションは、ファクトテーブルの粒度に一致する単一の値を持ちます。しかし、ディメンションが複数の値を持つ状況もいくつか存在します。例えば、医療治療を受ける患者が複数の診断を同時に受ける場合です。このような場合、多値ディメンションは、グループディメンションキーを介してブリッジテーブルに接続されます。このブリッジテーブルには、グループ内の各同時診断毎に 1 行が作成されます。

多値ブリッジテーブルは、SCD Type 2 に基づく場合があります。例えば、銀行口座と個々の顧客間の多対多の関係を実装するブリッジテーブルは、通常、タイプ 2 の口座ディメンションと顧客ディメンションに基づく必要があります。この場合、口座と顧客間の不正確なリンクを防ぐために、ブリッジテーブルには有効日と失効日（またはタイムスタンプ）を含める必要があります。また、リクエストするアプリケーションは、特定の時点に制約をかけて、整合性のあるスナップショットを生成する必要があります。

> In a classic dimensional schema, each dimension attached to a fact table has a single value consistent with the fact table’s grain. But there are a number of situations in which a dimension is legitimately multivalued. For example, a patient receiving a healthcare treatment may have multiple simultaneous diagnoses. In these cases, the multivalued dimension must be attached to the fact table through a group dimension key to a bridge table with one row for each simultaneous diagnosis in a group.

> A multivalued bridge table may need to be based on a type 2 slowly changing dimension. For example, the bridge table that implements the many-to-many relationship between bank accounts and individual customers usually must be based on type 2 account and customer dimensions. In this case, to prevent incorrect linkages between accounts and customers, the bridge table must include effective and expiration date/time stamps, and the requesting application must constrain the bridge table to a speciﬁc moment in time to produce a consistent snapshot.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/multivalued-dimension-bridge-table/

## Behavior Tag Time Series

データウェアハウス内のほぼすべてのテキストは、ディメンションテーブル内の記述的なテキストです。顧客クラスタ分析は、通常、定期的に識別されるテキスト形式の行動タグを生成します。この場合、顧客の行動測定値は、これら行動タグのシーケンスとなり、この時系列は顧客ディメンション内の位置属性として保存されるべきです。また、タグの完全なシーケンスを表すオプションのテキスト文字列も含めることができます。行動タグは、数値計算ではなく、複雑な同時クエリの対象となるため、位置設計でモデル化されます。

> Almost all text in a data warehouse is descriptive text in dimension tables. Data mining customer cluster analyses typically results in textual behavior tags, often identified on a periodic basis. In this case, the customers’ behavior measurements over time become a sequence of these behavior tags; this time series should be stored as positional attributes in the customer dimension, along with an optional text string for the complete sequence of tags. The behavior tags are modeled in a positional design because the behavior tags are the target of complex simultaneous queries rather than numeric computations.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/behavior-tag-series-attribute/

## Behavior Study Groups

複雑な顧客行動は、長時間の反復分析でのみ発見される場合があります。顧客ディメンション内すべての制約を考慮する BI アプリケーションに、このような複雑な行動分析を埋め込むのは非現実的です。しかし、複雑な行動分析の結果は、顧客の永続キー（durable key）のみで構成されるシンプルなテーブル、いわゆる「研究グループ（study group）」に保存することができます。この静的なテーブルは、クエリ時にターゲットスキーマ内の顧客ディメンション永続キーに study group 列を追加することで、顧客ディメンションの持つ任意のディメンションキーをフィルターとして使用できます。複数の study group を定義することができ、交差、和集合、差集合を用いて派生的な study group を作成することも可能です。

> Complex customer behavior can sometimes be discovered only by running lengthy iterative analyses. In these cases, it is impractical to embed the behavior analyses inside every BI application that wants to constrain all the members of the customer dimension who exhibit the complex behavior. The results of the complex behavior analyses, however, can be captured in a simple table, called a study group, consisting only of the customers’ durable keys. This static table can then be used as a kind of filter on any dimensional schema with a customer dimension by constraining the study group column to the customer dimension’s durable key in the target schema at query time. Multiple study groups can be defined and derivative study groups can be created with intersections, unions, and set differences.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/behavior-study-group/

## Aggregated Facts as Dimension Attributes

ビジネスユーザーは、集計されたパフォーマンス指標に基づいて顧客ディメンションを制約することに関心を持つことがあります。例えば、昨年や顧客の生涯において、一定の金額以上を消費したすべての顧客をフィルタリングする場合などです。選択された集計ファクトは、制約のターゲットやレポートの行ラベルとしてディメンションに配置することができます。これらの指標は、ディメンションテーブル内で帯域化された範囲として提示されることがよくあります。集計されたパフォーマンス指標を表すディメンション属性は、ETL 処理に負担をかけますが、BI レイヤーでの分析の負担を軽減します。

> Business users are often interested in constraining the customer dimension based on aggregated performance metrics, such as filtering on all customers who spent over a certain dollar amount during last year or perhaps over the customer’s lifetime. Selected aggregated facts can be placed in a dimension as targets for constraining and as row labels for reporting. The metrics are often presented as banded ranges in the dimension table. Dimension attributes representing aggregated performance metrics add burden to the ETL processing, but ease the analytic burden in the BI layer.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/aggregated-fact-attribute/

## Dynamic Value Banding

動的な値の帯域化レポートは、ターゲットとなる数値ファクトの可変サイズの範囲を段階的に定義する一連のレポート行ヘッダーとして構成されます。例えば、銀行でよく見られる値の帯域化レポートでは、「残高 0 ドルから 10 ドル」、「残高 10.01 ドルから 25 ドル」などのラベルが付いた多くの行があります。この種のレポートは動的であり、特定の行ヘッダーは ETL 処理中ではなく、クエリ時に定義されます。行の定義は、ファクトテーブルに対して「より大きい/より小さい」結合を使用する小さな値の帯域化ディメンションテーブルで実装することができます。または、SQL の CASE 文内にのみ存在させることも可能です。値の帯域化ディメンションを使用するアプローチは、特にカラム型データベースでは高いパフォーマンスを発揮する可能性があります。なぜなら、CASE 文アプローチでは、ファクトテーブルのほぼ無制約なリレーションスキャンが必要になるためです。

> A dynamic value banding report is organized as a series of report row headers that define a progressive set of varying-sized ranges of a target numeric fact. For instance, a common value banding report in a bank has many rows with labels such as “Balance from 0 to $10,” “Balance from $10.01 to $25,” and so on. This kind of report is dynamic because the specific row headers are defined at query time, not during the ETL processing. The row definitions can be implemented in a small value banding dimension table that is joined via greater-than/less-than joins to the fact table, or the definitions can exist only in an SQL CASE statement. The value banding dimension approach is probably higher performing, especially in a columnar database, because the CASE statement approach involves an almost unconstrained relation scan of the fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dynamic-value-banding/

## Text Comments

自由形式のコメントをファクトテーブル内のテキスト指標として扱うのではなく、ファクトテーブルの外部に、対応する外部キーを持つ別のコメントディメンションに保存するべきです。または、コメントのカーディナリティが一意のトランザクション数と一致する場合は、1 トランザクションあたり 1 行を持つディメンションの属性として保存することも可能です。

> Rather than treating freeform comments as textual metrics in a fact table, they should be stored outside the fact table in a separate comments dimension (or as attributes in a dimension with one row per transaction if the comments’ cardinality matches the number of unique transactions) with a corresponding foreign key in the fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/text-comment/

## Multiple Time Zones

複数のタイムゾーンを扱うアプリケーションで、世界標準時（UTC）とローカル時刻の両方を記録するには、影響を受けるファクトテーブルに 2 つの外部キーを配置し、それぞれ 2 つの役割を持つ日付（および必要に応じて時刻）ディメンションテーブルに結合する必要があります。

> To capture both universal standard time, as well as local times in multi-time zone applications, dual foreign keys should be placed in the affected fact tables that join to two role-playing date (and potentially time-of-day) dimension tables.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/multiple-time-zones/

## Measure Type Dimensions

ファクトテーブルに多数のファクトがあり、個々の行でまばらに埋められている場合、測定タイプディメンションを作成して、ファクトテーブルの行を単一の汎用ファクトに圧縮し、測定タイプディメンションで識別することを検討することがあります。しかし、私たちはこのアプローチを一般的には推奨しません。この方法は、すべての空のファクト列を削除する一方で、ファクトテーブルのサイズを各行で埋められている列の平均数だけ増加させ、列間の計算を非常に困難にします。この手法は、潜在的なファクトの数が極端に多い（数百に及ぶ）場合には許容されますが、特定のファクトテーブル行に適用されるファクトがほんの一握りである場合には推奨されません。

> Sometimes when a fact table has a long list of facts that is sparsely populated in any individual row, it is tempting to create a measure type dimension that collapses the fact table row down to a single generic fact identified by the measure type dimension. We generally do not recommend this approach. Although it removes all the empty fact columns, it multiplies the size of the fact table by the average number of occupied columns in each row, and it makes intra-column computations much more difficult. This technique is acceptable when the number of potential facts is extreme (in the hundreds), but less than a handful would be applicable to any given fact table row.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/measure-type-dimension/

## Step Dimensions

ウェブページイベントのような連続的なプロセスでは、通常、トランザクションファクトテーブル内の各ステップごとに別々の行が存在します。個々のステップが全体セッション内でどの位置にあるかを示すために、ステップディメンションが使用されます。このディメンションは、現在のステップがどのステップ番号を表しているか、そしてセッションを完了するためにあと何ステップ必要かを示します。

> Sequential processes, such as web page events, normally have a separate row in a transaction fact table for each step in a process. To tell where the individual step ﬁts into the overall session, a step dimension is used that shows what step number is represented by the current step and how many more steps were required to complete the session.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/step-dimension/

## Hot Swappable Dimensions

ホットスワップ可能なディメンションは、同じファクトテーブルが異なるコピーの同じディメンションと交互にペアリングされる場合に使用されます。例えば、株式ティッカーの見積もりを含む単一のファクトテーブルが、複数の個別の投資家に同時に公開される場合があります。それぞれの投資家には、異なる株式に割り当てられた独自の専有属性が存在します。

> Hot swappable dimensions are used when the same fact table is alternatively paired with different copies of the same dimension. For example, a single fact table containing stock ticker quotes could be simultaneously exposed to multiple separate investors, each of whom has unique and proprietary attributes assigned to different stocks.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/hot-swappable-dimension/

## Abstract Generic Dimensions

一部のモデラーは、抽象的な汎用ディメンションに魅力を感じることがあります。例えば、スキーマに店舗、倉庫、顧客ディメンションに埋め込まれた地理的属性の代わりに、単一の汎用的なロケーションディメンションを含める場合があります。同様に、従業員、顧客、ベンダーの連絡先を含む「人」ディメンションを作成することもあります。これらはすべて人間であるためです。しかし、それぞれのタイプに対して収集される属性が大きく異なる場合が多いため、抽象的な汎用ディメンションはディメンションモデルでは避けるべきです。もし共通の属性（例えば地理的な州）がある場合、それらは店舗の州と顧客の州を区別するために一意のラベルを付けるべきです。さらに、すべての種類のロケーション、人、または製品を単一のディメンションにまとめると、ディメンションテーブルが大きくなりがちです。データの抽象化は、運用ソースシステムや ETL 処理では適切かもしれませんが、ディメンションモデルではクエリのパフォーマンスや可読性に悪影響を与えます。

> Some modelers are attracted to abstract generic dimensions. For example, their schemas include a single generic location dimension rather than embedded geographic attributes in the store, warehouse, and customer dimensions. Similarly, their person dimension includes rows for employees, customers, and vendor contacts because they are all human beings, regardless that significantly different attributes are collected for each type. Abstract generic dimensions should be avoided in dimensional models. The attribute sets associated with each type often differ. If the attributes are common, such as a geographic state, then they should be uniquely labeled to distinguish a store’s state from a customer’s. Finally, dumping all varieties of locations, people, or products into a single dimension invariably results in a larger dimension table. Data abstraction may be appropriate in the operational source system or ETL processing, but it negatively impacts query performance and legibility in the dimensional model.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/abstract-generic-dimension/

## Audit Dimensions

ETL バックルームでファクトテーブルの行が作成される際、その時点で知られている ETL 処理メタデータを含む監査ディメンションを作成することが役立ちます。シンプルな監査ディメンションの行には、データ品質の基本的な指標が 1 つ以上含まれる場合があります。これらは、データ処理中に発生したデータ品質違反を記録するエラーイベントスキーマを調査して導き出されることがあります。他の有用な監査ディメンション属性としては、ファクト行を作成するために使用された ETL コードのバージョンや、ETL プロセスの実行タイムスタンプを記述する環境変数が含まれます。これらの環境変数は、特定の ETL ソフトウェアのバージョンでどの行が作成されたかを特定するために BI ツールがドリルダウンでき、コンプライアンスや監査の目的で特に有用です。

> When a fact table row is created in the ETL back room, it is helpful to create an audit dimension containing the ETL processing metadata known at the time. A simple audit dimension row could contain one or more basic indicators of data quality, perhaps derived from examining an error event schema that records data quality violations encountered while processing the data. Other useful audit dimension attributes could include environment variables describing the versions of ETL code used to create the fact rows or the ETL process execution time stamps. These environment variables are especially useful for compliance and auditing purposes because they enable BI tools to drill down to determine which rows were created with what versions of the ETL software.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/audit-dimension/

## Late Arriving Dimensions

運用ビジネスプロセスからのファクトが、関連するディメンションコンテキストよりも数分、数時間、数日、または数週間早く到着することがあります。例えば、リアルタイムデータ配信の状況では、特定の商品を購入する顧客の自然キーを示す在庫減少行が到着する場合があります。この場合、リアルタイム ETL システムでは、顧客や商品の識別がすぐに特定できなくても、この行を BI レイヤーに投稿する必要があります。このような場合、未解決の自然キーを属性として持つ特別なディメンション行が作成されます。当然ながら、これらのディメンション行には、ほとんどの記述列に汎用的な未知の値が含まれます。適切なディメンションコンテキストは、後でソースから提供されると想定されます。このディメンションコンテキストが最終的に提供されると、プレースホルダーのディメンション行はタイプ 1 の上書きで更新されます。また、遅延到着ディメンションデータは、タイプ 2 ディメンション属性に遡及的な変更が加えられる場合にも発生します。この場合、ディメンションテーブルに新しい行を挿入し、関連するファクト行を再記述する必要があります。

> Sometimes the facts from an operational business process arrive minutes, hours, days, or weeks before the associated dimension context. For example, in a real-time data delivery situation, an inventory depletion row may arrive showing the natural key of a customer committing to purchase a particular product. In a real-time ETL system, this row must be posted to the BI layer, even if the identity of the customer or product cannot be immediately determined. In these cases, special dimension rows are created with the unresolved natural keys as attributes. Of course, these dimension rows must contain generic unknown values for most of the descriptive columns; presumably the proper dimensional context will follow from the source at a later time. When this dimensional context is eventually supplied, the placeholder dimension rows are updated with type 1 overwrites. Late arriving dimension data also occurs when retroactive changes are made to type 2 dimension attributes. In this case, a new row needs to be inserted in the dimension table, and then the associated fact rows must be restated.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/late-arriving-dimension/
