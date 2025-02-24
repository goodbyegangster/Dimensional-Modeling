# Dimension Hierarchy Techniques <!-- omit in toc -->

- [Fixed Depth Positional Hierarchies](#fixed-depth-positional-hierarchies)
- [Slightly Ragged/Variable Depth Hierarchies](#slightly-raggedvariable-depth-hierarchies)
- [Ragged/Variable Depth Hierarchies](#raggedvariable-depth-hierarchies)

## Fixed Depth Positional Hierarchies

固定深度の階層は、製品からブランド、カテゴリ、部門といった「多対一の関係」の連続です。固定深度の階層が定義され、階層レベルで合意された名前がある場合、階層レベルはディメンジョンテーブル内で個別のポジショナル属性として表示されるべきです。

固定深度の階層は、上記の条件が満たされている限り、最も理解しやすくナビゲートしやすいものです。また、予測可能で高速なクエリパフォーマンスを提供します。一方で、階層が「多対一の関係」の連続ではない場合や、レベル数が変動してレベルに合意された名前がない場合は、不規則な階層技法を使用する必要があります。

> A fixed depth hierarchy is a series of many-to-one relationships, such as product to brand to category to department. When a fixed depth hierarchy is defined and the hierarchy levels have agreed upon names, the hierarchy levels should appear as separate positional attributes in a dimension table. A fixed depth hierarchy is by far the easiest to understand and navigate as long as the above criteria are met. It also delivers predictable and fast query performance. When the hierarchy is not a series of many-to-one relationships or the number of levels varies such that the levels do not have agreed upon names, a ragged hierarchy technique must be used.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/fixed-depth-hierarchy/

## Slightly Ragged/Variable Depth Hierarchies

やや不規則な階層は、固定されたレベル数を持たないものの、深さの範囲が小さいものです。例えば、地理的な階層は深さが 3 レベルから 6 レベル程度に収まることがよくあります。予測不可能な可変階層のための複雑な仕組みを使用する代わりに、やや不規則な階層を固定深度のポジショナル設計に強制的に適合させることができます。この場合、最大レベル数に対応する個別のディメンジョン属性を用意し、ビジネスルールに基づいて属性値を設定します。

> Slightly ragged hierarchies don’t have a fixed number of levels, but the range in depth is small. Geographic hierarchies often range in depth from perhaps three levels to six levels. Rather than using the complex machinery for unpredictably variable hierarchies, you can force-ﬁt slightly ragged hierarchies into a fixed depth positional design with separate dimension attributes for the maximum number of levels, and then populate the attribute value based on rules from the business.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/slightly-ragged-variable-depth-hierarchy/

## Ragged/Variable Depth Hierarchies

深さが不定の不規則な階層は、リレーショナルデータベースでモデリングおよびクエリを行うのが難しいです。SQL 拡張機能や OLAP アクセス言語は、再帰的な親/子関係をサポートする機能を提供していますが、これらのアプローチには制約があります。SQL 拡張機能では、クエリ時に代替の不規則な階層を置き換えることができず、共有所有構造がサポートされず、時間変化する不規則な階層もサポートされません。これらの問題は、特別に構築されたブリッジテーブルを使用して不規則な階層をモデリングすることで、リレーショナルデータベース内で克服することができます。このブリッジテーブルには、不規則な階層内のすべての可能なパスに対応する行が含まれており、特別な言語拡張を使用せずに標準 SQL で階層のすべての形式のトラバーサルを実現できます。

不規則な可変深度の階層に対するブリッジテーブルの使用は、ディメンション内にパスストリング属性を実装することで回避できます。ディメンションの各行に対して、パスストリング属性には、階層の最上位ノードからその行が記述するノードまでの完全なパスを含む特別にエンコードされたテキスト文字列が含まれます。その後、多くの標準的な階層分析リクエストは、SQL 言語拡張を使用せずに標準 SQL で処理できます。ただし、パスストリングアプローチでは、代替階層の迅速な置き換えや共有所有階層を実現することはできません。また、不規則な階層の構造変更により、階層全体を再ラベルする必要が生じる可能性があるため、このアプローチは脆弱性を持つ場合があります。

> Ragged hierarchies of indeterminate depth are difficult to model and query in a relational database. Although SQL extensions and OLAP access languages provide some support for recursive parent/child relationships, these approaches have limitations. With SQL extensions, alternative ragged hierarchies cannot be substituted at query time, shared ownership structures are not supported, and time varying ragged hierarchies are not supported. All these objections can be overcome in relational databases by modeling a ragged hierarchy with a specially constructed bridge table. This bridge table contains a row for every possible path in the ragged hierarchy and enables all forms of hierarchy traversal to be accomplished with standard SQL rather than using special language extensions.

> The use of a bridge table for ragged variable depth hierarchies can be avoided by implementing a pathstring attribute in the dimension. For each row in the dimension, the pathstring attribute contains a specially encoded text string containing the complete path description from the supreme node of a hierarchy down to the node described by the particular dimension row. Many of the standard hierarchy analysis requests can then be handled by standard SQL, without resorting to SQL language extensions. However, the pathstring approach does not enable rapid substitution of alternative hierarchies or shared ownership hierarchies. The pathstring approach may also be vulnerable to structure changes in the ragged hierarchy that could force the entire hierarchy to be relabeled.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/ragged-variable-depth-hierarchy/
