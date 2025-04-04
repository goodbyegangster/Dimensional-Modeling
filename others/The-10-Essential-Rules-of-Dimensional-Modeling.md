# The 10 Essential Rules of Dimensional Modeling

Kimball Group のディメンショナルモデリング講座に参加したある学生から、「Kimball の戒律（Commandments）」のリストを教えてほしいと頼まれました。宗教的な表現は避けますが、ここでは絶対に破ってはいけないルールと、推奨される設計の指針を紹介します。

## 詳細な原子レベルのデータをディメンショナル構造に格納する

ディメンショナルモデルには、最も詳細な原子レベルのデータを格納するべきです。ビジネスユーザーのクエリでは、予測できないフィルタリングやグルーピングが求められるため、詳細データが必要になります。

通常、ユーザーは 1 件ずつのレコードを閲覧するわけではありませんが、どのような形でデータを集約・フィルタリングしたいかは予測できません。もし集計済みのデータしか格納しなかった場合、ユーザーが詳細を掘り下げたいときに行き詰まってしまいます。

もちろん、詳細データに加えて、集計済みのディメンショナルモデルを補助的に用意することで、一般的なクエリのパフォーマンスを向上させることは可能です。しかし、ビジネスユーザーは集計データだけでは満足できません。常に詳細データが必要になります。

## ディメンショナルモデルはビジネスプロセスを中心に構築する

ビジネスプロセスとは、組織が実行する活動のことであり、注文の受付や請求処理などの測定イベントを指します。 これらのプロセスは、各イベントに関連するパフォーマンス指標（メトリクス）を記録します。これらのメトリクスはファクト（事実）として表現され、各ビジネスプロセスごとに 1 つの原子レベルのファクトテーブルを作成するのが基本です。

また、複数のビジネスプロセスのメトリクスを統合した「統合ファクトテーブル」を作成することもあります。ただし、統合ファクトテーブルは、詳細な単一プロセスのファクトテーブルを補完するものであり、代替するものではありません。

## すべてのファクトテーブルには、日付ディメンションを関連付ける

ルール #2 で述べた測定イベントには、必ず何らかの日付情報が含まれます。例えば、月次の残高スナップショットや、秒単位で記録される資金移動などです。

したがって、すべてのファクトテーブルには、少なくとも 1 つの外部キーとして日付ディメンションを持たせるべきです。日付ディメンションの粒度は 1 日を単位とし、カレンダー属性や、会計月・企業の休日フラグなどの非標準的な日付情報を含めます。

また、ファクトテーブルには複数の日付キーが含まれることもあります（例: 受注日、出荷日、請求日など）。

## すべてのファクトテーブル内のファクトは、同じ粒度（グレイン）であること

ファクトテーブルには、トランザクション型、定期スナップショット型、累積スナップショット型の 3 つの基本的な粒度（グレイン）があります。どの粒度を選択する場合でも、ファクトテーブル内のすべての測定値（ファクト）は、同じ詳細レベルでなければなりません。異なる粒度のファクトを混在させると、ビジネスユーザーが混乱し、BI アプリケーションの結果が誤ったものになるリスクが高まります。

## ファクトテーブル内の多対多（N:N）関係を適切に解決する

ファクトテーブルは、ビジネスプロセスのイベント結果を格納するため、外部キー同士の間には本質的に多対多（M:M）の関係が存在します。例えば、複数の商品が、複数の店舗で、複数の日に販売されるといったケースです。このような場合、外部キーのフィールドは決して NULL にしてはいけません。

また、1 つの測定イベントに対して、複数のディメンション値が関連付く場合（例: 1 つの診療に対して複数の診断がある、1 つの銀行口座に複数の顧客が紐づく）は、多対多のブリッジテーブルを使用して解決する必要があります。

## ディメンションテーブル内の多対一（N:1）関係を適切に解決する

ディメンションテーブル内の階層的な多対一（N:1）関係は、通常、正規化せずに 1 つのディメンションテーブルに統合（フラット化）します。トランザクション処理システムの設計に慣れている人は、N:1 の関係を正規化してスノーフレーク構造にしたくなるかもしれませんが、ディメンショナルモデリングでは非推奨です。

ただし、ディメンションテーブルの行数が数百万件に及び、属性の変更が頻繁に発生する場合は、例外的にファクトテーブル側で N:1 の関係を解決することもあります。

## レポートのラベルやフィルタリング用の値は、ディメンションテーブルに格納する

コードやラベル、フィルタリングに使用する値は、ディメンションテーブルに格納するべきです。また、ファクトテーブル内に暗号化されたコードや長い説明文を格納するのは避けるべきです。

## ディメンションテーブルには、必ずサロゲートキーを使用する

ディメンションテーブルの主キーには、業務システムのキーではなく、意味を持たないサロゲートキーを使用するべきです。

サロゲートキーを使用することで、以下のメリットがあります。

- ファクトテーブルのサイズが小さくなり、パフォーマンスが向上する
- 業務システムの変更（例: 顧客 ID の再利用）からデータウェアハウスを保護できる
- ディメンション属性の変更を追跡しやすくなる

## 適合ディメンション（Conformed Dimension）を作成し、データを統合する

適合ディメンション（Conformed Dimension）は、エンタープライズデータウェアハウス（EDW）において、データの一貫性を確保するために不可欠です。

## ビジネス要件とデータの現実をバランスよく考慮し、実用的な DW/BI ソリューションを提供する

ディメンショナルモデリングでは、ビジネスユーザーの要件と、実際のデータの制約をバランスよく考慮することが重要です。

このルールブックを活用し、優れたディメンショナルモデルを設計してください！

[The 10 Essential Rules of Dimensional Modeling](https://www.kimballgroup.com/2009/05/the-10-essential-rules-of-dimensional-modeling/)
