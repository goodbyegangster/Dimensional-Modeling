# Fundamental Concepts <!-- omit in toc -->

- [Gather business requirements and data realities](#gather-business-requirements-and-data-realities)
- [Collaborative Dimensional Modeling Workshops](#collaborative-dimensional-modeling-workshops)
- [Four-Step Dimensional Design Process](#four-step-dimensional-design-process)
- [Business Processes](#business-processes)
- [Grain](#grain)
- [Dimensions for Descriptive Context](#dimensions-for-descriptive-context)
- [Star Schemas and OLAP Cubes](#star-schemas-and-olap-cubes)
- [Grace Extensions to Dimensional Modeling](#grace-extensions-to-dimensional-modeling)

## Gather business requirements and data realities

dimensional modeling の取り組みを開始する前に、チームはビジネスの要求と、基盤となる source data の現実を理解する必要があります。ビジネス代表者との対話を通じて、主要業績指標（KPI）、重要なビジネス課題、意思決定プロセス、そしてそれを支える分析要求に基づいた目標を理解することで、要件を明らかにします。同時に、source 側システムの専門家との対話や、高レベルのデータプロファイリングを行うことで、データの実現可能性を評価し、データの現実を明らかにします。

> Before launching a dimensional modeling effort, the team needs to understand the needs of the business, as well as the realities of the underlying source data. You uncover the requirements via sessions with business representatives to understand their objectives based on key performance indicators, compelling business issues, decision-making processes, and supporting analytic needs. At the same time, data realities are uncovered by meeting with source system experts and doing high-level data profiling to assess data feasibilities.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/business-requirements-data-realities/

## Collaborative Dimensional Modeling Workshops

dimensional modeling は、業務の専門家や data governance の代表者と協力して設計されるべきです。モデル設計者が主導しますが、モデルは業務側代表者との高度に双方向ワークショップを通じて展開されるべきです。これらのワークショップは、業務とともに要件を具体化するためのもう一つの機会を提供します。dimensional modeling は、業務やその要求を十分に理解していない人々が孤立して設計するべきではありません。協力が重要です！

> Dimensional models should be designed in collaboration with subject matter experts and data governance representatives from the business. The data modeler is in charge, but the model should unfold via a series of highly interactive workshops with business representatives. These workshops provide another opportunity to flesh out the requirements with the business. Dimensional models should not be designed in isolation by folks who don’t fully understand the business and their needs; collaboration is critical!

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/collaborative-modeling-workshop/

## Four-Step Dimensional Design Process

dimensional modeling の設計において行われる 4 つの重要な決定事項は以下の通りです。

1. ビジネスプロセスを選択する
2. 粒度（グレイン）を定義する
3. dimension を特定する
4. fact を特定する

これらの問いへの答えは、共同モデリングセッションの中で、ビジネスの要求と基盤となる source data の現実を考慮しながら決定されます。ビジネスプロセス、粒度、dimension、fact の定義が完了した後、設計チームはテーブル名やカラム名、サンプルのドメイン値、ビジネスルールを決定します。この詳細な設計作業には、ビジネス側の data governance 代表者が参加し、ビジネス側の合意を確保する必要があります。

> The four key decisions made during the design of a dimensional model include:

> 1. Select the business process.

> 2. Declare the grain.

> 3. Identify the dimensions.

> 4. Identify the facts.

> The answers to these questions are determined by considering the needs of the business along with the realities of the underlying source data during the collaborative modeling sessions. Following the business process, grain, dimension, and fact declarations, the design team determines the table and column names, sample domain values, and business rules. Business data governance representatives must participate in this detailed design activity to ensure business buy-in.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/four-4-step-design-process/

## Business Processes

ビジネスプロセスとは、組織が実行する業務活動のことで、例えば注文の受注、保険請求の処理、クラスへの学生登録、または月別の全ユーザーのスナップショット作成などが挙げられます。ビジネスプロセスのイベントは、パフォーマンス指標を生成または記録し、それがファクトテーブル内のファクト（事実）に変換されます。ほとんどのファクトテーブルは、単一のビジネスプロセスの結果に焦点を当てています。

プロセスを選択することは重要です。なぜなら、それが特定の設計目標を定義し、粒度（グレイン）、dimension、fact を宣言することを可能にするからです。各ビジネスプロセスは、enterprise data warehouse の bus matrix における 1 行に対応します。

> Business processes are the operational activities performed by your organization, such as taking an order, processing an insurance claim, registering students for a class, or snapshotting every account each month. Business process events generate or capture performance metrics that translate into facts in a fact table. Most fact tables focus on the results of a single business process. Choosing the process is important because it defines a specific design target and allows the grain, dimensions, and facts to be declared. Each business process corresponds to a row in the enterprise data warehouse bus matrix.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/business-process/

## Grain

`粒度（grain）` を定義することは、dimensional modeling における最も重要なステップです。粒度は、ファクトテーブルの 1 行が正確に何を表しているのかを確立します。粒度の定義は設計上の契約のようなものであり、dimension や fact を選択する前に必ず定義されなければなりません。なぜなら、すべての候補となる dimension や fact は粒度と一貫性がなければならないからです。この一貫性は、BI アプリケーションのパフォーマンスや使いやすさにとって重要な、すべての dimension modeling における統一性を保証します。

`アトミック粒度（Atomic Grain）` とは、特定のビジネスプロセスによって記録されるデータの、最も低いレベルを指します。私たちは、まずアトミック粒度のデータに焦点を当てることを強く推奨します。なぜなら、アトミック粒度のデータは予測不可能なユーザーのクエリにも耐えられるからです。一方で、集計されたサマリー粒度はパフォーマンスチューニングにおいて重要ですが、それはビジネスの一般的な質問を前提としています。

提案された各ファクトテーブルの粒度は、個別の物理テーブルを生み出します。異なる粒度を同じファクトテーブル内で混在させてはなりません。

> Declaring the grain is the pivotal step in a dimensional design. The grain establishes exactly what a single fact table row represents. The grain declaration becomes a binding contract on the design. The grain must be declared before choosing dimensions or facts because every candidate dimension or fact must be consistent with the grain. This consistency enforces a uniformity on all dimensional designs that is critical to BI application performance and ease of use. Atomic grain refers to the lowest level at which data is captured by a given business process. We strongly encourage you to start by focusing on atomic-grained data because it withstands the assault of unpredictable user queries; rolled-up summary grains are important for performance tuning, but they pre-suppose the business’s common questions. Each proposed fact table grain results in a separate physical table; different grains must not be mixed in the same fact table.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/grain/

## Dimensions for Descriptive Context

`次元（dimension）`は、ビジネスプロセスのイベントを取り巻く `who`, `what`, `where`, `when`, `why`, そして `how` といった文脈を提供します。ディメンションテーブルには、BI アプリケーションがファクトをフィルタリングやグループ化する際に使用する記述的な属性が含まれています。ファクトテーブルの粒度を明確に把握した上で、すべての可能なディメンションを特定することができます。可能であれば、1 つのディメンションは特定のファクト行に関連付けられた単一値であるべきです。

ディメンションテーブルは、データウェアハウスの「魂」と呼ばれることもあります。なぜなら、ディメンションテーブルには、DW/BI システムをビジネス分析に活用するためのエントリーポイントや記述ラベルが含まれているからです。ディメンションテーブルは、ユーザーの BI 体験を左右する要素であるため、そのデータガバナンスや開発には不釣り合いなほど多くの労力が注がれます。

> Dimensions provide the “who, what, where, when, why, and how” context surrounding a business process event. Dimension tables contain the descriptive attributes used by BI applications for filtering and grouping the facts. With the grain of a fact table firmly in mind, all the possible dimensions can be identified. Whenever possible, a dimension should be single valued when associated with a given fact row.

> Dimension tables are sometimes called the “soul” of the data warehouse because they contain the entry points and descriptive labels that enable the DW/BI system to be leveraged for business analysis. A disproportionate amount of effort is put into the data governance and development of dimension tables because they are the drivers of the user’s BI experience.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/dimensions-for-context/

## Star Schemas and OLAP Cubes

dimensional model は、プロセスの測定イベントに焦点を当て、データを測定値または `who`, `what`, `where`, `when`, `why`, そして `how` といった文脈に分けて扱います。

dimensional model は、リレーショナルデータベース（スター・スキーマと呼ばれる）や多次元データベース（オンライン分析処理、OLAP キューブとして知られる）の両方で実装することができます。スター・スキーマは、通常、ファクトテーブルが主キー/外部キーの関係を介して関連する次元テーブルとリンクされて構成されます。一方、OLAP キューブは、リレーショナルなスター・スキーマと同等の内容を持つ場合もあれば、より頻繁にはスター・スキーマから派生したものです。OLAP キューブには次元属性とファクトが含まれていますが、SQL よりも分析機能が豊富な言語（例えば XMLA）を介してアクセスされます。OLAP キューブは、次元型 DW/BI システムの最終的な展開ステップとして、またはよりアトミックなリレーショナルスター・スキーマに基づく集計構造として存在することが多いため、基本的な技術のリストに含まれています。

「キンボール（Kimball）」という言葉は、dimensional modeling と同義語とされています。ラルフ・キンボール（Ralph Kimball）は、ファクトと次元の基本概念そのものを発明したわけではありませんが、適合ディメンション（conformed dimensions）、緩やかに変化する次元（SCD）、ジャンク・ディメンション（junk dimensions）、ミニ・ディメンション（mini-dimensions）、ブリッジテーブル（bridge tables）、定期スナップショット・ファクトテーブルや累積スナップショット・ファクトテーブルなど、dimensional modeling の技法や用語の広範なポートフォリオを確立しました。この 30 年近くの間に、Ralph と彼の Kimball Group の同僚たちは、dimensional modeling に関する数百もの記事や設計のヒントを執筆し、また、代表的な書籍[『The Data Warehouse Toolkit, Third Edition』](http://www.kimballgroup.com/data-warehouse-business-intelligence-resources/books/data-warehouse-dw-toolkit/)（Wiley, 2013）を出版しました。

Ralph が先導した一方で、dimensional modeling は Kimball アーキテクチャを採用する組織だけでなく、ビル・インモン（Bill Inmon）らが提唱する Corporate Information Factory（CIF）のハブ＆スポークアーキテクチャを採用する組織にも適しています。dimensional modeling のベストプラクティスは、アーキテクチャに依存しない中立的なものです。

dimensional modeling の概要を簡単に知りたい場合は、以下の一連の記事から始めることをお勧めします。詳細については、[『The Data Warehouse Toolkit, Third Edition』](http://www.kimballgroup.com/data-warehouse-business-intelligence-resources/books/data-warehouse-dw-toolkit/)に完全な内容が記載されています。

- “[A Dimensional Modeling Manifesto](http://www.kimballgroup.com/1997/08/02/a-dimensional-modeling-manifesto/)”, DBMS, August 1997

- “[Fact Tables and Dimension Tables](http://www.kimballgroup.com/2003/01/01/fact-tables-and-dimension-tables/)”, Intelligent Enterprise, January, 2003

- “[Dividing the World](http://www.kimballgroup.com/2008/02/20/dividing-the-world/)”, Data Management Review, March 2008

- “[Essential Steps for the Integrated Enterprise Data Warehouse, Part 1](http://www.kimballgroup.com/2008/03/17/essential-steps-for-the-integrated-enterprise-data-warehouse-part-1/)”, Data Management Review, April 2008

- “[Essential Steps for the Integrated Enterprise Data Warehouse, Part 2](http://www.kimballgroup.com/2008/04/11/essential-steps-for-the-integrated-enterprise-data-warehouse-part-2/)”, Data Management Review, May 2008

- “[Kimball’s Ten Essential Rules of Dimensional Modeling](http://www.kimballgroup.com/2009/05/29/the-10-essential-rules-of-dimensional-modeling/)“, Intelligent Enterprise, May 2009

> Dimensional models focus on process measurement events, dividing data into either measurements or the “who, what, where, when, why, and how” descriptive context.

> Dimensional models can be instantiated in both relational databases, referred to as star schemas, or multidimensional databases, known as online analytical processing (OLAP) cubes. Star schemas characteristically consist of fact tables linked to associated dimension tables via primary/foreign key relationships. OLAP cubes can be equivalent in content to, or more often derived from, a relational star schema. An OLAP cube contains dimensional attributes and facts, but it is accessed via languages with more analytic capabilities than SQL, such as XMLA. OLAP cubes are included in this list of basic techniques because a cube is often the final deployment step of a dimensional DW/BI system, or may exist as an aggregate structure based on a more atomic relational star schema.

> The word “Kimball” is synonymous with dimensional modeling. Ralph didn’t invent the original basic concepts of facts and dimensions, however, he established an extensive portfolio of dimensional techniques and vocabulary, including conformed dimensions, slowly changing dimensions, junk dimensions, mini-dimensions, bridge tables, periodic and accumulating snapshot fact tables, and the list goes on. Over the past nearly 30 years, Ralph and his Kimball Group colleagues have written hundreds of articles and Design Tips on dimensional modeling, as well as the seminal text, The Data Warehouse Toolkit, Third Edition (Wiley, 2013).

> While Ralph led the charge, dimensional modeling is appropriate for organizations who embrace the Kimball architecture, as well as those who follow the Corporate Information Factory (CIF) hub-and-spoke architecture espoused by Bill Inmon and others. Dimensional modeling best practices are architecture-neutral.

> For a brief overview of dimensional modeling, we suggest starting with the following series of articles. Full coverage is available in The Data Warehouse Toolkit, Third Edition.

> ・“A Dimensional Modeling Manifesto”, DBMS, August 1997

> ・“Fact Tables and Dimension Tables”, Intelligent Enterprise, January, 2003

> ・“Dividing the World”, Data Management Review, March 2008

> ・“Essential Steps for the Integrated Enterprise Data Warehouse, Part 1”, Data Management Review, April 2008

> ・“Essential Steps for the Integrated Enterprise Data Warehouse, Part 2”, Data Management Review, May 2008

> ・“Kimball’s Ten Essential Rules of Dimensional Modeling“, Intelligent Enterprise, May 2009

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/star-schema-olap-cube/

## Grace Extensions to Dimensional Modeling

dimensional model は、データの関係が変化しても柔軟に対応できる設計となっています。以下のような変更を、既存の BI クエリやアプリケーションを変更することなく、またクエリ結果に影響を与えることなく実装することが可能です。

- 既存のファクトテーブルの粒度と一致するファクトを、新しいカラムを作成することで追加できます。

- ディメンションがファクトテーブルの粒度を変更しない限り、新しい外部キー（foreign key）カラムを作成することで、既存のファクトテーブルにディメンションを追加できます。

- 新しいカラムを作成することで、既存のディメンションテーブルに属性を追加できます。

- ファクトテーブルの粒度をよりアトミック（細かい粒度）にするために、既存のディメンションテーブルに属性を追加し、その後、ファクトテーブルをより低い粒度で再定義することができます。この際、ファクトテーブルおよびディメンションテーブル内の既存のカラム名を保持するよう注意が必要です。

> Dimensional models are resilient when data relationships change. All the following changes can be implemented without altering any existing BI query or application, and without any change in query results.

> ・Facts consistent with the grain of an existing fact table can be added by creating new columns.

> ・Dimensions can be added to an existing fact table by creating new foreign key columns, presuming they don’t alter the fact table’s grain.

> ・Attributes can be added to an existing dimension table by creating new columns.

> ・The grain of a fact table can be made more atomic by adding attributes to an existing dimension table, and then restating the fact table at the lower grain, being careful to preserve the existing column names in the fact and dimension tables.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/extensions/
