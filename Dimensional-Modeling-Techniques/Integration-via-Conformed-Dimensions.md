# Integration via Conformed Dimensions <!-- omit in toc -->

- [Conformed Dimensions](#conformed-dimensions)
- [Shrunken Rollup Dimensions](#shrunken-rollup-dimensions)
- [Drilling Across](#drilling-across)
- [Value Chain](#value-chain)
- [Enterprise Data Warehouse Bus Architecture](#enterprise-data-warehouse-bus-architecture)
- [Enterprise Data Warehouse Bus Matrix](#enterprise-data-warehouse-bus-matrix)
- [Opportunity/Stakeholder Matrix](#opportunitystakeholder-matrix)

## Conformed Dimensions

適合ディメンションは、異なるディメンション間の属性で、同じ列名や同じドメイン内容をもつ場合に「準拠」しているとみなします。適合ディメンションの属性を使用することで、異なるファクトテーブルから 1 つのレポートに統合することが可能になります。例えば、適合ディメンションの属性が行ヘッダー（つまり、SQL クエリにおける group by 句の列）として使用される場合、異なるファクトテーブルの結果をドリル・アクロス・レポートで同じ行に揃えることができます。これが、エンタープライズのデータウェアハウス（DW）/ビジネスインテリジェンス（BI）システムにおける統合の本質です。

ビジネスのデータ・ガバナンス担当者と協力して定義できれば、適合ディメンションは複数のファクトテーブルで再利用できます。同じ仕組みを何度も再構築する必要がなくなるため、分析の一貫性が確保されるだけでなく、将来的な開発コストも削減されます。

> Dimension tables conform when attributes in separate dimension tables have the same column names and domain contents. Information from separate fact tables can be combined in a single report by using conformed dimension attributes that are associated with each fact table. When a conformed attribute is used as the row header (that is, the grouping column in the SQL query), the results from the separate fact tables can be aligned on the same rows in a drill-across report. This is the essence of integration in an enterprise DW/ BI system. Conformed dimensions, defined once in collaboration with the business’s data governance representatives, are reused across fact tables; they deliver both analytic consistency and reduced future development costs because the wheel is not repeatedly re-created.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/conformed-dimension/

## Shrunken Rollup Dimensions

`シュランケン・ディメンション（Shrunken dimensions）` は、ベースとなる適合ディメンションの行や列の一部を抜き出したサブセットです。

シュランケン・ロールアップ・ディメンションは、集計ファクトテーブルを構築する際に必要となります。また、月ごとの予測やブランドごとの予測（販売データに関連するより詳細な日付や製品ではなく）といった、より粗い粒度でデータを自然に収集するビジネスプロセスにおいても必要です。

また、適合ディメンションのサブセットが発生するもう一つのケースとして、2 つのディメンションが同じ詳細レベルにあるものの、一方のみが行の一部を表している場合があります。

> Shrunken dimensions are conformed dimensions that are a subset of rows and /or columns of a base dimension. Shrunken rollup dimensions are required when constructing aggregate fact tables. They are also necessary for business processes that naturally capture data at a higher level of granularity, such as a forecast by month and brand (instead of the more atomic date and product associated with sales data). Another case of conformed dimension subsetting occurs when two dimensions are at the same level of detail, but one represents only a subset of rows.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/shrunken-rollup-dimension/

## Drilling Across

`ドリルアクロス（Drilling Across）` とは、単純に言えば、2 つ以上のファクトテーブルに対して個別にクエリを実行し、それぞれクエリの行ヘッダー（つまり、SQL クエリにおける group by 句の列）が同一の適合属性で構成されていることを指します。2 つのクエリから得られた結果セットは、共通のディメンション属性の行ヘッダーに基づいてソート・マージ操作を行うことができます。BI ツールのベンダーは、この機能を「ステッチ」や「マルチパスクエリ」など、さまざまな名称で呼んでいます。

> Drilling across simply means making separate queries against two or more fact tables where the row headers of each query consist of identical conformed attributes. The answer sets from the two queries are aligned by performing a sort-merge operation on the common dimension attribute row headers. BI tool vendors refer to this functionality by various names, including stitch and multipass query.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/drilling-across/

## Value Chain

`バリューチェーン（Value chain）` は、組織の主要なビジネスプロセスの、自然な流れを特定するものです。

例えば、小売業者のバリューチェーンは、「仕入れ」から「倉庫管理」、そして「小売販売」へと続く流れで構成されるかもしれません。一方、一般会計のバリューチェーンは、「予算編成」から「契約」、そして「支払い」へと進む流れで構成される場合があります。

運用系のソースシステムは、通常、バリューチェーンの各ステップでトランザクションやスナップショットを生成します。各プロセスは、`固有の指標` を `固有の時間間隔` で、`固有の粒度` と `固有の次元構造` で生成するため、通常、各プロセスは少なくとも 1 つのアトミックファクトテーブル（最も詳細なレベルのファクトテーブル）を生み出します。

> A value chain identifies the natural ﬂow of an organization’s primary business processes. For example, a retailer’s value chain may consist of purchasing to ware- housing to retail sales. A general ledger value chain may consist of budgeting to commitments to payments. Operational source systems typically produce transactions or snapshots at each step of the value chain. Because each process produces unique metrics at unique time intervals with unique granularity and dimensionality, each process typically spawns at least one atomic fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/value-chain/

## Enterprise Data Warehouse Bus Architecture

`enterprise data warehouse bus architecture` は、エンタープライズ DW/BI システムを段階的に構築するためのアプローチを提供します。このアーキテクチャは、ビジネスプロセスに焦点を当てることで、DW/BI の計画プロセスを管理可能な単位に分解し、プロセス間で再利用可能な、標準化された適合ディメンションを通じて統合を実現します。

バスアーキテクチャは、全体的なアーキテクチャの枠組みを提供すると同時に、enterprise data warehouse bus matrix の行に対応する形で、管理しやすいアジャイル実装を行える単位で作業範囲を分解します。このアーキテクチャは、特定の技術やデータベースプラットフォームに依存せず、リレーショナル構造と OLAP 次元構造の両方が参加可能です。

> The enterprise data warehouse bus architecture provides an incremental approach to building the enterprise DW/BI system. This architecture decomposes the DW/ BI planning process into manageable pieces by focusing on business processes, while delivering integration via standardized conformed dimensions that are reused across processes. It provides an architectural framework, while also decomposing the program to encourage manageable agile implementations corresponding to the rows on the enterprise data warehouse bus matrix. The bus architecture is technology and database platform independent; both relational and OLAP dimensional structures can participate.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/enterprise-data-warehouse-bus-architecture/

## Enterprise Data Warehouse Bus Matrix

`enterprise data warehouse bus matrix` は、enterprise data warehouse bus architecture を設計し、伝達するための重要なツールです。このマトリックスの行はビジネスプロセスを表し、列はディメンション（次元）を表します。マトリックスの塗りつぶされたセルは、特定のビジネスプロセスにディメンションが関連付けられていることを示します。

設計チームは、各行をスキャンして、候補となるディメンションがそのビジネスプロセスに対して適切に定義されているかを確認します。また、各列をスキャンして、ディメンションが複数のビジネスプロセス間で適合（準拠）すべき箇所を特定します。技術的な設計上考慮事項に加えて、バスマトリックスは、DW/BI プロジェクトの優先順位をビジネス側と決定するためにも使用されます。チームは、マトリックスの 1 行を一度に実装する形で進めるべきです。

`詳細実装バスマトリックス（detailed implementation bus matrix）` は、より細かい粒度で表現されたバスマトリックスであり、各ビジネスプロセスの行が展開され、具体的なファクトテーブルや OLAP キューブが示されます。この詳細レベルでは、正確な粒度の定義やファクトのリストを文書化することが可能です。

> The enterprise data warehouse bus matrix is the essential tool for designing and communicating the enterprise data warehouse bus architecture. The rows of the matrix are business processes and the columns are dimensions. The shaded cells of the matrix indicate whether a dimension is associated with a given business process. The design team scans each row to test whether a candidate dimension is well-defined for the business process and also scans each column to see where a dimension should be conformed across multiple business processes. Besides the technical design considerations, the bus matrix is used as input to prioritize DW/BI projects with business management as teams should implement one row of the matrix at a time.

> The detailed implementation bus matrix is a more granular bus matrix where each business process row has been expanded to show specific fact tables or OLAP cubes. At this level of detail, the precise grain statement and list of facts can be documented.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/enterprise-data-warehouse-bus-matrix/

## Opportunity/Stakeholder Matrix

enterprise data warehouse bus matrix の行が特定された後、ディメンション列をマーケティング、セールス、ファイナンスといったビジネス機能に置き換えた別のマトリックスを作成することができます。そして、マトリックスのセルを塗りつぶして、どのビジネス機能がどのビジネスプロセスの行に関心を持っているかを示します。この `機会/ステークホルダーマトリックス（opportunity/stakeholder matrix）` は、各ビジネスプロセスの行に対するセッションに、どのビジネスグループを招待すべきかに役立ちます。

> After the enterprise data warehouse bus matrix rows have been identified, you can draft a different matrix by replacing the dimension columns with business functions, such as marketing, sales, and finance, and then shading the matrix cells to indicate which business functions are interested in which business process rows. The opportunity/stakeholder matrix helps identify which business groups should be invited to the collaborative design sessions for each process-centric row.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/opportunity-stakeholder-matrix/
