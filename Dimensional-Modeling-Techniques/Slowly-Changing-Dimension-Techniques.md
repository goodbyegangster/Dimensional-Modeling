# Slowly Changing Dimension Techniques <!-- omit in toc -->

- [Type 0: Retain Original](#type-0-retain-original)
- [Type 1: Overwrite](#type-1-overwrite)
- [Type 2: Add New Row](#type-2-add-new-row)
- [Type 3: Add New Attribute](#type-3-add-new-attribute)
- [Type 4: Add Mini-Dimension](#type-4-add-mini-dimension)
- [other](#other)

## Type 0: Retain Original

SCD Type 0 では、ディメンション属性の値は決して変更されません。そのため、ファクトは常に元の値でグループ化されます。Type 0 は「元の値」とラベル付けできる属性、例えば「顧客の初期信用スコア」や「永続的な識別子」に適しています。また、日付ディメンションのほとんどの属性にも適用されます。

> With slowly changing dimension type 0, the dimension attribute value never changes, so facts are always grouped by this original value. Type 0 is appropriate for any attribute labeled “original,” such as a customer’s original credit score or a durable identifier. It also applies to most attributes in a date dimension.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-0/

## Type 1: Overwrite

SCD Type 1 では、ディメンション行の古い属性値が新しい値で上書きされます。Type 1 属性は常に最新の値を反映するため、この手法では履歴が失われます。このアプローチは実装が簡単で、追加のディメンション行を作成する必要がありません。しかしながら、この変更によって影響を受ける集計ファクトテーブルや OLAP キューブを再計算する必要がある点に注意が必要です。

> With slowly changing dimension type 1, the old attribute value in the dimension row is overwritten with the new value; type 1 attributes always reflects the most recent assignment, and therefore this technique destroys history. Although this approach is easy to implement and does not create additional dimension rows, you must be careful that aggregate fact tables and OLAP cubes affected by this change are recomputed.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-1/

## Type 2: Add New Row

SCD Type 2 では、更新された属性値を持つ新しい行が、ディメンションに追加されます。この場合、ディメンションの主キーを natural key や永続的なキーを超えて一般化する必要があります。なぜなら、各メンバーを記述する複数の行が存在する可能性があるからです。ディメンション・メンバーに新しい行が作成されると、新しい主キー（サロゲートキー）が割り当てられ、更新後から次の変更が発生して新しいディメンション・キーと更新されたディメンション行が作成されるまで、すべてのファクトテーブルで外部キーとして使用されます。

SCD Type 2 の変更には、ディメンション行に最低 3 つの追加列を追加する必要があります。

- 1. 行の有効開始日または日付/タイムスタンプ
- 2. 行の有効終了日または日付/タイムスタンプ
- 3. 現在の行であることを示すインジケータ。

> Slowly changing dimension type 2 changes add a new row in the dimension with the updated attribute values. This requires generalizing the primary key of the dimension beyond the natural or durable key because there will potentially be multiple rows describing each member. When a new row is created for a dimension member, a new primary surrogate key is assigned and used as a foreign key in all fact tables from the moment of the update until a subsequent change creates a new dimension key and updated dimension row.

> A minimum of three additional columns should be added to the dimension row with type 2 changes: 1) row effective date or date/time stamp; 2) row expiration date or date/time stamp; and 3) current row indicator.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-2/

## Type 3: Add New Attribute

SCD Type 3 では、古い属性値を保持するためにディメンションに新しい属性が追加されます。新しい値は SCD Type 1 の変更と同様にメイン属性を上書きします。このような SCD Type 3 の変更は、時に「代替現実」と呼ばれることがあります。ビジネスユーザーは、現在の値または代替現実のいずれかでファクトデータをグループ化およびフィルタリングすることができます。この技法は比較的使用頻度が低いです。

> Slowly changing dimension type 3 changes add a new attribute in the dimension to preserve the old attribute value; the new value overwrites the main attribute as in a type 1 change. This kind of type 3 change is sometimes called an alternate reality. A business user can group and filter fact data by either the current value or alternate reality. This slowly changing dimension technique is used relatively infrequently.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-3/

## Type 4: Add Mini-Dimension

SCD Type 4 は、ディメンション内の属性グループが急速に変化する場合に使用され、これらの属性がミニディメンションに分割されます。この状況は、時に「急速に変化するモンスター・ディメンション」と呼ばれることがあります。数百万行のディメンションテーブルで頻繁に使用される属性は、頻繁に変化しない場合でもミニディメンションの設計候補となります。SCD Type 4 のミニ・ディメンションには独自の主キーが必要であり、関連するファクトテーブルには、基底ディメンションとミニ・ディメンション両方の主キーが記録されます。

> Slowly changing dimension type 4 is used when a group of attributes in a dimension rapidly changes and is split off to a mini–dimension. This situation is sometimes called a rapidly changing monster dimension. Frequently used attributes in multimillion-row dimension tables are mini-dimension design candidates, even if they don’t frequently change. The type 4 mini-dimension requires its own unique primary key; the primary keys of both the base dimension and mini-dimension are captured in the associated fact tables.

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/type-4-mini-dimension/

## other

- [Design Tip #152 Slowly Changing Dimension Types 0, 4, 5, 6 and 7](https://www.kimballgroup.com/2013/02/design-tip-152-slowly-changing-dimension-types-0-4-5-6-7/)
