# Basic Fact Table Techniques <!-- omit in toc -->

- [Fact Table Structure](#fact-table-structure)
- [Additive, Semi-Additive, and Non-Additive Facts](#additive-semi-additive-and-non-additive-facts)
- [Nulls in Fact Tables](#nulls-in-fact-tables)
- [Transaction Fact Tables](#transaction-fact-tables)
- [Periodic Snapshot Fact Tables](#periodic-snapshot-fact-tables)
- [Accumulating Snapshot Fact Tables](#accumulating-snapshot-fact-tables)
- [Factless Fact Tables](#factless-fact-tables)
- [Aggregate Fact Tables or Cubes](#aggregate-fact-tables-or-cubes)
- [Consolidated Fact Tables](#consolidated-fact-tables)

## Fact Table Structure

ファクトテーブルは、現実世界での業務測定イベントによって生成される数値的な指標を含むものです。最も細かい粒度では、ファクトテーブルの 1 行は 1 つの測定イベントに対応し、その逆も同様です。そのため、ファクトテーブルの基本的な設計は、完全に物理的な活動に基づいており、最終的に作成されうるレポートには影響されません。

数値的な指標に加えて、ファクトテーブルには常に関連する各ディメンションの外部キーが含まれています。また、オプションとして degenerate ディメンションキーや日付/時間スタンプが含まれることもあります。ファクトテーブルは、クエリによって発生する計算や動的な集計の主要な対象となります。

> A fact table contains the numeric measures produced by an operational measurement event in the real world. At the lowest grain, a fact table row corresponds to a measurement event and vice versa. Thus the fundamental design of a fact table is entirely based on a physical activity and is not influenced by the eventual reports that may be produced. In addition to numeric measures, a fact table always contains foreign keys for each of its associated dimensions, as well as optional degenerate dimension keys and date/time stamps. Fact tables are the primary target of computations and dynamic aggregations arising from queries.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/fact-table-structure/

## Additive, Semi-Additive, and Non-Additive Facts

ファクトテーブルに含まれる数値的な指標は、3 つのカテゴリに分類されます。

最も柔軟で有用な指標は「完全加算可能（fully additive）」なものであり、これらの指標はファクトテーブルに関連付けられた任意のディメンションにわたって合計することができます。

「部分加算可能（semi-additive）」な指標は、いくつかのディメンションにわたって合計することはできますが、すべてのディメンションにわたって合計することはできません。例えば、残高（balance amounts）は一般的な部分加算可能な指標であり、時間以外のすべてのディメンションにわたって加算可能です。

最後に、「完全非加算（non-additive）」な指標も存在します。例えば、比率（ratios）は完全非加算の例です。非加算の指標に対する良いアプローチは、可能であれば、その非加算指標を構成する完全加算可能な要素を保存し、これらの要素を最終的な非加算指標を計算する前に合計することです。この最終的な計算は、BI（ビジネスインテリジェンス）レイヤーや OLAP キューブ内で行われることが多いです。

> The numeric measures in a fact table fall into three categories. The most flexible and useful facts are fully additive; additive measures can be summed across any of the dimensions associated with the fact table. Semi-additive measures can be summed across some dimensions, but not all; balance amounts are common semi-additive facts because they are additive across all dimensions except time. Finally, some measures are completely non-additive, such as ratios. A good approach for non-additive facts is, where possible, to store the fully additive components of the non-additive measure and sum these components into the final answer set before calculating the final non-additive fact. This final calculation is often done in the BI layer or OLAP cube.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/additive-semi-additive-non-additive-fact/

## Nulls in Fact Tables

ファクトテーブルにおける NULL 値は、問題なく扱うことができます。集計関数（SUM、COUNT、MIN、MAX、AVG）は、NULL 値を含むデータに対しても適切に動作します。

しかし、ファクトテーブルの外部キーに NULL 値を含めることは避けなければなりません。なぜなら、これらの NULL 値は参照整合性違反を自動的に引き起こすからです。NULL 値を外部キーで使用する代わりに、関連するディメンションテーブルには、未知（unknown）または該当なし（not applicable）の状態を表すデフォルト行（およびサロゲートキー）を用意する必要があります。

> Null-valued measurements behave gracefully in fact tables. The aggregate functions (SUM, COUNT, MIN, MAX, and AVG) all do the “right thing” with null facts. However, nulls must be avoided in the fact table’s foreign keys because these nulls would automatically cause a referential integrity violation. Rather than a null foreign key, the associated dimension table must have a default row (and surrogate key) representing the unknown or not applicable condition.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/fact-table-null/

## Transaction Fact Tables

`トランザクションファクトテーブル` の 1 行は、特定の時点と場所で発生した測定イベントに対応します。最も細かい粒度（アトミック粒度）のトランザクションファクトテーブルは、最も多次元的で表現力の高いファクトテーブルです。この強力な多次元性により、トランザクションデータを最大限に細分化（スライス＆ダイス）することが可能になります。

トランザクションファクトテーブルは、測定が行われた場合にのみ行が存在するため、密（dense）である場合もあれば、疎（sparse）である場合もあります。これらのファクトテーブルには、常に関連する各ディメンションの外部キーが含まれており、オプションとして正確なタイムスタンプや degenerate ディメンションキーが含まれることもあります。また、測定された数値的な指標は、トランザクションの粒度と一貫性がある必要があります。

> A row in a transaction fact table corresponds to a measurement event at a point in space and time. Atomic transaction grain fact tables are the most dimensional and expressive fact tables; this robust dimensionality enables the maximum slicing and dicing of transaction data. Transaction fact tables may be dense or sparse because rows exist only if measurements take place. These fact tables always contain a foreign key for each associated dimension, and optionally contain precise time stamps and degenerate dimension keys. The measured numeric facts must be consistent with the transaction grain.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/transaction-fact-table/

## Periodic Snapshot Fact Tables

`定期スナップショットファクトテーブル` の 1 行は、1 日、1 週間、1 か月といった標準的な期間内に発生した多くの測定イベントを要約したものです。この場合の粒度は個々のトランザクションではなく、期間そのものです。定期スナップショットファクトテーブルには、多くの指標が含まれることがよくあります。これは、ファクトテーブルの粒度に一致する測定イベントであれば、すべて許容されるためです。

これらのファクトテーブルは、外部キーに関して一様に密（dense）であるのが特徴です。なぜなら、その期間中に活動がまったくなかった場合でも、通常は各指標にゼロまたは NULL を含む行がファクトテーブルに挿入されるからです。

> A row in a periodic snapshot fact table summarizes many measurement events occurring over a standard period, such as a day, a week, or a month. The grain is the period, not the individual transaction. Periodic snapshot fact tables often contain many facts because any measurement event consistent with the fact table grain is permissible. These fact tables are uniformly dense in their foreign keys because even if no activity takes place during the period, a row is typically inserted in the fact table containing a zero or null for each fact.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/periodic-snapshot-fact-table/

## Accumulating Snapshot Fact Tables

`蓄積スナップショットファクトテーブル` の 1 行は、プロセスの開始から終了までのステップで発生する測定イベントを要約したものです。注文処理やクレーム処理のように、明確な開始点や標準的な中間点、終了点を持つパイプラインやワークフロープロセスは、この種類のファクトテーブルでモデル化することができます。

蓄積スナップショットファクトテーブルには、プロセス内の重要なマイルストーンごとに日付の外部キーが含まれます。たとえば、注文の 1 行（注文明細）に対応する蓄積スナップショットファクトテーブルのレコードは、注文明細が作成された時点で最初に挿入されます。その後、パイプラインの進捗に応じて、この蓄積ファクトテーブルの行が更新されます。このようにファクトテーブルの行を一貫して更新するプロセスは、3 種類のファクトテーブルの中で蓄積スナップショットファクトテーブル特有の特徴です。

さらに、重要な各ステップに関連する日付の外部キーに加えて、蓄積スナップショットファクトテーブルには他のディメンションの外部キーが含まれ、オプションとして degenerate ディメンションを含むこともあります。また、粒度に一致する数値的な遅延測定値や、マイルストーンの完了カウンターが含まれることが多いです。

> A row in an accumulating snapshot fact table summarizes the measurement events occurring at predictable steps between the beginning and the end of a process. Pipeline or workflow processes, such as order fulfillment or claim processing, that have a defined start point, standard intermediate steps, and defined end point can be modeled with this type of fact table. There is a date foreign key in the fact table for each critical milestone in the process. An individual row in an accumulating snapshot fact table, corresponding for instance to a line on an order, is initially inserted when the order line is created. As pipeline progress occurs, the accumulating fact table row is revisited and updated. This consistent updating of accumulating snapshot fact rows is unique among the three types of fact tables. In addition to the date foreign keys associated with each critical process step, accumulating snapshot fact tables contain foreign keys for other dimensions and optionally contain degenerate dimensions. They often include numeric lag measurements consistent with the grain, along with milestone completion counters.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/accumulating-snapshot-fact-table/

## Factless Fact Tables

ほとんどの測定イベントは数値的な結果を記録しますが、イベントが特定時点で複数ディメンション・エンティティに結びついたことを単に記録するだけの場合もあります。たとえば、特定の日に学生が授業に出席したというイベントには、数値的な指標は記録されていないかもしれません。しかし、カレンダーの日付、学生、教師、場所、授業といった外部キーを持つファクト行は明確に定義されています。同様に、顧客とのコミュニケーションもイベントですが、関連する数値的な指標が存在しない場合があります。

このような `ファクトレス・ファクトテーブル（factless fact table）` は、発生しなかった事象を分析するためにも使用できます。この種のクエリは常に 2 つの部分から成り立っています。1 つは、発生する可能性のあるすべてのイベントを含む `ファクトレス・カバレッジテーブル（factless coverage table）` であり、もう 1 つは実際に発生したイベントを含む `アクティビティ・テーブル（activity table）` です。アクティビティをカバレッジから差し引くことで、発生しなかったイベントの集合を得ることができます。

> Although most measurement events capture numerical results, it is possible that the event merely records a set of dimensional entities coming together at a moment in time. For example, an event of a student attending a class on a given day may not have a recorded numeric fact, but a fact row with foreign keys for calendar day, student, teacher, location, and class is well-defined. Likewise, customer communications are events, but there may be no associated metrics. Factless fact tables can also be used to analyze what didn’t happen. These queries always have two parts: a factless coverage table that contains all the possibilities of events that might happen and an activity table that contains the events that did happen. When the activity is subtracted from the coverage, the result is the set of events that did not happen.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/factless-fact-table/

## Aggregate Fact Tables or Cubes

`集約ファクトテーブル（Aggregate Fact Table）` は、クエリのパフォーマンスを向上させることを目的として、アトミックファクトテーブル（最も細かい粒度のファクトテーブル）のデータを単純に数値的に集約したものです。これらの集約ファクトテーブルは、アトミックファクトテーブルと同時に BI レイヤーで利用可能であるべきです。これにより、BI ツールのクエリで適切な集約レベルをスムーズに選択できるようになります。このプロセスは「集約ナビゲーション（Aggregate Navigation）」と呼ばれ、すべてのレポート作成者、クエリツール、BI アプリケーションが同じパフォーマンス向上の恩恵を受けられるよう、オープンな形で実行される必要があります。

適切に設計された集約セットは、データベースのインデックスのように機能します。つまり、クエリのパフォーマンスを向上させるものの、BI アプリケーションやビジネスユーザーが直接操作することはありません。集約ファクトテーブルには、`縮小された適合ディメンション（Shrunken Conformed Dimensions）` の外部キーと、よりアトミックなファクトテーブルの測定値を合計して作成された集約指標が含まれています。

最後に、要約された測定値を持つ集約 OLAP キューブは、リレーショナルな集約と同じ方法で構築されることが多いですが、OLAP キューブはビジネスユーザーが直接アクセスすることを目的としています。

> Aggregate fact tables are simple numeric rollups of atomic fact table data built solely to accelerate query performance. These aggregate fact tables should be available to the BI layer at the same time as the atomic fact tables so that BI tools smoothly choose the appropriate aggregate level at query time. This process, known as aggregate navigation, must be open so that every report writer, query tool, and BI application harvests the same performance benefits. A properly designed set of aggregates should behave like database indexes, which accelerate query performance but are not encountered directly by the BI applications or business users. Aggregate fact tables contain foreign keys to shrunken conformed dimensions, as well as aggregated facts created by summing measures from more atomic fact tables. Finally, aggregate OLAP cubes with summarized measures are frequently built in the same way as relational aggregates, but the OLAP cubes are meant to be accessed directly by the business users.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/aggregate-fact-table-cube/

## Consolidated Fact Tables

複数プロセスのファクトを、同じ粒度で表現できる場合には、1 つの統合ファクトテーブルにまとめることが便利な場合があります。たとえば、実際の売上データ（sales actuals）と売上予測データ（sales forecasts）を 1 つのファクトテーブルに統合することで、実績と予測を比較分析する作業が、別々のファクトテーブルを使用してドリルアクロスアプリケーションを構築する場合と比べて、簡単かつ高速になります。

統合ファクトテーブルは、ETL 処理に負担を増やす一方で、BI アプリケーションにおける分析の負担を軽減します。頻繁に一緒に分析されるクロス・プロセスの指標に対しては、統合ファクトテーブルの使用を検討すべきです。

> It is often convenient to combine facts from multiple processes together into a single consolidated fact table if they can be expressed at the same grain. For example, sales actuals can be consolidated with sales forecasts in a single fact table to make the task of analyzing actuals versus forecasts simple and fast, as compared to assembling a drill-across application using separate fact tables. Consolidated fact tables add burden to the ETL processing, but ease the analytic burden on the BI applications. They should be considered for cross-process metrics that are frequently analyzed together.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/consolidated-fact-table/
