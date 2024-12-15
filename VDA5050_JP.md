![logo](./assets/logo.png)

# 無人搬送車（AGV）とマスターコントロール間の通信インターフェース

## VDA 5050

## Version 2.1.0

![制御システムと無人搬送車](./assets/csagv.png)



### 概要

無人搬送システム（DTS）の通信インターフェースの定義について。
この勧告では、中央のマスタ制御と自動誘導車両（AGV）の間で、物流プロセスにおける命令およびステータスデータを交換するための通信インターフェースについて説明しています。



### 免責事項
以下の説明は、自動誘導車両（AGV）とマスターコントロール間の通信インターフェースの実行に関する指針であり、誰に対しても自由に適用でき、拘束力のないものです。
適用する者は、特定のケースにおいて適切に適用されることを保証しなければなりません。

また、各問題提起の時点で主流となっている技術水準を考慮しなければならない。
提案を適用することで、各自の行動に対する責任を回避できるわけではない。
これらの声明は、包括的であることや、現行法の正確な解釈であることを主張するものではない。
関連する政策、法律、規制の研究に取って代わるものではない。
さらに、それぞれの製品の特別な機能や、異なる可能性のある用途も考慮しなければならない。
この点については、各自が自己責任で行動するものとします。
VDAおよび提案の開発または適用に関与する者の責任は免除されます。

提案の適用に誤りがある場合、または誤った解釈の可能性がある場合は、欠陥を修正できるよう、VDAに直ちに通知してください。

**発行者**
Verband der Automobilindustrie e.V. (VDA)
Behrenstraße 35, 10117 Berlin,
Germany
www.vda.de

**権利表示**
Association of the Automotive Industry (VDA)
複製およびその他のあらゆる形態の複製は、出所を明記する場合にのみ許可されます。

Version 2.1.0


## 目次

[1 巻頭言](#1-巻頭言)<br>
[2 本文書の目的](#2-本文書の目的)<br>
[3 本文書の範囲](#3-本文書の範囲)<br>
[3.1 その他の適用される文書](#31-その他の適用される文書)<br>
[4 要件とプロトコルの定義](#4-要件とプロトコルの定義)<br>
[5 コミュニケーションのプロセスと内容](#5-コミュニケーションのプロセスと内容)<br>
[6 プロトコル仕様](#6-プロトコル仕様)<br>
[6.1 表の記号と書式の意味](#61-表の記号と書式の意味)<br>
[6.1.1 オプション項目](#611-オプション項目)<br>
[6.1.2 許可された文字およびフィールド長](#612-許可された文字およびフィールド長)<br>
[6.1.3 フィールド、トピック、列挙の表記](#613-フィールド、トピック、列挙の表記) <br>
[6.1.4 JSON データ型](#614-json-データ型)<br>
[6.2 MQTT 接続の処理、セキュリティ、QoS](#62-mqtt-接続の処理、セキュリティ、QoS)<br>
[6.3 MQTT トピックレベル](#63-mqtt-トピックレベル)<br>
[6.4 プロトコルヘッダー](#64-プロトコルヘッダー)<br>
[6.5 通信トピック](#65-通信トピック)<br>
[6.6 トピック："order"（マスターコントロールからAGVへの）](#66-トピック：order（マスターコントロールからagvへの）)<br>
[6.6.1 概念とロジック](#661-概念とロジック)<br>
[6.6.2 注文と注文の更新](#662-注文と注文の更新)<br>
[6.6.3 注文キャンセル（マスターコントロールによる）](#663-注文キャンセル（マスターコントロールによる）)<br>
[6.6.4 注文の拒否](#664-注文の拒否)<br>
[6.6.5 Corridors](#665-corridors)<br>
[6.6.6 注文メッセージの実装](#666-注文メッセージの実装)<br>
[6.7 地図](#67-地図)<br>
[6.7.1 地図の配布](#671-地図の配布)<br>
[6.7.2 車両状態の地図](#672-車両状態の地図)<br>
[6.7.3 地図のダウンロード](#673-地図のダウンロード)<br>
[6.7.4 ダウンロードされた地図の有効化](#674-ダウンロードされた地図の有効化)<br>
[6.7.5 車両上地図の削除](#675-車両上地図の削除)<br>
[6.8 Actions](#68-actions)<br>
[6.8.1 Definition, parameters, effects and scope of predefined actions](#681-definition-parameters-effects-and-scope-of-predefined-actions)<br>
[6.8.2 States of predefined actions](#682-states-of-predefined-actions)<br>
[6.9 Topic: "instantActions" (from master control to AGV)](#69-topic-instantactions-from-master-control-to-agv)<br>
[6.10 Topic: "state" (from AGV to master control)](#610-topic-state-from-agv-to-master-control)<br>
[6.10.1 Concept and logic](#6101-concept-and-logic)<br>
[6.10.2 Traversal of nodes and entering/leaving edges, triggering of actions](#6102-traversal-of-nodes-and-enteringleaving-edges-triggering-of-actions)<br>
[6.10.3 Base request](#6103-base-request)<br>
[6.10.4 Information](#6104-information)<br>
[6.10.5 Errors](#6105-errors)<br>
[6.10.6 Implementation of the state message](#6106-implementation-of-the-state-message)<br>
[6.11 actionStates](#611-actionstates)<br>
[6.12 Action blocking types and sequence](#612-action-blocking-types-and-sequence)<br>
[6.13 Topic "visualization"](#613-topic-visualization)<br>
[6.14 Topic "connection"](#614-topic-connection)<br>
[6.15 Topic "factsheet"](#615-topic-factsheet)<br>
[6.15.1 Factsheet JSON structure](#6151-factsheet-json-structure)<br>
[7 ベストプラクティス](#7-ベストプラクティス)<br>
[7.1 エラー参照](#71-エラー参照)<br>
[7.2 パラメータのフォーマット](#72-パラメータのフォーマット)<br>
[8 用語集](#8-用語集)<br>
[8.1 定義](#81-定義)<br>


# 1 巻頭言

このインターフェースは、Verband der Automobilindustrie e.V. (VDA) とVerband Deutscher Maschinen- und Anlagenbau e. V. (VDMA) の協力により策定されました。
両者の目的は、汎用的に利用できるインターフェースを作成することです。
インターフェースの変更に関する提案はVDAに提出され、VDMAと共同で評価され、肯定的な判断が下された場合には新しいバージョンのステータスに採用されます。
GitHub経由で本ドキュメントに貢献していただいた場合は、大変感謝いたします。
リポジトリは次のリンクからご覧いただけます：https://github.com/vda5050/vda5050。


# 2 本文書の目的

この勧告の目的は、新しい車両を既存のマスターコントロールシステムに接続する作業を簡素化し、同一の作業環境において異なるメーカーのAGVや従来のシステム（在庫管理システム）との並行運転を可能にすることである。

マスタ制御とAGV間の統一インターフェースを定義する。
これは以下の点によって達成されるべきである。

- AGVとマスタ制御間の通信の標準を記述し、協調型搬送車両を使用した連続プロセスオートメーションへの搬送システムの統合の基礎とする。
車両の自律性、プロセスモジュールおよびインターフェースの向上、そしてできればイベント制御コマンドチェーンの厳格なシーケンスの分離などによる柔軟性の向上。
必要な情報（注文情報など）は中央サービスから提供され、一般的に有効であるため、「プラグ＆プレイ」機能が高く、実装時間の短縮が可能。労働安全衛生要件を考慮した実装作業は同じであるため、車両はメーカーに関係なく稼働できることが望ましい。
- すべての輸送車両、車両モデル、およびメーカーに対応するロジックを統一した包括的な調整を行うことにより、システムの複雑性を低減し、「プラグ・アンド・プレイ」機能を向上させる。
- 車両制御レベルと調整レベル間の共通インターフェースを使用することにより、メーカーの独立性を向上させる。
- 独自仕様のマスターコントロールと上位のマスターコントロール間の垂直通信を実装することにより、独自仕様のDTS在庫システムを統合する（図1参照）。

![Figure 1 Integration of DTS inventory systems](./assets/concept_DTS.png)
>図1 DTSの在庫管理システムの統合

上記の目的を達成するために、本書では、AGVとマスターコントロール間の注文およびステータス情報の通信インターフェースについて説明する。

AGVとマスターコントロール間の動作（経路計画などにおいて特別なスキルを自由に考慮するなど）や、他のシステムコンポーネント（外部周辺機器、防火ゲートなど）との通信に必要なその他のインターフェースは、本書には初めから含まれていない。


# 3 本文書の範囲

この勧告には、無人搬送車（AGV）とマスターコントロール間の通信に関する定義とベストプラクティスが含まれている。
異なる特性を持つAGV（例えば、アンダーラントラクターやフォークリフトAGV）が、統一された言語でマスターコントロールと通信できるようにすることが目的である。
これにより、マスターコントロールで任意の組み合わせのAGVを操作するための基盤が構築される。
マスターコントロールは、命令を出し、AGVの交通を調整する。

インターフェースは、自動車業界における生産および工場内物流の要件に基づいて設計されている。
策定された要件に従い、イントラロジスティクスの要件は、物流部門の要件、すなわち、無人搬送車や誘導型搬送車の制御による入庫から生産供給、出庫までの物流プロセスをカバーしている。

無人搬送車とは対照的に、自律走行車両は、対応するセンサーシステムとアルゴリズムに基づいて発生する問題を独自に解決し、動的な環境の変化に適応したり、その変化にすぐに適応したりすることができる。
障害物を独自に回避するなどの自律走行特性は、誘導車両だけでなく、自由走行車両でも実現できます。
ただし、経路計画が車両自体で実行されると、この文書では自由走行車両（用語集を参照）について説明する。
自律型システムは完全に分散型（群知能）ではなく、あらかじめ定義されたルールに従って、規定された動作を行う。

持続可能なソリューションを目的として、以下に構造を拡張可能なインターフェースを説明する。
これにより、誘導される車両のマスターコントロールを完全にカバーすることが可能となる。
自由に走行する車両も構造に統合することが可能である。これに必要な詳細な仕様は、本勧告の対象ではない。

独自仕様の在庫管理システムの統合には、個別のインターフェース定義が必要になる場合があるが、これは本勧告の対象ではない。


## 3.1 その他の適用される文書

文書 | バージョン | 説明
---|---|---
VDI Guideline 2510 | October 2005 | 無人搬送システム（DTS）
VDI Guideline 4451 Sheet 7 | October 2005 | 無人搬送システム（DTS）の互換性 - DTSマスターコントロール
DIN EN ISO 3691-4 | December 2023 | 産業用トラックの安全要件と検証 - 第4部：無人運転トラックとそのシステム
LIF – Layout Interchange Format| March 2024 | 無人搬送車と（サードパーティの）マスターコントロールシステム間のトラックレイアウトの交換フォーマットの定義について。

# 4 要件とプロトコルの定義

通信インターフェースは、以下の要件をサポートするように設計されている。

- 最低1000台の車両の制御
- さまざまな自律性の車両の統合を可能にできること
- 例えばルート選択や交差点での挙動などを決定できること

車両は、一定の間隔で、または車両の状態が変化した時点で、その状態を転送する。

通信は、接続障害やメッセージの損失の影響を考慮した上で、ワイヤレスネットワークを介して行われる。

メッセージプロトコルは、JSON構造と併用されるMessage Queuing Telemetry Transport (MQTT) である。
このプロトコルの開発中にMQTT 3.1.1がテストされ、互換性のために必要とされる最低バージョンとなった。
MQTTでは、「トピック」と呼ばれるサブチャンネルへのメッセージ配信が可能である。
MQTTネットワークの参加者はこれらのトピックを購読し、関心のある情報を受信する。

JSON構造により、追加のパラメータによるプロトコルの将来的な拡張が可能になる。
パラメータは英語で記述されており、プロトコルがドイツ語圏以外でも読みやすく、理解しやすく、適用しやすいように配慮されている。


# 5 コミュニケーションのプロセスと内容

AGVの運用には、少なくとも以下の参加者が存在する。

- AGVシステムの運用者（基本的な情報を提供する）
- マスターコントロール（運用を組織化および管理）
- AGV（命令を実行）

図2は、運用段階における通信内容を説明している。
実装または変更時には、AGVとマスターコントロールを手動で構成する。


![Figure 2 Structure of the Information Flow](./assets/information_flow_VDA5050.png)
>図2 情報の流れの構造

実装段階では、マスターコントロールとAGVで構成される無人搬送システム（DTS）が設定される。
必要な枠組み条件はオペレーターによって定義され、必要な情報はオペレーターが手動で入力するか、他のシステムからインポートしてマスターコントロールに保存する。
基本的には、以下の内容が関係する。
- ルートの定義：CADインポートを使用して、ルートをマスターコントロールにインポートすることができる。
あるいは、オペレーターがマスターコントロールに手動でルートを実装することもできる。
ルートは一方通行の道路であったり、特定の車両グループ（サイズ比に基づく）に制限を設けたりすることができる。
- ルートネットワーク構成：
ルート内では、荷積みおよび荷降ろしステーション、バッテリー充電ステーション、周辺環境（ゲート、エレベーター、バリア）、待機位置、バッファステーションなどが定義される。
車両構成：AGVの物理的特性（サイズ、利用可能な荷台の数など）は、オペレータが保存する。
AGVは、この情報を、この文書のセクション[6.15 `factsheet`トピック](#615-topic-factsheet) で定義されている特定の方法で、`factsheet`トピックを介して通信する。

上述のルート構成およびルートネットワークは、本ドキュメントの対象ではない。
これらは、この情報と完了すべき輸送要件に基づいて、マスターコントロールによる注文管理とコース割り当てを可能にするための基礎となる。
AGVに対する注文は、MQTTメッセージブローカーを介して車両に転送される。
これにより、ジョブの実行と並行して、車両の状態がマスターコントロールに継続的に報告される。
これも、MQTTメッセージブローカーを使用して行われる。

マスターコントロールの機能は以下の通りである。

- AGVへの注文の割り当て
- AGVのルート計算と誘導（各AGVの物理的特性（サイズ、操縦性など）の制限を考慮）
- 障害（デッドロック）の検出と解決
- エネルギー管理：充電の注文が搬送の注文を中断できる
- 交通管理：バッファルートと待機位置
- 特定エリアの解放や最高速度の変更など、環境の（一時的な）変化
- ドア、ゲート、エレベーターなどの周辺システムとの通信
- 通信エラーの検出と解決

AGVの機能は以下の通りである。

- 位置推定
- 関連ルートに沿ったナビゲーション（誘導または自律）
- アクションの実行
- 車両状態の継続的な送信

さらに、インテグレーターは、システム全体を構成する際に以下の点に留意する必要がある（不完全なリスト）：

- マップ構成：マスターコントロールとAGVの座標システムを一致させる必要がある。
- ピボットポイント：AGVの異なるポイントまたは充電ポイントをピボットポイントとして使用すると、車両の異なるエンベロープにつながる。状況によって基準点が異なる場合がある。例えば、荷物を運搬しているAGVと荷物を運搬していないAGVでは基準点が異なる場合がある。


# 6 プロトコル仕様

次のセクションでは、通信プロトコルの詳細について説明する。
このプロトコルは、マスターコントロールとAGV間の通信を規定する。
AGVと周辺機器間の通信、例えばAGVとゲート間の通信は対象外である。

異なるメッセージは、命令、状態などとして送信されるJSONのフィールドの内容を説明する表に示されている。

また、JSONスキーマは、公開されているGitリポジトリ（https://github.com/VDA5050/VDA5050）で検証できる。
JSONスキーマは、VDA5050のリリースごとに更新される。JSONスキーマと本書の内容に相違がある場合は、本書の記述が優先される。


## 6.1 表の記号と書式の意味

表には、識別子の名前、その単位、データタイプ、および説明（存在する場合）が含まれる。

識別子 | 説明
---|---
標準|変数は基本データタイプである
**太字**|変数は非基本データタイプ（例：JSON オブジェクトまたは配列）であり、別途定義されている
*イタリック体*|変数はオプションである
*****イタリック体およびボールド体*****|変数はオプションかつ非基本データタイプである
**非基本データ型配列名[配列データ型]**|オプションの変数（ここでは配列名）は、角括弧内に含まれるデータ型の配列（ここでは配列データ型）である

すべてのキーワードは大文字と小文字が区別される。
すべてのフィールド名はキャメルケースである。
すべての列挙はアンダースコアなしの大文字表記である。


### 6.1.1 オプション項目

変数がオプションとしてマークされている場合、それは送信者にとってオプションであることを意味します。なぜなら、変数は特定のケースでは適用できない可能性があるからです（例えば、マスターコントロールがAGVに命令を送信する場合、一部のAGVは自ら軌道を計画し、その注文の `edge` オブジェクト内の `trajectory` フィールドは省略できます）。
このプロトコルでオプションと指定されたフィールドを含むメッセージをAGVが受信した場合、AGVはそれに応じて動作することが期待され、そのフィールドを無視することはできません。

AGVがメッセージを適切に処理できない場合、期待される動作は、エラーメッセージ内でその旨を通知し、注文を拒否することです。
マスタ制御は、AGVがサポートするオプション情報のみを送信します。

例：軌道はオプションです。
AGVが軌道を処理できない場合、マスターコントロールは車両に軌道を送信しない。

AGVは、必要なオプションパラメータをAGV `factsheet`メッセージで通信する。


### 6.1.2 許可された文字およびフィールド長

すべての通信は、記述の国際的な適応を可能にするためにUTF-8でエンコードされる。
IDには以下の文字のみを使用することが推奨される。

A-Z a-z 0-9 _ - . :

メッセージの最大長は定義されていませんが、MQTTプロトコルの仕様およびファクトシート内で定義された技術的制約によって制限される場合がある。
AGVのメモリが受信した注文を処理するには不十分な場合、その注文は拒否される。
最大フィールド長、文字列長、または値の範囲の一致はインテグレータの判断に委ねられる。
統合を容易にするため、AGVベンダーは[Factsheet Section](#616-topic-factsheet)で詳細を説明しているAGVファクトシートを提供しなければならない。


### 6.1.3 フィールド、トピック、列挙の表記

この文書内のトピックおよびフィールドは、次のスタイルでハイライト表示されます。「exampleField」および「exampleTopic」。
列挙は大文字で表記します。これらの値は、文書内で単一引用符で囲みます。
これには、`actionStatus` フィールド（「WAITING」、「FINISHED」など）などのキーワードが含まれます。

### 6.1.4 JSON データ型

可能な場合は、JSON データ型を使用するものとします。
したがって、ブール値は「true」または「false」でエンコードされ、列挙（「TRUE」、「FALSE」）やマジックナンバーは使用しません。
数値データ型は、型と精度（例：float64 または uint32）で指定します。IEEE 754 の NaN や infinity のような特殊な数値値はサポートされていません。


## 6.2 MQTT 接続の処理、セキュリティ、QoS

MQTT プロトコルでは、クライアントの最終メッセージを設定するオプションが提供されている。
何らかの理由でクライアントが予期せず切断した場合、ブローカーによって他の購読中のクライアントに最終メッセージが配信される。
この機能の使用については、セクション [6.14 トピック「接続」](#614-topic-connection) で説明されている。

AGVがブローカーから切断された場合、すべての注文情報を保持し、最後にリリースされたノードまで注文を履行する。

プロトコルのセキュリティは、ブローカー構成で考慮する必要がある。

通信オーバーヘッドを削減するために、トピック `order`、`instantActions`、`state`、`factsheet`、および `visualization` には、MQTT QoSレベル0（ベストエフォート）を使用する。
トピック「connection」は、QoSレベル1（最低でも1回）を使用する。


## 6.3 MQTT トピックレベル

クラウドプロバイダーの必須のトピック構造により、MQTT トピック構造は厳密には定義されていない。
クラウドベースの MQTT ブローカーでは、トピック構造は本プロトコルで定義されたトピックに一致するように個別に適応させる必要がある。
つまり、以下のセクションで定義されたトピック名は必須である。

ローカルブローカーでは、MQTT トピックレベルは以下のようにお勧めする。

**インターフェース名/メジャーバージョン/メーカー/シリアル番号/トピック**

例：
```
uagv/v2/KIT/0001/order
```

MQTT トピックレベル|データタイプ|説明
---|---|---
interfaceName | string | 使用インターフェースの名前
majorVersion | string | `v`で始まる VDA 5050 勧告のメジャーバージョン番号
manufacturer | string | AGVのメーカー名
serialNumber | string | 以下の文字で構成されるAGVの固有のシリアル番号：<br>A-Z <br>a-z <br>0-9 <br>_ <br>. <br>: <br>-
topic | string | トピック（例：注文や状態） セクション[6.5 トピックによる通信](#65-topics-for-communication)を参照

注：トピック階層を定義するために`/`文字が使用されるため、前述のフィールドのいずれにも使用しないでください。
`$`文字も、一部のMQTTブローカーで特別な内部トピックに使用されるため、使用しないでください。


## 6.4 プロトコルヘッダー

各 JSON メッセージはヘッダーで始まります。
以下のセクションでは、読みやすさを考慮して、以下のフィールドをヘッダーとして参照します。
ヘッダーは以下の個々の要素で構成されます。
ヘッダーは JSON オブジェクトではありません。

オブジェクト構造/識別子|データ型|説明
---|---|---
headerId|uint32|メッセージのヘッダーID。<br>headerIdはトピックごとに定義され、送信メッセージごとに1ずつインクリメントされる（受信では必要ない）。
timestamp | string | タイムスタンプ（ISO 8601、UTC）YYYY-MM-DDTHH:mm:ss.ffZ（例：「2017-04-15T11:40:03.12Z」）。
version | string | プロトコルのバージョン [メジャー].[マイナー].[パッチ]（例：1.3.2）。
manufacturer | string | AGVのメーカー名。
serialNumber | string | AGVのシリアル番号。


### プロトコルのバージョン

プロトコルのバージョンは、バージョンスキーマとしてセマンティックバージョンニングを使用します。

メジャーバージョン変更の例：

- ブレイキングチェンジ、例えば、必須フィールドの追加

マイナーバージョン変更の例：

- 可視化のためのトピックの追加などの新機能

パッチバージョンの例：

- batteryChargeのより高い精度


## 6.5 通信トピック

AGVプロトコルでは、マスターコントロールとAGV間の情報交換に以下のトピックを使用する。

トピック名 | 公開者 | 購読者 | 用途 | 実装 | スキーマ
---|---|---|---|---|---
order|マスターコントロール|AGV|マスターコントロールからAGVへの運転命令の通信|必須|order.schema
instantActions | マスターコントロール | AGV | 即座に実行されるアクションの通信 | 必須 |instantActions.schema
state | AGV | マスターコントロール | AGVの状態の通信 | 必須 | state.schema
visualization | AGV | 視覚化システム | 視覚化のみを目的とした位置情報を有する更新頻度の高いトピック | オプション | visualization.schema
connection | ブローカー/AGV | マスターコントロール | AGV接続が失われたことを示す。車両の状態を確認するためにマスターコントロールで使用しない。MQTTプロトコルの接続レベルチェック用に追加された | 必須 | connection.schema 
factsheet | AGV | マスターコントロール | マスターコントロールにおけるAGVのセットアップを支援するパラメータまたはベンダー固有の情報 | 必須 | factsheet.schema


## 6.6 トピック："order"（マスターコントロールからAGVへの）

"order"トピックは、AGVがJSON形式でカプセル化された注文を受信するMQTTトピックです。


### 6.6.1 概念とロジック

注文の基本構造は、ノードとエッジのグラフである。
AGVは、注文を履行するために、ノードとエッジを横断することが期待される。
すべての接続されたノードとエッジの完全なグラフは、マスタ制御によって保持される。

マスタ制御のグラフ表現には、例えば、どのAGVがどのエッジを横断することが許可されているかなどの制限が含まれている。
これらの制限はAGVに通知されない。
マスターコントロールには、該当するAGVが通過することが許可されているエッジのみが含まれる。

![Figure 3 Graph representation in master control and graph transmitted in orders](./assets/graph_representation_transmission.png)
>図3 マスターコントロールのグラフ表現とオーダーで送信されるグラフ

ノードとエッジは、2つのリストとしてオーダーメッセージで渡される。
これらのリスト内のノードとエッジの順序は、ノードとエッジが走査される順序も決定する。

有効な順序では、少なくとも1つのノードが存在し、エッジの数はノードの数から1を引いた数と等しくなければならない。

順序の最初のノードは、AGVが簡単に到達できるものでなければならない。
これは、AGVがすでにそのノード上に存在しているか、またはAGVがそのノードの偏差範囲内にあることを意味する。

ノードとエッジには、ブール値の属性`released`があります。
ノードまたはエッジがリリース済みの場合、AGVはそれを横断することが期待される。
ノードまたはエッジがリリースされていない場合、AGVはそれを横断しません。

エッジは、その始点と終点の両方のノードがリリースされている場合にのみリリースすることができる。

未解放のエッジの後に、解放済みのノードまたはエッジを配置することはできない。

解放済みのノードとエッジのセットは「ベース」と呼ばれる。
未解放のノードとエッジのセットは「ホライズン」と呼ばれる。

ホライズンなしでオーダーを送信することは有効です。

注文メッセージは、必ずしも完全な輸送順序を記述しているとは限りません。
トラフィック制御やリソースに制約のある車両への対応のため、完全な輸送順序（多数のノードやエッジで構成される場合もあります）を多数のサブ順序に分割し、それらを `orderId` および `orderUpdateId` によって接続することができます。
注文の更新プロセスについては、次のセクションで説明します。


### 6.6.2 注文と注文の更新

トラフィック管理をサポートするために、マスターコントロールは注文によって伝達された経路を2つの部分に分割することができる。

- *「ベース」*：これはAGVが走行することを許可された定義済みのルートである。ベースルートのすべてのノードとエッジは、車両用にマスターコントロールによってすでにリリースされている。ベースの最後のノードは決定ポイントと呼ばれる。
- *「ホライズン」*：これは、決定ポイント以降にAGVが走行するよう、マスターコントロールによって現在計画されているルートである。ホライズンルートは、マスターコントロールによってまだリリースされていない。

ベースに追加のノードやエッジが追加されない場合、AGVは決定ポイントで停止する。円滑な移動を確保するため、交通状況が許す場合、AGVが決定ポイントに到達する前に、マスターコントロールがベースを拡張する必要がある。

MQTTは非同期プロトコルであり、無線ネットワークを介した通信は信頼性が低いため、ベースは変更できない。したがって、マスターコントロールは、ベースはすでにAGVによって実行済みであると想定する。後述のセクションでは、注文をキャンセルする手順について説明するが、これも前述の通信制限により信頼性が低いと考えられる。

マスターコントロールは、変更されたノードとエッジのリストを含む更新されたルートをAGVに送信することで、ホライズンを変更することができる。ホライズンルートの変更手順は、図4に示されている。

![Figure 4 Procedure for changing the driving route "Horizon"](./assets/driving_route_horizon.png)
>図4 運転ルート「ホライズン」変更手順

図4では、時刻t=1にコントロールパネルから最初のジョブが送信される。
図5は、可能なジョブの擬似コードを示している。
読みやすさを考慮し、完全なJSONの例はここでは省略されている。

```
{
	orderId: "1234"
	orderUpdateId:0,
	nodes: [
	 	 f {released: True},
	 	 d {released: True},
	 	 g {released: True},
	 	 b {released: False},
	 	 h {released: False}
	],
	edges: [
		e1 {released: True},
		e3 {released: True},
		e8 {released: False},
		e9 {released: False}
	]
}
```
>図5 注文の擬似コード

時刻 t = 3 で、注文の更新が注文の拡張を送信することで行われる（図6の例を参照）。
`orderUpdateId` がインクリメントされ、注文更新の最初のノードが、前の注文メッセージの最後の共有ベースノードに対応していることに注目すること。

これにより、AGVが注文更新も実行できることが保証される。すなわち、AGVがすでに認識しているエッジを実行することで、ジョブ更新の最初のノードに到達できることが保証される。

```
{
	orderId: 1234,
	orderUpdateId: 1,
	nodes: [
		g {released: True},
		b {released: True},
		h {released: True},
		i {released: False}
	],
	edges: [
		e8 {released: True},
		e9 {released: True},
		e10 {released: False}
	]
}
```
>図6 注文更新の擬似コード - `orderUpdateId` の変更に注目すること。

また、これは、注文の更新が失われた場合（例えば、信頼性の低いワイヤレスネットワークが原因）にも役立つ。
AGVは、常に最後の既知のベースノードが最初の新しいベースノードと同じ `nodeId`（および `nodeSequenceId`、これについては後述）を持っていることを確認できる。

また、ノードgのみが再度送信されるベースノードである点にも注目すること。
ベースは変更できないため、ノードfとdの再送信は無効である。

ステッチングノード（例ではノードg）の内容が変更されないことが重要である。
アクション、偏差範囲などについては、AGVは最初のオーダー（図5、orderUpdateId 0）で提供された指示を使用する。

![Figure 7 Regular update process - order extension](./assets/update_order_extension.png)
>図7 定期更新プロセス - 注文延長。

図7は、注文がどのように拡張されるかを説明している。
これは、現在AGV上で利用可能な情報を示している。
`orderId`は同じまま、`orderUpdateId`はインクリメントされる。

前のベースの最後のノードは、更新された注文の最初のベースノードである。
このノードを使用して、AGVは現在の注文に更新された注文を追加することができる（ステッチング）。
前のベースからの他のノードとエッジは再送信されない。

マスターコントロールは、まったく異なるノードを新しいベースとして送信することで、ホライズンに変更を加えるオプションがある。
また、ホライズンを削除することもできる。

注文にループ（ノードaからbへ行き、その後aに戻るなど）を許可するには、ノードおよびエッジオブジェクトに`sequenceId`が割り当てられる。
この `sequenceId` は、ノードとエッジに順に割り当てられる（注文の最初のノードには 0、最初のエッジには 1、2番目のノードには 2、以下同様）。
これにより、注文の進行状況をより簡単に追跡できるようになる。

一度 `sequenceId` が割り当てられると、注文の更新では変更されない（図7を参照）。
これは、AGV側で、マスターコントロールがどのノードを参照しているかを判断するために必要である。

図8は、注文または注文更新の受け付けプロセスを説明している。

![Figure 8 The process of accepting an order or orderUpdate](./assets/process_order_update.png)
>図8 注文の受注または注文の更新のプロセス

1)	**受け取った注文は有効か？**:
すべての書式およびJSONデータタイプは正しい？

2)	**新しい注文か、現在の注文の更新か？**:
受け取った注文の `orderId` は、車両が現在保持している注文の `orderId` と異なるか？

3)	**車両はまだ命令を実行中か、それとも更新を待っているか？**:
`nodeStates` が空ではないか、または `actionStates` が 'FAILED' でも 'FINISHED' でもない状態を含んでいるか？ ノード、エッジ、およびオーダーホライズンの対応するアクション状態も状態に含まれる。 車両はまだホライズンを持っている可能性があり、そのため更新を待機し、オーダーを実行している可能性がある。

4) **現在の位置に対して新しい注文の開始位置は十分近いか？**:	車両はすでにノード上に存在しているのか、それともノードの偏差範囲内にあるのか（セクション[6.6.1 概念とロジック](#661-概念とロジック)を参照）。

5) **受け取った注文の更新は非推奨か？**: `orderUpdateId`は現在車両に搭載されているものよりも小さいですか？

6)	**車両に現在、注文更新が受信されているか？**: `orderUpdateId`は現在車両に搭載されているものと同一か？

7)	**受信した更新は、現在も継続中の注文の有効な継続であるか？**:	受信した注文の最初のノードは、現在の決定ポイント（現在のベースの最後のノード）と等しいか？車両は、以前の注文更新でリリースされたベースに関連する動作をまだ実行中、またはホライズンを保有しているため、注文の継続を待機している状態です。この場合、新しいベースの最初のノードが、前のベースの最後のノードと等しい場合にのみ、注文更新が受け入れられる。

8)	**受信した更新は、以前に完了した注文の有効な継続であるか？**: 受信した注文更新の最初のノードの `nodeId` と `sequenceId` が、`lastNodeId` と `lastNodeSequenceId` に等しいか？車両はもはや何もアクションを実行せず、また注文の継続を待つこともない（つまり、関連するすべてのアクションを完了し、ホライズンを持たないことを意味する）。注文の更新は、最後に横断したノードから継続する場合は受け入れられるため、新しいベースの最初のノードは、車両の `lastNodeId` および `lastNodeSequenceId` と一致する必要がある。

9)	populate/append statesは、`actionStates`/`nodeStates`/`edgeStates`を参照します。


### 6.6.3 注文キャンセル（マスターコントロールによる）

ベースノードに予定外の変更が生じた場合は、instantAction `cancelOrder` を使用して注文をキャンセルする。

instantAction `cancelOrder` を受信すると、車両は停止する（車両の能力に基づいて、例えば、その場で、あるいは次のノードで）。

スケジュールされたアクションがある場合は、それらのアクションをキャンセルし、`actionState` に'FAILED'と報告する。
実行中のアクションがある場合は、それらのアクションをキャンセルし、'FAILED'と報告しなければならない。
アクションを中断できない場合は、そのアクションの `actionState` は実行中には「RUNNING」を報告し、その後はそれぞれの状態（成功した場合は'FINISHED'、失敗した場合は'FAILED'）を報告することで、それを反映しなければならない。
アクションが実行されている間、cancelOrderアクションは、すべてのアクションがキャンセルまたは終了するまで'RUNNING'と報告する。
車両のすべての動作とアクションが停止した後、`cancelOrder`アクションの状態は'FINISHED'と報告する。

orderIdとorderUpdateIdは保持される。

図9は、AGVのさまざまな能力に対する期待される動作を示している。

![Figure 9 Expected behavior after a cancelOrder](./assets/process_cancel_order.png)
>図 9 `cancelOrder`を受け取ったときに期待される動作.


#### 6.6.3.1 キャンセル後の新規注文の受理

注文がキャンセルされた後、車両は新たな注文を受けられる状態になければならない。

タグによってノードに位置を特定するAGVの場合、新たな注文はAGVが現在立っているノードから開始しなければならない（図5も参照）。

ノードの間に停止できるAGVの場合、次の注文をどのように開始するかは、マスタ制御によって選択される。
AGVは両方の方法を受け入れなければならない。

2つのオプションがあります。

- 最初のノードをAGVが現在位置する仮のノードとして、オーダーを送信する。AGVは、このノードが簡単に到達可能であることを認識し、オーダーを受け入れる。
- 最初のノードは前回の注文の最後の通過ノードですが、偏差範囲を大きく設定し、AGVがその範囲内に収まるようにする。これにより、AGVは、このノードを通過したと認識し、注文を受け入れる。


#### 6.6.3.2 AGVに注文がない場合のcancelOrderアクションの受信

AGVが「cancelOrder」アクションを受信したが、AGVに現在注文がない場合、または以前の注文がキャンセルされた場合、`cancelOrder`アクションは`FAILED`として報告される。

AGVは"noOrderToCancel"エラーを'WARNING'に設定した`errorLevel`とともに報告する。
`instantAction`の`actionId`は`errorReference`として渡されます。


### 6.6.4 注文の拒否

注文が拒否される状況はいくつかある。
これらの状況は図8に示されており、以下に説明されている。


#### 6.6.4.1 車両が不正な新規注文を取得する

解決策：

1. 車両は新規注文を内部バッファに引き継がない。
2. 車両は「validationError」という警告を報告する。
3. 車両が新規注文を受け入れるまで、この警告が報告される。


#### 6.6.4.2 車両が実行できないアクションまたは使用できないフィールドを含む注文を受け取る

例：

- 実行不可能なアクション：最大リフト高を超えるリフト高、ストロークが設置されていないにもかかわらずリフト動作など
- 使用不可能なフィールド：軌道など

解決策：

1. 車両は、新しい注文を内部バッファに引き継がない。
2. 車両は、誤ったフィールドをエラー参照として警告「orderError」を報告する。
3. 警告は、車両が新しい注文を受け入れるまで報告されるものとする。


#### 6.6.4.3 車両が同じorderIdで、現在のorderUpdateIdよりも小さいorderUpdateIdを持つ新規注文を取得

解決策：

1. 車両は新規注文を内部バッファに引き継がない。
2. 車両は以前の注文をバッファに保持する。
3. 車両は「orderUpdateError」という警告を報告する。
4. 車両は以前の注文の実行を継続する。

AGVが同じ `orderId` と `orderUpdateId` を持つ注文を2回受信した場合、2番目の注文は無視される。
これは、ステートメッセージがマスターコントロールに遅れて受信されたため、最初の注文が受信されたことを確認できなかった場合に、マスターコントロールが注文を再送信した場合に発生する可能性がある。

### 6.6.5 Corridors

オプションの `corridor` エッジ属性は、障害物を回避するために車両がエッジ軌道から逸脱することを許可し、車両が動作することを許可する境界を定義する。
`corridor`属性を使用するには、`corridor`属性が定義されていない場合に車両が従うことになる、あらかじめ定義された軌道が必要である。これは、マスターコントロールで認識されている車両で定義された軌道、または命令で送信された軌道のいずれかになる。`corridor`属性を使用する車両の動作は、障害物を回避するために一時的に軌道から逸脱することが許可されていることを除いて、依然としてラインガイド車両の動作である。

*注記：*
*オーダー内のエッジは、2つのノード間の論理的な接続を定義するものであり、車両が開始ノードから終了ノードまで走行する際にたどる（実際の）軌道であるとは限りません。*
*車両の種類に応じて、開始ノードと終了ノードの間の車両の軌道は、軌道エッジ属性を介してマスター制御によって定義されるか、または車両に事前定義の軌道として割り当てられます。*
*車両の内部状態に応じて、選択される軌道は異なります。*

![Figure 10 Edges with corridor attribute.](./assets/edges_with_corridors.png)
>図10 障害物を避けるために車両が予め設定された軌道から逸脱することが許される左右の境界を定義する「corridor」属性を持つエッジ。左側では、運動中心が許容される逸脱を定義し、右側では、荷重により拡張されるかもしれない車両の輪郭が許容される逸脱を定義する。これは`corridorRefPoint`パラメータによって定義される。

車両が単独で走行（元のエッジ軌道から逸脱）することが許可される領域は、左右の境界によって定義される。
オプションの `corridorRefPoint` フィールドは、車両の制御点または車両の輪郭が定義された境界の内側にあるべきかどうかを指定する。
エッジの境界は、車両がノードを通過すると同時に、新しいエッジと現在のエッジの境界の内側になるように定義される。
車両が軌道から逸脱しない場合は、corridor境界をゼロに設定する代わりに、マスターコントロールは`corridor`属性を使用しない。

車両のモーション制御ソフトウェアは、車両が定義された境界内に常に収まっていることを確認する。
収まっていない場合は、車両は許容されたナビゲーション空間から外れているため停止し、エラーを報告する。
マスター制御は、ユーザーとの対話が必要かどうか、または車両が現在の命令を取り消して、車両が再び移動できるcorridor情報を含む新しい命令を車両に送信することで、車両が走行を継続できるかどうかを決定できる。

*注記：車両が軌道から逸脱することを許可すると、走行中の車両のフットプリントが大きくなる可能性がある。この状況は初期運用時に考慮すべきであり、マスターコントロールが車両のフットプリントに基づいて交通制御の決定を下す場合がある。*

セクション[6.10.2 ノードの横断とエッジへの進入/退出](#6102-traversal-of-nodes-and-enteringleaving-edges-triggering-of-actions)も参照すること。


## 6.6.6 注文メッセージの実装

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
headerId | | uint32 | メッセージのヘッダーID<br>ヘッダーIDはトピックごとに定義され、送信されるメッセージごとに1ずつインクリメントされます（受信時は必要ない）。
timestamp | | string | タイムスタンプ (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ffZ (e.g., "2017-04-15T11:40:03.12Z")
version | | string | プロトコルのバージョン [メジャー].[マイナー].[パッチ] (例：1.3.2)
manufacturer | | string | AGVのメーカ名
serialNumber | | string | AGVのシリアル番号
orderId | | string | 注文の識別子<br>これは、同一の注文に属する複数の注文メッセージを識別するために使用する。
orderUpdateId | | uint32 | 注文更新識別子<br>orderIdごとに一意である。<br>注文更新が拒否された場合、このフィールドは拒否メッセージに渡される。
*zoneSetId* | | string | AGVがナビゲーションに使用する、またはマスターコントロールが計画に使用したゾーンセットの固有の識別子<br> <br>オプション：一部のマスターコントロールシステムではゾーンを使用しない。br>一部のAGVはゾーンを理解しない。<br>ゾーンを使用しない場合は、メッセージに追加しないこと。
**nodes [node]** | | array | 注文を履行するために巡回するノードオブジェクトの配列<br>有効な注文には1つのノードで十分である。<br>その場合はエッジ配列を空欄のままにしておく。
**edges [edge]** | | array | 注文を履行するために巡回するエッジオブジェクトの配列<br>有効な注文には1つのノードで十分である。<br>その場合はエッジ配列を空欄のままにしておく。

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**node** { | | JSON object|
nodeId | | string | 固有ノード識別子
sequenceId | | uint32 | ノードとエッジのシーケンスを追跡し、注文の更新を簡素化するための番号<br>主な目的は、1つのorderId内で複数回渡されるノードを区別することである。<br>この変数sequenceIdは、同じ注文のすべてのノードとエッジにまたがって適用され、新しいorderIdが発行されるとリセットされる。
*nodeDescription* | | string | ノードに関する追加情報
released | | boolean | "true"は、ノードがベースの一部であることを示す。<br>"false"は、ノードがホライズンの一部であることを示す。
***nodePosition*** | | JSON object | ノード位置 <br>ノード位置を必要としない車両タイプ（例：線路誘導車両）の場合はオプション。
**actions [action]** <br> } | | array | ノード上で実行されるアクションの配列<br>アクションが不要な場合は空の配列。

オブジェクト構造 | 単位 | データ型 | 説明
---| --- |--- | ---
**nodePosition** { | | JSON object | グローバルなプロジェクト固有の世界座標系におけるマップ上の位置を定義する<br>各フロアには独自のマップがある。<br>すべてのマップは、同じプロジェクト固有のグローバル原点を使用する。
x | m | float64 | マップ座標システムを参照するマップ上のX位置<br>精度は特定の実装に依存する。
y | m | float64 | マップ座標システムを参照するマップ上のY位置<br>精度は特定の実装に依存する。
*theta* | rad | float64 | 範囲: [-Pi ... Pi] <br><br>ノード上でのAGVの絶対方向<br>オプション：車両は自ら経路を計画できる。<br>定義されている場合、AGVはノード上のシータ角を想定しなければならない。<br>前のエッジが回転を禁止している場合、AGVはノード上で回転する。<br>次のエッジに異なる方向が定義されているが回転が禁止されている場合、AGVはエッジに進入する前に、ノード上でエッジの希望する方向に回転する。
*allowedDeviationXY* | m | float64 | AGVがノードの位置にどれだけ正確に一致しなければならないかを示し、通過したと見なされる。 <br><br> If = 0.0: 偏差は許されない（偏差がないとは、AGVメーカーの通常の許容範囲内であることを意味する）。<br><br> If > 0.0: 許容される偏差半径（メートル単位）。 <br>AGVが偏差半径内でノードを通過した場合、そのノードは通過したと見なされる。
*allowedDeviationTheta* | rad | float64 | 範囲: [0.0 ... Pi] <br><br> AGVがノードで満たさなければならない、thetaで定義された方向の正確さを示す。<br>許容可能な最小角度はtheta - allowedDeviationTheta、許容可能な最大角度はtheta + allowedDeviationThetaである。
mapId | | string | 位置が参照されるマップの固有の識別<br>各マップには、プロジェクト固有のグローバル座標の原点が存在する。<br>AGVがエレベーターを使用する場合、例えば出発階から目的階へ移動する場合、AGVは出発階のマップ上から消え、目的階のマップ上の関連エレベーターノードに生成される。
*mapDescription* <br> } | | string | 地図に関する追加情報

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**action** { | | JSON object | AGVが実行できるアクションを記述する。
actionType | | string | "アクションとパラメータ"の最初の列に記載されているアクションの名前<br>アクションの機能を識別する。
actionId | | string | アクションを識別し、状態のactionStateにマッピングするための一意のID<br>提案：UUIDを使用すること。
*actionDescription* | | string | アクションに関する追加情報
blockingType | | string | Enum {'NONE', 'SOFT', 'HARD'}: <br> 'NONE'：運転やその他の操作が可能。<br>'SOFT'：運転以外の操作は可能だが、運転は不可。<br>'HARD'：その時間帯に許可されているのはこの操作のみ。
***actionParameters [actionParameter]*** <br><br> } | | array | 指定されたアクションのアクションパラメータオブジェクトの配列、例えば"deviceId"、"loadId"、"external triggers"など。 <br><br>実装例は、[7.2 パラメータの形式](#72-format-of-parameters)を参照してください。

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**edge** { | | JSON object | 2つのノード間の方向性のある接続
edgeId | | string | 固有エッジ識別子
sequenceId | | uint32 | ノードとエッジのシーケンスを追跡し、注文更新を簡素化するための番号<br>変数 sequenceId は同じ注文のすべてのノードとエッジにまたがって適用され、新しい orderId が発行されるとリセットされます。
*edgeDescription* | | string | エッジに関する追加情報
released | | boolean | "true"は、エッジがベースの一部であることを示す。<br>"false"は、エッジがホライズンの一部であることを示す。
startNodeId | | string | 注文内の最初のノードのノードID
endNodeId | | string | 注文内の最後のノードのノードID
*maxSpeed* | m/s | float64 | エッジでの許容最大速度<br>速度は車両の最速の測定値によって定義されます。
*maxHeight* | m | float64 | エッジ上での車両の最大許容高さ（積載物を含む）
*minHeight* | m | float64 | エッジ上での荷役装置の最小許容高さ
*orientation* | rad | float64 | AGVのエッジ上の方向。`orientationType`の値は、グローバルなプロジェクト固有のマップ座標系に対する相対値として解釈されるか、エッジに対する接線として解釈されるかを定義する。エッジに対する接線として解釈される場合、0.0は前進、PIは後進を意味する。<br>例：方向 Pi/2 rad は90度の回転を意味する。`rotationAllowed`が"true"に設定されている場合、AGVが異なる方向から開始した場合は、エッジ上で車両を希望の方向に回転させる。<br>`rotationAllowed`が"false" の場合は、エッジに進入する前に回転させる。<br>それが不可能な場合は、注文を拒否する。<br><br>軌道が定義されていない場合は、エッジの2つの接続ノード間の直接経路に方向を適用する。<br>エッジに軌道が定義されている場合は、その軌道に方向を適用する。
*orientationType* | | string | Enum {'GLOBAL', 'TANGENTIAL'}: <br>'GLOBAL'：グローバルなプロジェクト固有のマップ座標系に対する相対値。<br>'TANGENTIAL'：エッジに対する接線。<br>定義されていない場合、デフォルト値は'TANGENTIAL'である。
*direction* | | string | 線路誘導またはワイヤ誘導の車両が分岐点で進む方向を設定します。最初に定義します（車両個別）。<br>例： "left", "right", "straight".
*rotationAllowed* | | boolean | "true": エッジで回転が許可される。<br>"false": エッジで回転が許可されない。<br><br>オプション:<br>設定されていない場合は制限なし。
*maxRotationSpeed* | rad/s | float64| 最大旋回速度<br><br>オプション:<br>設定されていない場合、上限はない。
***trajectory*** | | JSON object | このエッジのNURBSとしての軌道JSONオブジェクト<br>エッジの開始ノードと終了ノードの間でAGVが移動すべき経路を定義する。<br><br>オプション：<br>AGVが軌道を処理できない場合、またはAGVが独自の軌道を計画している場合は、省略できる。
*length* | m | float64 | 開始ノードから終了ノードまでのパスの長さ<br><br>オプション：<br>この値は、停止位置に到達する前に速度を落とすために、誘導線に従うAGVで使用される。
***corridor*** | | JSON object | 車両が軌道から逸脱できる境界の定義、例えば障害物を避ける場合など。<br>
**action [action]**<br><br><br> } | | array | エッジ上で実行されるアクションの配列<br>アクションが不要な場合は空の配列。<br>エッジによってトリガーされたアクションは、AGVがそのアクションをトリガーしたエッジを横断している間のみ有効である。<br>AGVがエッジを離れると、アクションは停止し、エッジに入る前の状態が復元される。


オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**trajectory** { | | JSON object |
degree | | float64 | 範囲: [1.0 ... float64.max]<br><br>軌道を定義するNURBS曲線の度合い<br><br>定義されていない場合は、デフォルト値は1である。
**knotVector [float64]** | | array | 範囲: [0.0 ... 1.0]<br><br>NURBSのノット値の配列<br><br>knotVectorは制御点の数+次数+1のサイズである。
**controlPoints [controlPoint]**<br><br> } | | array | NURBSの制御点を定義するcontrolPointオブジェクトの配列<br>開始点と終了点も明示的に含む。

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**controlPoint** { | | JSON object |
x | | float64 | 世界座標系で表記されたX座標
y | | float64 | 世界座標系で表記されたY座標
*weight* | | float64 | 範囲: [0.0 ... float64.max]<br><br>カーブ上の制御点の重み<br>定義されていない場合、デフォルトは1.0となる。
} | | |

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
***corridor*** { | | JSON object |
leftWidth | m | float64 | 範囲: [0.0 ... float64.max]<br>車両の軌跡に関連する左側の通路の幅をメートル単位で定義する（図13参照）。
rightWidth | m | float64 | 範囲: [0.0 ... float64.max]<br>車両の軌跡に関連する右側の通路の幅をメートル単位で定義する（図13参照）。
*corridorRefPoint* <br><br>**}**| | string | 境界が車両の運動中心または輪郭のどちらに対して有効であるかを定義する。指定されていない場合、境界は車両の運動中心に対して有効である。<br> Enum { 'KINEMATICCENTER' , 'CONTOUR' }

### 6.7 地図

異なる種類のAGV間のナビゲーションの一貫性を確保するため、位置は常にプロジェクト固有の座標系を参照して指定される（図11を参照）。
サイトや場所の異なるレベル間の区別には、固有の`mapId`が使用される。
マップ座標系は、z軸が空を指す右手座標系として指定される。
したがって、正の回転は反時計回りの回転と解釈される。
車両座標系もまた、x軸が車両の進行方向を指し、z軸が上方向を指す右ねじれ座標系として指定される。車両の基準点は、特に指定のない限り、車両基準フレームにおいて (0,0,0) と定義される。
これは、DIN ISO 8855 のセクション2.11 に準拠している。

![Figure 11 Coordinate system with sample AGV and orientation](./assets/coordinate_system_vehicle_orientation.png)
>図11 サンプルAGVと方向を示す座標系

X、Y、Zの座標はメートル法で表記する。
方向はラジアンで表記し、+Piと-Piの範囲内とする。

![Figure 12 Coordinate systems for map and vehicle](./assets/coordinate_system_vehicle_map.png)
>図12 地図と車両の座標系


### 6.7.1 地図の配布

地図の自動配布と、必要に応じて車両を再起動するインテリジェントな管理を可能にするために、地図を配布するための標準化された方法が導入されている。

配布される地図ファイルは、車両からアクセス可能な専用の地図サーバーに保存される。効率的な送信を確実に行うために、各送信は単一のファイルで構成される必要がある。複数の地図またはファイルが必要な場合は、それらを単一のファイルにバンドルまたはパックする必要がある。地図サーバから車両に地図を転送するプロセスはプル操作であり、マスターコントロールが`instantAction`を使用してダウンロードコマンドをトリガーすることで開始される。

各地図は、地図識別子（フィールド`mapId`）と地図バージョン（フィールド`mapVersion`）の組み合わせによって一意に識別される。マップ識別子は車両の物理的な作業領域の特定のエリアを記述し、マップバージョンは以前のバージョンに対する更新を示す。新しい注文を受け入れる前に、車両は要求された注文の各マップ識別子に対して車両にマップがあることを確認する。車両を操作するために正しいマップが起動されていることを確認するのはマスターコントロールの責任である。

ダウンタイムを最小限に抑え、マスターコントロールによる新しいマップのアクティベーションの同期を容易にするには、車両に事前にマップをロードしておくか、バッファリングしておくことが不可欠である。車両上のマップの状態は、車両状態チャンネルを介してアクセスできる。ここで注意すべき点は、AGVにマップを転送することと、マップをアクティベートすることは、異なるプロセスであるということである。車両に事前にロードされたマップをアクティベートするには、マスターコントロールが即時アクションを送信する。この場合、同じマップ識別子で異なるバージョンの他のマップは自動的に無効になる。マップは、別の即時アクションによりマスターコントロールで削除することができます。このプロセスの結果は、車両の状態に表示される。

マップ配布の流れは、図13に示されている。

![Figure 13 Map distribution process](./assets/map_distribution_process.png)
>図13 マスターコントロール、AGV、マップサーバー間で、マップのダウンロード、有効化、削除を行うために必要な通信。


#### 6.7.2 車両状態の地図

ステートの `agvPosition` の `mapId` フィールドは、現在アクティブなマップを表す。車両で利用可能なマップに関する情報は、ステートメッセージのコンポーネントである `maps` 配列に表示される。この配列の各エントリは、必須フィールドである `mapId`、`mapVersion`、および `mapStatus` から構成される JSON オブジェクトである。これらのフィールドは、'ENABLED' または 'DISABLED' のいずれかになる。'ENABLED'のマップは必要に応じて車両で使用することができる。'DISABLED'のマップは使用できない。ダウンロード処理の状態は、現在のアクションが完了していないことによって示される。エラーも状態に報告される。

異なる `mapId` を持つ複数のマップを同時に有効にできることに注意すること。同じ `mapId` を持つマップの有効なバージョンは、同時に1つだけである。 `maps` 配列が空の場合、車両で現在利用可能なマップがないことを意味する。

#### 6.7.3 地図のダウンロード

マップのダウンロードは、マスターコントロールからのインスタントアクション`downloadMap`によって開始される。このコマンドには、マップサーバーに保存され、車両からアクセスされるマップの必須パラメータ`mapId`と`mapDownloadLink`が含まれている。

AGVは、マップファイルのダウンロードを開始すると同時に、`actionStatus`を'RUNNING'に設定する。ダウンロードが成功すると、`actionStatus`は'FINISHED'に更新される。ダウンロードに失敗した場合は、ステータスが'FAILED'に設定される。ダウンロードが正常に完了すると、マップは状態の配列 `maps` に追加される。マップは、有効化できる状態になる前に状態に報告されてはならない。

マップのダウンロード処理が、車両上の既存のマップを修正、削除、有効化、または無効化しないことを確認することが重要である。
車両には、すでに車両に存在する `mapId` および `mapVersion` を持つ地図のダウンロードは拒否される。エラーが報告され、その時点でのアクションのステータスは'FAILED'に設定される。マスターコントロールは、まず車両上の地図を削除し、その後ダウンロードを再開する。


#### 6.7.4 ダウンロードされた地図の有効化

車両でマップを有効にするには、2つの方法がある。

1. **マスターコントロールでマップを有効にする**：インスタントアクション`enableMap`を使用して、車両のマップを'ENABLED'に設定します。同じ`mapId`で`mapVersion`が異なる他のバージョンは、'DISABLED'に設定される。
2. **車両でマップを手動で有効にする**：場合によっては、車両でマップを直接有効にする必要がある。結果は車両ステートで報告される。

注文で `nodePosition` の一部として対応する `mapId` を送信する際には、車両に正しいマップがアクティブになっていることを確認することがマスターコントロールの責任となる。
車両を新しいマップの特定の位置に設定する場合は、インスタントアクションの `initPosition` を使用する。

#### 6.7.5 車両上地図の削除

マスターコントロールは、特定のマップの削除を車両に要求することができる。これは即時アクションの `deleteMap` を使用して行う。車両のメモリが不足した場合は、マスターコントロールに報告し、マスターコントロールがマップの削除を開始する。車両自体がマップを削除することは許可されていない。
マップの削除に成功した後、車両の状態における車両のマップの配列からそのマップのエントリを削除することが重要である。


## 6.8 Actions

If the AGV supports actions other than driving, these actions are executed via the action field that is attached to either a node or an edge, or sent via the separate topic `instantActions` (see Section [6.10 Topic "instantActions"](#610-topic-instantactions-from-master-control-to-agv)).

Actions that are to be executed on an edge shall only run while the AGV is on the edge (see Section [6.11.2 Traversal of nodes and entering/leaving edges](#6112-traversal-of-nodes-and-enteringleaving-edges-triggering-of-actions)).

Actions that are triggered on nodes can run as long as they need to run and should be self-terminating (e.g., an audio signal that lasts for five seconds or a pick action, that is finished after picking up a load) or formulated pairwise (e.g., "activateWarningLights" and "deactivateWarningLights"), although there may be exceptions.

The following section presents predefined actions that shall be used by the AGV, if the AGV's capabilities map to the action description.
If there is a sensible way to use the defined parameters, they shall be used.
Additional parameters can be defined, if they are needed to execute an action successfully.

If there is no way to map some action to one of the actions of the following section, the AGV manufacturer can define additional actions that shall be used by master control.


### 6.8.1 Definition, parameters, effects and scope of predefined actions

general | | scope
:---:|--- | :---:
action, counter action, description, idempotent, parameters | linked state | instant, node, edge

action | counter action | description | idempotent | parameters | linked state | instant | node | edge
---|---|---|---|---|---|---|---|---
startPause | stopPause | Activates the pause mode. <br>A linked state is required, because many AGVs can be paused by using a hardware switch. <br>No more AGV driving movements - reaching next node is not necessary.<br>Actions can continue. <br>Order is resumable. | yes | - | paused | yes | no | no
stopPause | startPause | Deactivates the pause mode. <br>Movement and all other actions will be resumed (if any).<br>A linked state is required because many AGVs can be paused by using a hardware switch. <br>stopPause can also restart vehicles that were stopped with a hardware button that triggered startPause (if configured). | yes | - | paused | yes | no | no
startCharging | stopCharging | Activates the charging process. <br>Charging can be done on a charging spot (vehicle standing) or on a charging lane (while driving). <br>Protection against overcharging is responsibility of the vehicle. | yes | - | .batteryState.charging | yes | yes | no
stopCharging | startCharging | Deactivates the charging process to send a new order. <br>The charging process can also be interrupted by the vehicle / charging station, e.g., if the battery is full. <br>Battery state is only allowed to be "false", when the AGV is ready to receive orders. | yes | - |.batteryState.charging | yes | yes | no
initPosition | - | Resets (overrides) the pose of the AGV with the given parameters. | yes | x (float64)<br>y (float64)<br>theta (float64)<br>mapId (string)<br>lastNodeId (string) | .agvPosition.x<br>.agvPosition.y<br>.agvPosition.theta<br>.agvPosition.mapId<br>.lastNodeId<br>.maps | yes | yes<br>(Elevator) | no
enableMap | - | Enable a previously downloaded map explicitly to be used in orders without initializing a new position. | yes | mapId (string)<br>mapVersion (string) | .maps | yes | yes | no
downloadMap | - | Trigger the download of a new map. Active during the download. Errors reported in vehicle state. Finished after verifying the successful download, preparing the map for use and setting the map in the state. | yes | mapId (string)<br>mapVersion (string)<br>mapDownloadLink (string)<br>mapHash (string, optional) | .maps | yes | no | no
deleteMap | - | Trigger the removal of a map from the vehicle memory. | yes | mapId (string)<br>mapVersion (string) | .maps | yes | no | no
stateRequest | - | Requests the AGV to send a new state report. | yes | - | - | yes | no | no
logReport | - | Requests the AGV to generate and store a log report. | yes | reason<br>(string) | - | yes | no | no
pick | drop<br><br>(if automated) | Request the AGV to pick a load. <br>AGVs with multiple load handling devices can process multiple pick operations in parallel. <br>In this case, the parameter lhd needs to be present (e.g., LHD1). <br>The parameter stationType informs how the pick operation is handled in detail (e.g., floor location, rack location, passive conveyor, active conveyor, etc.). <br>The load type informs about the load unit and can be used to switch field for example (e.g., EPAL, INDU, etc). <br>For preparing the load handling device (e.g., pre-lift operations based on the height parameter), the action could be announced in the horizon in advance. <br>But, pre-Lift operations, etc., are not reported as 'RUNNING' in the AGV state, because the associated node is not released yet.<br>If on an edge, the vehicle can use its sensing device to detect the position for picking the node. | no |lhd (string, optional)<br>stationType (string)<br>stationName(string, optional)<br>loadType (string) <br>loadId(string, optional)<br>height (float64) (optional)<br>defines bottom of the load related to the floor<br>depth (float64) (optional) for forklifts<br>side(string) (optional) e.g., conveyor | .load | no | yes | yes
drop | pick<br><br>(if automated) | Request the AGV to drop a load. <br>See action pick for more details. | no | lhd (string, optional)<br>stationType (string, optional)<br>stationName (string, optional)<br>loadType (string, optional)<br>loadId(string, optional)<br>height (float64, optional)<br>depth (float64, optional) <br>… | .load | no | yes | yes
detectObject | - | AGV detects object (e.g., load, charging spot, free parking position). | yes | objectType(string, optional) | - | no | yes | yes
finePositioning | - | On a node, AGV will position exactly on a target.<br>The AGV is allowed to deviate from its node position.<br>On an edge, the AGV will e.g., align on stationary equipment while traversing an edge.<br>InstantAction: AGV starts positioning exactly on a target. | yes | stationType(string, optional)<br>stationName(string, optional) | - | no | yes | yes
waitForTrigger | - | AGV has to wait for a trigger on the AGV (e.g., button press, manual loading). <br>Master control is responsible to handle the timeout and has to cancel the order if necessary. | yes | triggerType(string) | - | no | yes | no
cancelOrder | - | AGV stops as soon as possible. <br>This could be immediately or on the next node. <br>Then the order is deleted. All actions are canceled. | yes | - | - | yes | no | no
factsheetRequest | - | Requests the AGV to send a factsheet | yes | - | - | yes | no | no


### 6.8.2 States of predefined actions

action | action states
---|:---:
 | | 'INITIALIZING', 'RUNNING', 'PAUSED', 'FINISHED', 'FAILED' |

action | 'INITIALIZING' | 'RUNNING' | 'PAUSED' | 'FINISHED' | 'FAILED'
---|---|---|---|---|---
startPause | - | Activation of the mode is in preparation.<br>If the AGV supports an instant transition, this state can be omitted. | - | Vehicle stands still. <br>All actions will be paused. <br>The pause mode is activated. <br>The AGV reports .paused: "true". | The pause mode cannot be activated for some reason (e.g., overridden by hardware switch).
stopPause | - | Deactivation of the mode is in preparation. <br>If the AGV supports an instant transition, this state can be omitted. | - | The pause mode is deactivated. <br>All paused actions will be resumed. <br>The AGV reports .paused: "false". | The pause mode cannot be deactivated for some reason (e.g., overwritten by hardware switch).
startCharging | - | Activation of the charging process is in progress (communication with charger is running). <br>If the AGV supports an instant transition, this state can be omitted. | - | The charging process is started. <br>The AGV reports .batteryState.charging: "true". | The charging process could not be started for some reason (e.g., not aligned to charger). Charging problems should correspond with an error.
stopCharging | - | Deactivation of the charging process is in progress (communication with charger is running). <br>If the AGV supports an instant transition, this state can be omitted. | - | The charging process is stopped. <br>The AGV reports .batteryState.charging: "false" | The charging process could not be stopped for some reason (e.g., not aligned to charger).<br> Charging problems should correspond with an error.
initPosition | - | Initializing of the new pose in progress (confidence checks, etc.). <br>If the AGV supports an instant transition, this state can be omitted. | - | The pose is reset. <br>The AGV reports <br>.agvPosition.x = x, <br>.agvPosition.y = y, <br>.agvPosition.theta = theta <br>.agvPosition.mapId = mapId <br>.agvPosition.lastNodeId = lastNodeId | The pose is not valid or cannot be reset. <br>General localization problems should correspond with an error.
| downloadMap | Initialize the connection to the map server. | AGV is downloading the map, until download is finished. | - | AGV updates its state by setting the mapId/mapVersion and the corresponding mapStatus to 'DISABLED'. | The download failed, updated in vehicle state (e.g., connection lost, Map server unreachable, mapId/mapVersion not existing on map server). |
| enableMap | - | AGV enables the map with the requested mapId and mapVersion while disabling other versions with the same mapId. | - | The AGV updates the corresponding mapStatus of the requested map to 'ENABLED' and the other versions with same mapId to 'DISABLED'. | The requested mapId/mapVersion combination does not exist.|
| deleteMap | - | AGV deletes map with requested mapId and mapVersion from its internal memory. | - | AGV removes mapId/mapVersion from its state. | Can not delete map, if map is currently in use. The requested mapId/mapVersion combination was already deleted before. |
stateRequest | - | - | - | The state has been communicated | -
logReport | - | The report is in generating. <br>If the AGV supports an instant generation, this state can be omitted. | - | The report is stored. <br>The name of the log will be reported in status. | The report can not be stored (e.g., no space).
pick | Initializing of the pick process, e.g., outstanding lift operations. | The pick process is running (AGV is moving into station, load handling device is busy, communication with station is running, etc.). | The pick process is being paused, e.g., if a safety field is violated. <br>After removing the violation, the pick process continues. | Pick is done. <br>Load has entered the AGV and AGV reports new load state. | Pick failed, e.g., station is unexpected empty. <br> Failed pick operations should correspond with an error.
drop | Initializing of the drop process, e.g., outstanding lift operations. | The drop process is running (AGV is moving into station, load handling device is busy, communication with station is running, etc.). | The drop process is being paused, e.g., if a safety field is violated. <br>After removing the violation the drop process continues. | Drop is done. <br>Load has left the AGV and AGV reports new load state. | Drop failed, e.g., station is unexpected occupied. <br>Failed drop operations should correspond with an error.
detectObject | - | Object detection is running. | - | Object has been detected. | AGV could not detect the object.
finePositioning | - | AGV positions itself exactly on a target. | The fine positioning process is being paused, e.g., if a safety field is violated. <br>After removing the violation, the fine positioning continues. | Goal position in reference to the station is reached. | Goal position in reference to the station could not be reached.
waitForTrigger | - | AGV is waiting for the trigger | - | Trigger has been triggered. | waitForTrigger fails, if order has been canceled.
cancelOrder | - | AGV is stopping or driving, until it reaches the next node. | - | AGV stands still and has canceled the order. | -
factsheetRequest | - | - | - | The factsheet has been communicated | -


## 6.9 Topic: "instantActions" (from master control to AGV)

In certain cases, it is necessary to send actions to the AGV that need to be performed immediately.
This is made possible by publishing an `instantAction` message to the topic `instantActions`.
These shall not conflict with the content of the AGV's current order (e.g., `instantAction` to lower fork, while order says to raise fork).

Some examples for which instant actions could be relevant are:
- pause the AGV without changing anything in the current order;
- resume order after pause;
- activate signal (optical, audio, etc.).

For additional information, see Section [7 Best practice](#7-best-practice).

Object structure | Data type | Description
---|---|---
headerId | uint32 | Header ID of the message.<br> The header ID is defined per topic and incremented by 1 with each sent (but not necessarily received) message.
timestamp | string | Timestamp (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ffZ (e.g., "2017-04-15T11:40:03.12Z")
version | string | Version of the protocol [Major].[Minor].[Patch] (e.g., 1.3.2).
manufacturer | string | Manufacturer of the AGV.
serialNumber | string | Serial number of the AGV.
actions [action] | array | Array of actions that need to be performed immediately and are not part of the regular order.

When an AGV receives an `instantAction`, an appropriate `actionStatus` is added to the `actionStates` array of the AGV's state.
The `actionStatus` is updated according to the progress of the action.
See also Figure 16 for the different transitions of an `actionStatus`.


## 6.10 Topic: "state" (from AGV to master control)

The AGV state will be transmitted on only one topic.
Compared to separate messages (e.g., for orders, battery state and errors) using one topic will reduce the workload of the broker and the master control for handling messages, while also keeping the information about the AGV state synchronized.

The AGV state message will be published with occurrence of relevant events or at the latest every 30s via MQTT broker to master control.

Events that trigger the transmission of the state message are:
- Receiving an order
- Receiving an order update
- Changes in the load status
- Errors or warnings
- Driving over a node
- Switching the operating mode
- Change in the `driving` field
- Change in the `nodeStates`, `edgeStates` or `actionStates`
- Change in the `maps` field

There should be an effort to curb the amount of communication.
If two events correlate with each other (e.g., the receiving of a new order usually forces an update of the `nodeStates` and `edgeStates`; as does the driving over a node), it is sensible to trigger one state update instead of multiple.


### 6.10.1 Concept and logic

The order progress is tracked by the `nodeStates` and `edgeStates`.
Additionally, if the AGV is able to derive its current position, it can publish its position via the `position` field.

If the AGV plans the path by itself, it shall communicate its calculated trajectory (including base and horizon) in the form of NURBS via the `trajectory` object in the state message, unless master control cannot use this field, and it was agreed during integration, that this field shall not be sent.
After nodes are released by master control, the AGV is not allowed to change its trajectory.

The `nodeStates` and `edgeStates` includes all nodes/edges, that the AGV still shall traverse.

![Figure 14 Order information provided by the state topic. Only the ID of the last node and the remaining nodes and edges are transmitted](./assets/order_information_state_topic.png)
>Figure 14 Order information provided by the state topic. Only the ID of the last node and the remaining nodes and edges are transmitted



### 6.10.2 Traversal of nodes and entering/leaving edges, triggering of actions

The AGV decides on its own, when a node should count as traversed.
Generally, the AGV's control point should be within the node's `allowedDeviationXY` and its orientation within `allowedDeviationTheta`.
If the edge attribute `corridor` of the subsequent edge is set, these boundaries should be met additionally.

The AGV reports the traversal of a node by removing its `nodeState` from the `nodeStates` array and setting the `lastNodeId`, `lastNodeSequenceId` to the traversed node's values.

As soon as the AGV reports the node as traversed, the AGV shall trigger the actions associated with the node, if any.

The traversal of a node also marks the leaving of the edge leading up to the node.
The edge shall then be removed from the `edgeStates` and the actions that were active on the edge shall be finished.

The traversal of the node also marks the moment, when the AGV enters the following edge, if there is one.
The edge's actions shall now be triggered.
An exception to this rule is, if the AGV has to pause on the edge (because of a soft or hard blocking edge, or otherwise) – then the AGV enters the edge after it begins moving again.

![Figure 15 Depiction of nodeStates, edgeStates, and actionStates during order handling](./assets/states_during_order_handling.png)
>Figure 15 Depiction of `nodeStates`, `edgeStates`, and `actionStates` during order handling


### 6.10.3 Base request

If the AGV detects that its base is running low, it can set the `newBaseRequest` flag to "true" to prevent unnecessary braking.


### 6.10.4 Information

The AGV can submit arbitrary additional information to master control via the `information` array.
It is up to the AGV how long it reports information via an information message.

Master control shall not use the info messages for logic, it shall only be used for visualization and debugging purposes.


### 6.10.5 Errors

The AGV reports errors via the `errors` array.
Errors have two levels: 'WARNING' and 'FATAL'.
A 'WARNING' is a self-resolving error, e.g., a field violation.
A 'FATAL' error needs human intervention.
Errors can pass references that help with finding the cause of the error via the `errorReferences` array.


### 6.10.6 Implementation of the state message

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
headerId | | uint32 | Header ID of the message.<br> The headerId is defined per topic and incremented by 1 with each sent (but not necessarily received) message.
timestamp | | string | Timestamp (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ffZ (e.g., "2017-04-15T11:40:03.12Z").
version | | string | Version of the protocol [Major].[Minor].[Patch] (e.g., 1.3.2).
manufacturer | | string | Manufacturer of the AGV.
serialNumber | | string | Serial number of the AGV.
*maps[map]* | | array | Array of map objects that are currently stored on the vehicle.
orderId| | string | Unique order identification of the current order or the previously finished order. <br>The orderId is kept until a new order is received. <br>Empty string (""), if no previous orderId is available.
orderUpdateId | | uint32 | Order update identification to identify, that an order update has been accepted by the AGV. <br>"0" if no previous orderUpdateId is available.
*zoneSetId* | |string | Unique ID of the zone set, that the AGV currently uses for path planning. <br>Shall be the same as the one used in the order.<br><br>Optional: If the AGV does not use zones, this field can be omitted.
lastNodeId | | string | Node ID of last reached node or, if the AGV is currently on a node, current node (e.g., "node7"). Empty string (""), if no `lastNodeId` is available.
lastNodeSequenceId | | uint32 | Sequence ID of the last reached node or, if the AGV is currently on a node, Sequence ID of current node. <br>"0" if no `lastNodeSequenceId` is available.
**nodeStates [nodeState]** | |array | Array of nodeState objects that need to be traversed for fulfilling the order<br>(empty array if idle)
**edgeStates [edgeState]** | |array | Array of edgeState objects that need to be traversed for fulfilling the order<br>(empty array if idle)
***agvPosition*** | | JSON object | Current position of the AGV on the map.<br><br>Optional: Can only be omitted for AGVs without the capability to localize themselves, e.g., line-guided AGVs.
***velocity*** | | JSON object | The AGV velocity in vehicle coordinates.
***loads [load]*** | | array | Loads, that are currently handled by the AGV.<br><br>Optional: If the AGV cannot determine the load state, this field shall be omitted completely and not be reported as an empty array. <br>If the AGV can determine the load state, but the array is empty, the AGV is considered unloaded.
driving | | boolean | "true": indicates, that the AGV is driving and/or rotating. Other movements of the AGV (e.g., lift movements) are not included here.<br>"false": indicates that the AGV is neither driving nor rotating.
*paused* | | boolean | "true": the AGV is currently in a paused state, either because of the push of a physical button on the AGV or because of an instantAction. <br>The AGV can resume the order.<br><br>"false": the AGV is currently not in a paused state.
*newBaseRequest* | | boolean | "true": the AGV is almost at the end of the base and will reduce speed, if no new base is transmitted. <br>Trigger for master control to send a new base.<br><br>"false": no base update required.
*distanceSinceLastNode* | meter | float64 | Used by line-guided vehicles to indicate the distance it has been driving past the lastNodeId. <br>Distance is in meters.
**actionStates [actionState]** | | array | Contains an array of all actions from the current order and all received instantActions since the last order. The action states are kept until a new order is received. Action states, except for running instant actions, are removed upon receiving a new order. <br>This may include actions from previous nodes, that are still in progress.<br><br>When an action is completed, an updated state message is published with actionStatus set to 'FINISHED' and if applicable with the corresponding resultDescription.
**batteryState** | | JSON object | Contains all battery-related information.
operatingMode | | string | Enum {'AUTOMATIC', 'SEMIAUTOMATIC', 'MANUAL', 'SERVICE', 'TEACHIN'}<br>For additional information, see Table 1 in Section [6.10.6 Implementation of the state message](#6106-implementation-of-the-state-message).
**errors [error]** | | array | Array of error objects. <br>All active errors of the AGV should be in the array.<br>An empty array indicates that the AGV has no active errors.
***information [info]*** | | array | Array of info objects. <br>An empty array indicates, that the AGV has no information. <br>This should only be used for visualization or debugging – it shall not be used for logic in master control.
**safetyState** | | JSON object | Contains all safety-related information.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**map**{ | | JSON object|
mapId | | string | ID of the map describing a defined area of the vehicle's workspace.
mapVersion | | string | Version of the map.
*mapDescription* | | string | Additional information on the map.
mapStatus <br>}| | string | Enum {'ENABLED', 'DISABLED'}<br>'ENABLED': Indicates this map is currently active / used on the AGV. At most one map with the same mapId can have its status set to 'ENABLED'.<br>'DISABLED': Indicates this map version is currently not enabled on the AGV and thus could be enabled or deleted by request.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**nodeState** { | JSON object | |
nodeId | | string | Unique node identification.
sequenceId | | uint32 | sequence ID to discern multiple nodes with same nodeId.
*nodeDescription* | | string | Additional information on the node.
released| | boolean | "true" indicates that the node is part of the base.<br>"false" indicates that the node is part of the horizon.
***nodePosition***<br><br>}| | JSON object | Node position. <br>The object is defined in Section [6.6 Topic "order"](#66-topic-order-from-master-control-to-agv) <br>Optional: <br>Master control has this information. <br>Can be sent additionally, e.g., for debugging purposes.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**edgeState** { | | JSON object | |
edgeId | | string | Unique edge identification.
sequenceId | | uint32 | sequence ID to differentiate between multiple edges with the same edgeId.
*edgeDescription* | | string | Additional information on the edge.
released | | boolean | "true" indicates that the edge is part of the base.<br>"false" indicates that the edge is part of the horizon.
***trajectory*** <br><br>} | | JSON object | The trajectory is to be communicated as NURBS and is defined in Section [6.6.6 Implementation of the order message](#666-implementation-of-the-order-message)<br><br>Trajectory segments start from the point, where the vehicle enters the edge, and terminate at the point, where the vehicle reports that the end node was traversed.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**agvPosition** { | | JSON object | Defines the position on a map in world coordinates. Each floor has its own map.
positionInitialized | | boolean | "true": position is initialized.<br>"false": position is not initialized.
*localizationScore* | | float64 | Range: [0.0 ... 1.0]<br><br>Describes the quality of the localization and can therefore be used, e.g., by SLAM AGVs to describe how accurate the current position information is.<br><br>0.0: position unknown<br>1.0: position known<br><br>Optional for vehicles that cannot estimate their localization score.<br><br>Only for logging and visualization purposes.
*deviationRange* | m | float64 | Value for the deviation range of the position in meters.<br><br>Optional for vehicles that cannot estimate their deviation e.g., grid-based localization.<br><br>Only for logging and visualization purposes.
x | m | float64 | X-position on the map in reference to the map coordinate system. <br>Precision is up to the specific implementation.
y | m | float64 | Y-position on the map in reference to the map coordinate system. <br>Precision is up to the specific implementation.
theta | | float64 | Range: [-Pi ... Pi]<br><br>Orientation of the AGV.
mapId | | string | Unique identification of the map in which the position is referenced.<br><br>Each map has the same origin of coordinates.<br>When an AGV uses an elevator from a departure floor to a destination floor, it leaves the map of the departure floor and spawns on the corresponding elevator node on the map of the destination floor.
*mapDescription*<br>} | | string | Additional information on the map.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**velocity** { | | JSON object |
*vx* | m/s | float64 | The AGV's velocity in its X-direction.
*vy* | m/s | float64 | The AGV's velocity in its Y-direction.
*omega*<br>}| Rad/s | float64 | The AGV's turning speed around its Z-axis.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**load** { | | JSON object |
*loadId* | | string | Unique identification of the load (e.g., barcode or RFID).<br><br>Empty field, if the AGV can identify the load but didn't identify the load yet.<br><br>Optional if the AGV cannot identify the load.
*loadType* | | string | Type of load.
*loadPosition* | | string | Indicates, which load handling/carrying unit of the AGV is used, e.g., in case the AGV has multiple spots/positions to carry loads.<br><br>For example: "front", "back", "positionC1", etc.<br><br>Optional for vehicles with only one loadPosition
***boundingBoxReference*** | | JSON object | Point of reference for the location of the bounding box. <br>The point of reference is always the center of the bounding box's bottom surface (at height = 0) and is described in coordinates of the AGV's coordinate system.
***loadDimensions*** | | JSON object | Dimensions of the load's bounding box in meters.
*weight*<br>} | kg | float64 | Range: [0.0 ... float64.max]<br><br>Absolute weight of the load measured in kg.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**boundingBoxReference** { | | JSON object | Point of reference for the location of the bounding box. <br>The point of reference is always the center of the bounding box's bottom surface (at height = 0) and is described in coordinates of the AGV's coordinate system.
x | | float64 | X-coordinate of the point of reference.
y | | float64 | Y-coordinate of the point of reference.
z | | float 64 | Z-coordinate of the point of reference.
*theta*<br> } | | float64 | Orientation of the loads bounding box. <br>Important for tuggers, trains, etc.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**loadDimensions** { | | JSON object | Dimensions of the load's bounding box in meters.
length | m | float64 | Absolute length of the load's bounding box.
width | m | float64 | Absolute width of the load's bounding box.
*height* <br>}| m | float64 | Absolute height of the load's bounding box.<br><br>Optional:<br><br>Set value only if known.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**actionState** { | | JSON object |
actionId | |string | Unique identifier of the action.
*actionType* | | string | Type of the action.<br><br>Optional: Only for informational or visualization purposes. Master control is aware of action type as dispatched in the order.
*actionDescription* | | string | Additional information on the current action.
actionStatus | | string | Enum {'WAITING', 'INITIALIZING', 'RUNNING', 'PAUSED', 'FINISHED', 'FAILED'}<br><br>See Section [6.11 actionStates](#611-actionstates).
*resultDescription*<br>} | | string | Description of the result, e.g., the result of an RFID reading.<br><br>Errors will be transmitted in errors.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**batteryState** { | | JSON object | 
batteryCharge | % | float64 | State of Charge: <br> if AGV only provides values for good or bad battery levels, these will be indicated as 20% (bad) and 80% (good). 
*batteryVoltage* | V | float64 | Battery voltage.
*batteryHealth* | % | int8 | Range: [0 ... 100]<br><br>State describing the battery's health. 
charging | | boolean | “true”: charging in progress.<br>“false”: the AGV is currently not charging.
*reach* <br>}| m | uint32 | Range: [0 ... uint32.max]<br><br>Estimated reach with current state of charge. 

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**error** { | | JSON object |
errorType | | string | Type/name of error
***errorReferences [errorReference]*** | | array | Array of references (e.g., nodeId, edgeId, orderId, actionId, etc.) to provide more information related to the error.<br>For additional information see [7 Best practice](#7-best-practice).
*errorDescription* | | string | Verbose description providing details and possible causes of the error.
*errorHint* | | string | Hint on how to approach or solve the reported error.
errorLevel <br> }| | string | Enum {'WARNING', 'FATAL'}<br><br>'WARNING': the AGV is ready to start (e.g., maintenance cycle expiration warning).<br>'FATAL': the AGV is not in running condition, user intervention required (e.g., laser scanner is contaminated).

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**errorReference** { | | JSON object |
referenceKey | | string | Specifies the type of reference used (e.g., nodeId, edgeId, orderId, actionId, etc.).
referenceValue <br>} | | string | The value that belongs to the reference key. For example, the ID of the node where the error occurred.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**info** { | | JSON object |
infoType | | string | Type/name of information.
*infoReferences [infoReference]* | | array | Array of references.
*infoDescription* | | string | Description of the information.
infoLevel <br>}| | string | Enum {'DEBUG', 'INFO'}<br><br>'DEBUG': used for debugging.<br> 'INFO': used for visualization.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**infoReference** { | | JSON object |
referenceKey | | string | References the type of reference (e.g., headerId, orderId, actionId, etc.).
referenceValue <br>} | | string | References the value, which belongs to the reference key.

オブジェクト構造 | 単位 | データ型 | 説明
---|---|---|---
**safetyState** { | | JSON object |
eStop | | string | Enum {'AUTOACK', 'MANUAL', 'REMOTE', 'NONE'}<br><br>Acknowledge-Type of eStop:<br>'AUTOACK': auto-acknowledgeable e-stop is activated, e.g., by bumper or protective field.<br>'MANUAL': e-stop hast to be acknowledged manually at the vehicle.<br>'REMOTE': facility e-stop has to be acknowledged remotely.<br>'NONE': no e-stop activated.
fieldViolation<br>} | | boolean | Protective field violation.<br>"true":field is violated<br>"false":field is not violated.

#### Operating Mode Description
The following description lists the operatingMode of the topic "state".

Identifier | Description
---|---
AUTOMATIC | AGV is under full control of the master control. <br>AGV drives and executes actions based on orders from the master control.
SEMIAUTOMATIC | AGV is under control of the master control.<br> AGV drives and executes actions based on orders from the master control. <br>The driving speed is controlled by the HMI (speed can't exceed the speed of automatic mode).<br>The steering is under automatic control (non-safe HMI possible).
MANUAL | Master control is not in control of the AGV. <br>Supervisor doesn't send driving order or actions to the AGV. <br>HMI can be used to control the steering and velocity and handling device of the AGV. <br>Location of the AGV is sent to the master control. <br>When the AGV enters or leaves this mode, it immediately clears all the orders (safe HMI required).
SERVICE | Master control is not in control of the AGV. <br>Master control doesn't send driving order or actions to the AGV. <br>Authorized personnel can reconfigure the AGV.
TEACHIN | Master control is not in control of the AGV. <br>Supervisor doesn't send driving order or actions to the AGV. <br>The AGV is being taught, e.g., mapping is done by a master control.

>Table 1 The operating modes and their meaning


## 6.11 Action states

When an AGV receives an `action` (either attached to a `node` or `edge` or via an `instantAction`), it shall represent this `action` with an `actionState` in its `actionStates` array.

`actionStates` describe in the field `actionStatus` at which stage of the action's life cycle the action is.

Table 2 describes, which value the enum `actionStatus` can hold.

actionStatus | Description
---|---
'WAITING' | Action was received by the AGV but the node where it triggers was not yet reached or the edge where it is active was not yet entered.
'INITIALIZING' | Action was triggered, preparatory measures are initiated.
'RUNNING' | The action is running.
'PAUSED' | The action is paused because of a pause instantAction or external trigger (pause button on the AGV)
'FINISHED' | The action is finished. <br>A result is reported via the resultDescription.
'FAILED' | Action could not be finished for whatever reason.

>Table 2 The acceptable values for the actionStatus field

A state transition diagram is provided in Figure 16.

![Figure 16 All possible status transitions for actionStates](./assets/action_state_transition.png)
>Figure 16 All possible status transitions for actionStates


## 6.12 Action blocking types and sequence

The order of multiple actions in a list define the sequence, in which those actions are to be executed.
The parallel execution of actions is governed by their respective `blockingType`.

Actions can have three distinct blocking types, described in Table 3.

blockingType | Description
---|---
NONE | Action can be executed in parallel with other actions and while the vehicle is driving.
SOFT | Action can be executed in parallel with other actions. Vehicle shall not drive.
HARD | Action shall not be executed in parallel with other actions. Vehicle shall not drive.

>Table 3 Action blocking types

If there are multiple actions on the same node with different blocking types, Figure 17 describes how the AGV should handle these actions.

![Figure 17 Handling multiple actions](./assets/handling_multiple_actions.png)
>Figure 17 Handling multiple actions


## 6.13 Topic "visualization"

For a near real-time position update the AGV can broadcast its position and velocity on the topic `visualization`.

The structure of the position object is the same as the position and velocity object in the state.
For additional information see Section [6.10.6 Implementation of the state message](#6106-implementation-of-the-state-message) for the vehicle state.
The update rate for this topic is defined by the integrator.


## 6.14 Topic "connection"

During the connection of an AGV client to the broker, a last will topic and message can be set, which is published by the broker upon disconnection of the AGV client from the broker.
Thus, the master control can detect a disconnection event by subscribing the connection topics of all AGVs.
The disconnection is detected via a heartbeat that is exchanged between the broker and the client.
The interval is configurable in most brokers and should be set around 15 seconds.
The Quality of Service level for the `connection` topic shall be 1 - At Least Once.

The suggested last will topic structure is:

**uagv/v2/manufacturer/SN/connection**

The last will message is defined as a JSON encapsulated message with the following fields:

Identifier | Data type | Description
---|---|---
headerId | uint32 | Header ID of the message. <br>The headerId is defined per topic and incremented by 1 with each sent (but not necessarily received) message.
timestamp | string | Timestamp (ISO8601, UTC); YYYY-MM-DDTHH:mm:ss.ffZ(e.g., "2017-04-15T11:40:03.12Z").
version | string | Version of the protocol [Major].[Minor].[Patch] (e.g., 1.3.2).
manufacturer | string | Manufacturer of the AGV.
serialNumber | string | Serial number of the AGV.
connectionState | string | Enum {'ONLINE', 'OFFLINE', 'CONNECTIONBROKEN'}<br><br>'ONLINE': connection between AGV and broker is active.<br><br>'OFFLINE': connection between AGV and broker has gone offline in a coordinated way. <br><br> 'CONNECTIONBROKEN': the connection between AGV and broker has unexpectedly ended.

The last will message will not be sent, when a connection is ended in a graceful way by using an MQTT disconnection command.
The last will message is only sent by the broker, if the connection is unexpectedly interrupted.

**Note**: Due to the nature of the last will feature in MQTT, the last will message is defined during the connection phase between the AGV and the MQTT broker.
As a result, the timestamp and headerId fields will always be outdated.

AGV wants to disconnect gracefully:

1. AGV sends "uagv/v2/manufacturer/SN/connection" with `connectionState` set to `OFFLINE`.
2. Disconnect the MQTT connection with a disconnect command.

AGV comes online:

1. Set the last will to "uagv/v2/manufacturer/SN/connection" with the field `connectionState` set to `CONNECTIONBROKEN`, when the MQTT connection is created.
2. Send the topic "uagv/v2/manufacturer/SN/connection" with `connectionState` set to `ONLINE`.

All messages on this topic shall be sent with a retained flag.

When connection between the AGV and the broker stops unexpectedly, the broker will send the last will topic: "uagv/v2/manufacturer/SN/connection" with the field `connectionState` set to `CONNECTIONBROKEN`.


## 6.15 Topic "factsheet"

The factsheet provides basic information about a specific AGV type series.
This information allows comparison of different AGV types and can be applied for the planning, dimensioning, and simulation of an AGV system.
The factsheet also includes information about AGV communication interfaces which are required for the integration of an AGV type series into a VDA-5050-compliant master control.

The values for some fields in the AGV factsheet can only be specified during system integration, for example the assignment of project-specific load and station types, together with the list of station and load types which are supported by this AGV.

The factsheet is intended as both a human-readable document and for machine processing, e.g., an import by the master control application, and thus is specified as a JSON document.

The master control can request the factsheet from the AGV by sending the instant action `factsheetRequest`.

All messages on this topic shall be sent with a retained flag.


### 6.15.1 Factsheet JSON structure
The factsheet consists of the JSON objects listed in the following table.

| **フィールド** | **データタイプ** | **説明** |
| --- | --- | --- |
| headerId | uint32 | Header ID of the message. <br>The headerId is defined per topic and incremented by 1 with each sent (but not necessarily received) message. |
| timestamp | string | Timestamp (ISO8601, UTC); YYYY-MM-DDTHH:mm:ss.ffZ(e.g., "2017-04-15T11:40:03.12Z"). |
| version | string | Version of the protocol [Major].[Minor].[Patch] (e.g., 1.3.2). |
| manufacturer | string | Manufacturer of the AGV. |
| serialNumber | string | Serial number of the AGV. |
| **typeSpecification** | JSON object | These parameters generally specify the class and the capabilities of the AGV. |
| **physicalParameters** | JSON object | These parameters specify the basic physical properties of the AGV. |
| **protocolLimits** | JSON object | Limits for length of identifiers, arrays, strings, and similar in MQTT communication. |
| **protocolFeatures** | JSON object | Supported features of VDA5050 protocol. |
| **agvGeometry** | JSON object | Detailed definition of AGV geometry. |
| **loadSpecification** | JSON object | Abstract specification of load capabilities. |
| ***vehicleConfig*** | JSON object | Summary of current software and hardware versions on the vehicle and optional network information. |

#### typeSpecification

This JSON object describes general properties of the AGV type.

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
| seriesName | string | Free text generalized series name as specified by manufacturer. |
| *seriesDescription* | string | Free text human-readable description of the AGV type series. |
| agvKinematic | string | Simplified description of the AGV kinematics type.<br/> [DIFF, OMNI, THREEWHEEL]<br/>DIFF: differential drive,<br/>OMNI: omni-directional vehicle,<br/>THREEWHEEL: three-wheel-driven vehicle or vehicle with similar kinematics. |
| agvClass | string | Simplified description of the AGV class.<br/>[FORKLIFT, CONVEYOR, TUGGER, CARRIER]<br/>FORKLIFT: forklift,<br/>CONVEYOR: AGV with conveyors on it,</br>TUGGER: tugger,<br/>CARRIER: load carrier with or without lifting unit. |
| maxLoadMass | float64 | [kg], Maximum loadable mass. |
| localizationTypes | array of string | Simplified description of localization type.<br/>Example values:<br/>NATURAL: natural landmarks,<br/>REFLECTOR: laser reflectors,<br/>RFID: RFID tags,<br/>DMC: data matrix code,<br/>SPOT: magnetic spots,<br/>GRID: magnetic grid.<br/>
| navigationTypes | array of string | Array of path planning types supported by the AGV, sorted by priority.<br/>Example values:<br/>PHYSICAL_LINE_GUIDED: no path planning, the AGV follows physical installed paths,<br/>VIRTUAL_LINE_GUIDED: the AGV follows fixed (virtual) paths,<br/>AUTONOMOUS: the AGV plans its path autonomously.|

#### physicalParameters

This JSON object describes physical properties of the AGV.

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
| speedMin | float64 | [m/s] Minimal controlled continuous speed of the AGV. |
| speedMax | float64 | [m/s] Maximum speed of the AGV. |
| *angularSpeedMin* | float64 | [Rad/s] Minimal controlled continuous rotation speed of the AGV. |
| *angularSpeedMax* | float64 | [Rad/s] Maximum rotation speed of the AGV. |
| accelerationMax | float64 | [m/s²] Maximum acceleration with maximum load. |
| decelerationMax | float64 | [m/s²] Maximum deceleration with maximum load. |
| heightMin | float64 | [m] Minimum height of the AGV. |
| heightMax | float64 | [m] Maximum height of the AGV. |
| width | float64 | [m] Width of the AGV. |
| length | float64 | [m] Length of the AGV. |

#### protocolLimits

This JSON object describes the protocol limitations of the AGV.
If a parameter is not defined or set to zero then there is no explicit limit for this parameter.

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
| **maxStringLens** { | JSON object | Maximum lengths of strings. |
| &emsp;*msgLen* | uint32 | Maximum MQTT message length. |
| &emsp;*topicSerialLen* | uint32 | Maximum length of serial number part in MQTT-topics.<br/><br/>Affected parameters:<br/>order.serialNumber<br/>instantActions.serialNumber<br/>state.SerialNumber<br/>visualization.serialNumber<br/>connection.serialNumber |
| &emsp;*topicElemLen* | uint32 | Maximum length of all other parts in MQTT topics.<br/><br/>Affected parameters:<br/>order.timestamp<br/>order.version<br/>order.manufacturer<br/>instantActions.timestamp<br/>instantActions.version<br/>instantActions.manufacturer<br/>state.timestamp<br/>state.version<br/>state.manufacturer<br/>visualization.timestamp<br/>visualization.version<br/>visualization.manufacturer<br/>connection.timestamp<br/>connection.version<br/>connection.manufacturer |
| &emsp;*idLen* | uint32 | Maximum length of ID strings.<br/><br/>Affected parameters:<br/>order.orderId<br/>order.zoneSetId<br/>node.nodeId<br/>nodePosition.mapId<br/>action.actionId<br/>edge.edgeId<br/>edge.startNodeId<br/>edge.endNodeId |
| &emsp;*idNumericalOnly* | boolean | If "true" ID strings need to contain numerical values only. |
| &emsp;*enumLen* | uint32 | Maximum length of enum and key strings.<br/><br/>Affected parameters:<br/>action.actionType action.blockingType<br/>edge.direction<br/>actionParameter.key<br/>state.operatingMode<br/>load.loadPosition<br/>load.loadType<br/>actionState.actionStatus<br/>error.errorType<br/>error.errorLevel<br/>errorReference.referenceKey<br/>info.infoType<br/>info.infoLevel<br/>safetyState.eStop<br/>connection.connectionState |
| &emsp;*loadIdLen* | uint32 | Maximum length of loadId strings. |
| } | | |
| **maxArrayLens** { | JSON object | Maximum lengths of arrays. |
| &emsp;*order.nodes* | uint32 | Maximum number of nodes per order processable by the AGV. |
| &emsp;*order.edges* | uint32 | Maximum number of edges per order processable by the AGV. |
| &emsp;*node.actions* | uint32 | Maximum number of actions per node processable by the AGV. |
| &emsp;*edge.actions* | uint32 | Maximum number of actions per edge processable by the AGV. |
| &emsp;*actions.actionsParameters* | uint32 | Maximum number of parameters per action processable by the AGV. |
| &emsp;*instantActions* | uint32 | Maximum number of instant actions per message processable by the AGV. |
| &emsp;*trajectory.knotVector* | uint32 | Maximum number of knots per trajectory processable by the AGV. |
| &emsp;*trajectory.controlPoints* | uint32 | Maximum number of control points per trajectory processable by the AGV. |
| &emsp;*state.nodeStates* | uint32 | Maximum number of nodeStates sent by the AGV, maximum number of nodes in base of AGV. |
| &emsp;*state.edgeStates* | uint32 | Maximum number of edgeStates sent by the AGV, maximum number of edges in base of AGV. |
| &emsp;*state.loads* | uint32 | Maximum number of load objects sent by the AGV. |
| &emsp;*state.actionStates* | uint32 | Maximum number of actionStates sent by the AGV. |
| &emsp;*state.errors* | uint32 | Maximum number of errors sent by the AGV in one state message. |
| &emsp;*state.information* | uint32 | Maximum number of information sent by the AGV in one state message. |
| &emsp;*error.errorReferences* | uint32 | Maximum number of error references sent by the AGV for each error. |
| &emsp;*information.infoReferences* | uint32 | Maximum number of info references sent by the AGV for each information. |
| } | | |
| **timing** { | JSON object | Timing information. |
| &emsp;minOrderInterval | float32 | [s], Minimum interval sending order messages to the AGV. |
| &emsp;minStateInterval | float32 | [s], Minimum interval for sending state messages. |
| &emsp;*defaultStateInterval* | float32 | [s], Default interval for sending state messages, *if not defined, the default value from the main document is used*. |
| &emsp;*visualizationInterval* | float32 | [s], Default interval for sending messages on visualization topic. |
| } | | |

#### protocolFeatures

This JSON object defines actions and parameters which are supported by the AGV.

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
| **optionalParameters** [**optionalParameter**] | array of JSON object | Array of supported and/or required optional parameters.<br/>Optional parameters that are not listed here are assumed to be not supported by the AGV. |
| { | | |
| &emsp;parameter | string | Full name of optional parameter, e.g., "*order.nodes.nodePosition.allowedDeviationTheta"*.|
| &emsp;support | enum | Type of support for the optional parameter, the following values are possible:<br/>'SUPPORTED': optional parameter is supported like specified.<br/>'REQUIRED': optional parameter is required for proper AGV operation. |
| &emsp;*description*| string | Free-form text: description of optional parameter, e.g., <ul><li>Reason, why the optional parameter direction is necessary for this AGV type and which values it can contain.</li><li>The parameter nodeMarker shall contain unsigned integers only.</li><li>NURBS support is limited to straight lines and circle segments.</li>|
| } | | |
| **agvActions** [**agvAction**] | array of JSON object | Array of all actions with parameters supported by this AGV. This includes standard actions specified in VDA5050 and manufacturer-specific actions. |
| { | | |
| &emsp;actionType | string | Unique type of action corresponding to action.actionType. |
| &emsp;*actionDescription* | string | Free-form text: description of the action. |
| &emsp;actionScopes | array of enum | Array of allowed scopes for using this action type.<br/><br/>'INSTANT': usable as instantAction.<br/>'NODE': usable on nodes.<br/>'EDGE': usable on edges.<br/><br/>For example: ['INSTANT', 'NODE']|
| &emsp;***actionParameters** [**actionParameter**]* | array of JSON object | Array of parameters an action has.<br/>If not defined, the action has no parameters.<br/> The JSON object defined here is a different JSON object than the one used in Section [6.6.6 Implementation of the order message](#666-implementation-of-the-order-message) within nodes and edges.|
|&emsp;*{* | | |
|&emsp;&emsp;key | string | Key string for parameter. |
|&emsp;&emsp;valueDataType | enum | Data type of value, possible data types are: 'BOOL', 'NUMBER', 'INTEGER', 'FLOAT', 'STRING', 'OBJECT', 'ARRAY'. |
|&emsp;&emsp;*description* | string | Free-form text: description of the parameter. |
|&emsp;&emsp;*isOptional* | boolean | "true": optional parameter. |
|&emsp;*}* | | |
|*resultDescription* | string | Free-form text: description of the result. |
|*blockingTypes* | array of enum | Array of possible blocking types for defined action. </br> Enum {'NONE', 'SOFT', 'HARD'} |
|*}* | | |

### agvGeometry

This JSON object defines the geometry properties of the AGV, e.g., outlines and wheel positions.

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
| ***wheelDefinitions** [**wheelDefinition**]* | array of JSON object | Array of wheels, containing wheel arrangement and geometry. |
| { | | |
| &emsp;type | enum | Wheel type<br/> Enum {'DRIVE', 'CASTER', 'FIXED', 'MECANUM'}. |
| &emsp;isActiveDriven | boolean | "true": wheel is actively driven. |
| &emsp;isActiveSteered | boolean | "true": wheel is actively steered. |
| &emsp;**position** { | JSON object | |
|&emsp;&emsp; x | float64 | [m], x-position in AGV coordinate system. |
|&emsp;&emsp; y | float64 | [m], y-position in AGV coordinate system. |
|&emsp;&emsp; *theta* | float64 | [rad], orientation of the wheel in AGV coordinate system. Necessary for fixed wheels. |
| &emsp;} | | |
| &emsp;diameter | float64 | [m], nominal diameter of wheel. |
| &emsp;width | float64 | [m], nominal width of wheel. |
| &emsp;*centerDisplacement* | float64 | [m], nominal displacement of the wheel's center to the rotation point (necessary for caster wheels).<br/> If the parameter is not defined, it is assumed to be 0. |
| &emsp;*constraints* | string | Free-form text: can be used by the manufacturer to define constraints. |
| } | | |
| ***envelopes2d** [**envelope2d**]* | array of JSON object | Array of AGV envelope curves in 2D, e.g., the mechanical envelopes for unloaded and loaded state, the safety fields for different speed cases. |
| { | | |
| &emsp;set | string | Name of the envelope curve set. |
| &emsp;**polygonPoints** **[polygonPoint]** | array of JSON object | Envelope curve as an x/y-polygon polygon is assumed as closed and shall be non-self-intersecting. |
| &emsp;{ | | |
|&emsp;&emsp; x | float64 | [m], X-position of polygon point. |
|&emsp;&emsp; y | float64 | [m], Y-position of polygon point. |
| &emsp;} | | |
| &emsp;*description* | string | Free-form text: description of envelope curve set. |
| *}* | | |
| ***envelopes3d [envelope3d]*** | array of JSON object | Array of AGV envelope curves in 3D. |
| *{* | | |
| &emsp;set | string | Name of the envelope curve set. |
| &emsp;format | string | Format of data, e.g., DXF. |
| &emsp;***data*** | JSON object | 3D-envelope curve data, format specified in 'format'. |
| &emsp;*url* | string | Protocol and URL definition for downloading the 3D-envelope curve data, e.g., <ftp://xxx.yyy.com/ac4dgvhoif5tghji>. |
| &emsp;*description* | string | Free-form text: description of envelope curve set |
| *}* | | |

#### loadSpecification

This JSON object specifies load handling and supported load types of the AGV.

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
| *loadPositions* | array of string | Array of load positions / load handling devices.<br/>This array contains the valid values for the parameter "state.loads[].loadPosition" and for the action parameter "lhd" of the actions pick and drop.<br/>*If this array doesn't exist or is empty, the AGV has no load handling device.* |
| ***loadSets [loadSet]*** | array of JSON object | Array of load sets that can be handled by the AGV |
| { | | |
|&emsp; setName | string | Unique name of the load set, e.g., DEFAULT, SET1, etc. |
|&emsp; loadType | string | Type of load, e.g., EPAL, XLT1200, etc. |
|&emsp; *loadPositions* | array of string | Array of load positions btw. load handling devices, this load set is valid for.<br/>*If this parameter does not exist or is empty, this load set is valid for all load handling devices on this AGV.* |
|&emsp; ***boundingBoxReference*** | JSON object | Bounding box reference as defined in parameter loads[] in state message. |
|&emsp; ***loadDimensions*** | JSON object | Load dimensions as defined in parameter loads[] in state message. |
|&emsp; *maxWeight* | float64 | [kg], maximum weight of load type. |
|&emsp; *minLoadhandlingHeight* | float64 | [m], minimum allowed height for handling of this load type and weight<br/>references to boundingBoxReference. |
|&emsp; *maxLoadhandlingHeight* | float64 | [m], maximum allowed height for handling of this load type and weight<br/>references to boundingBoxReference. |
|&emsp; *minLoadhandlingDepth* | float64 | [m], minimum allowed depth for this load type and weight<br/>references to boundingBoxReference. |
|&emsp; *maxLoadhandlingDepth* | float64 | [m], maximum allowed depth for this load type and weight<br/>references to boundingBoxReference. |
|&emsp; *minLoadhandlingTilt* | float64 | [rad], minimum allowed tilt for this load type and weight. |
|&emsp; *maxLoadhandlingTilt* | float64 | [rad], maximum allowed tilt for this load type and weight. |
|&emsp; *agvSpeedLimit* | float64 | [m/s], maximum allowed speed for this load type and weight. |
|&emsp; *agvAccelerationLimit* | float64 | [m/s²], maximum allowed acceleration for this load type and weight. |
|&emsp; *agvDecelerationLimit* | float64 | [m/s²], maximum allowed deceleration for this load type and weight. |
|&emsp; *pickTime* | float64 | [s], approx. time for picking up the load |
|&emsp; *dropTime* | float64 | [s], approx. time for dropping the load. |
|&emsp; *description* | string | Free-form text: description of the load handling set. |
| } | | |

#### vehicleConfig

This JSON object details the software and hardware versions running on the vehicle, as well as a brief summary of network information.

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
| *versions[versionInfo]* | array of JSON object | Array of key-value pair objects containing software and hardware information.| | { | | |
|&emsp; key | string | Key of the software/hardware version used. (e.g., softwareVersion) |
|&emsp; value | string | The version corresponding to the key. (e.g., v1.12.4-beta) |
| } | | |
| *network* { | JSON object | Information about the vehicle's network connection. The listed information shall not be updated while the vehicle is operating. |
|&emsp;&emsp; *dnsServers* | array of string | Array of Domain Name Servers (DNS) used by the vehicle. |
|&emsp;&emsp; *ntpServers* | array of string | Array of Network Time Protocol (NTP) servers used by the vehicle. |
|&emsp;&emsp; *localIpAddress* | string | A priori assigned IP address used to communicate with the MQTT broker. Note that this IP address should not be modified/changed during operations. |
|&emsp;&emsp; *netmask* | string | The subnet mask used in the network configuration corresponding to the local IP address.|
|&emsp;&emsp; *defaultGateway* | string | The default gateway used by the vehicle, corresponding to the local IP address. |
| &emsp;} | | |


# 7 ベストプラクティス

このセクションには、プロトコルのロジックと併せて共通理解を促進するのに役立つ追加情報が含まれている。


## 7.1 エラー参照

誤った注文によりエラーが発生した場合、AGV はフィールド errorReferences（ステートトピックのセクション [6.10.6 ステートメッセージの実装](#6106-implementation-of-the-state-message) を参照）に意味のあるエラー参照を返す必要がある。
これには以下の情報が含まれる。

- `headerId`
- トピック (`order` または `instantAction`)
- エラーが注文の更新によって引き起こされた場合は、`orderId` および `orderUpdateId`
- エラーがアクションによって引き起こされた場合は、`actionId`
- エラーが誤ったアクションパラメータによって引き起こされた場合は、パラメータのリスト

外部要因（例えば、期待された位置で読み込みが行われないなど）によりアクションが完了できない場合、actionId を参照すること。


## 7.2 パラメータのフォーマット

エラー、情報、アクション用のパラメータは、キーと値のペアを持つJSONオブジェクトの配列として設計されています。

| **フィールド** | **データタイプ** | **説明** |
|---|---|---|
**actionParameter** { | JSON object | 指定されたアクション用のアクションパラメータ、例えば、deviceId、loadId、外部トリガー。
key | string | パラメータのキー。
value</br>} | 以下のいずれか:</br>array,</br>boolean,</br>number,</br>string,</br>object | キーに対応するパラメータの値。

ステーションタイプと負荷タイプに対するキーと値のペアを持つアクション「someAction」の「actionParameter」の例：

"actionParameters":[
{"key":"stationType", "value": "floor"},
{"key":"weight", "value": 8.5},
{"key": "loadType", "value": "pallet_eu"}
]

"key", "actualKey", "value", "actualValue"という提案されたスキームを使用する理由は、実装を汎用的に保つためです。 "actualValue"は、float、bool、さらにはオブジェクトなど、あらゆるJSONデータタイプが可能です。


# 8 用語集


## 8.1 定義

コンセプト|説明
---|---
フリーナビゲーションAGV|地図を使用して自らの経路を計画する車両。<br>マスターコントロールは、出発地と目的地の座標のみを送信する。<br>車両は、その経路をマスターコントロールに送信する。<br>マスターコントロールとの接続が切れた場合でも、車両は走行を継続できる。<br>フリーナビゲーション車両は、局所的な障害物を回避することが許可される場合がある。<br>また、受け取り/払い出し位置の微調整を車両自身が行うことも可能である。
誘導車両（物理的または仮想的）|マスターコントロールから経路情報を受け取る車両。<br>経路の計算はマスターコントロールで行われる。<br>マスターコントロールとの通信が途切れた場合、車両はリリースされたノードとエッジ（「ベース」）を終了し、停止する。<br>誘導された車両は、局所的な障害物を回避することが許可される場合がある。<br>また、受け取り/払い出し位置の微調整が車両自身によって行われる場合もある。
中央マップ|マスターコントロールで一元管理されるマップ。<br>これは最初に作成され、その後使用される。

