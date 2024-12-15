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
[6.6 Topic: "order" (from master control to AGV)](#66-topic-orderfrom-master-control-to-agv)<br>
[6.6.1 Concept and logic](#661-concept-and-logic)<br>
[6.6.2 Orders and order updates](#662-orders-and-order-update)<br>
[6.6.3 Order cancellation (by master control)](#663-order-cancellation-by-master-control)<br>
[6.6.4 Order rejection](#664-order-rejection)<br>
[6.6.5 Corridors](#665-corridors)<br>
[6.6.6 Implementation of the order message](#666-implementation-of-the-order-message)<br>
[6.7 Maps](#67-maps)<br>
[6.7.1 Map distribution](#671-map-distribution)<br>
[6.7.2 Maps in vehicle state](#672-maps-in-the-vehicle-state)<br>
[6.7.3 Map download](#673-map-download)<br>
[6.7.4 Enable downloaded maps](#674-enable-downloaded-maps)<br>
[6.7.5 Delete maps on vehicle](#675-delete-maps-on-vehicle)<br>
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
[7 Best practice](#7-best-practice)<br>
[7.1 Error reference](#71-error-reference)<br>
[7.2 Format of parameters](#72-format-of-parameters)<br>
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
- 独自仕様のマスター制御と上位のマスター制御間の垂直通信を実装することにより、独自仕様のDTS在庫システムを統合する（図1参照）。

![Figure 1 Integration of DTS inventory systems](./assets/concept_DTS.png)
>図1 DTSの在庫管理システムの統合

上記の目的を達成するために、本書では、AGVとマスター制御間の注文およびステータス情報の通信インターフェースについて説明する。

AGVとマスター制御間の動作（経路計画などにおいて特別なスキルを自由に考慮するなど）や、他のシステムコンポーネント（外部周辺機器、防火ゲートなど）との通信に必要なその他のインターフェースは、本書には初めから含まれていない。


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
これにより、誘導される車両のマスター制御を完全にカバーすることが可能となる。
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

変数がオプションとしてマークされている場合、それは送信者にとってオプションであることを意味します。なぜなら、変数は特定のケースでは適用できない可能性があるからです（例えば、マスター制御がAGVに命令を送信する場合、一部のAGVは自ら軌道を計画し、その注文の `edge` オブジェクト内の `trajectory` フィールドは省略できます）。
このプロトコルでオプションと指定されたフィールドを含むメッセージをAGVが受信した場合、AGVはそれに応じて動作することが期待され、そのフィールドを無視することはできません。

AGVがメッセージを適切に処理できない場合、期待される動作は、エラーメッセージ内でその旨を通知し、注文を拒否することです。
マスタ制御は、AGVがサポートするオプション情報のみを送信します。

例：軌道はオプションです。
AGVが軌道を処理できない場合、マスター制御は車両に軌道を送信しない。

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

AGVプロトコルでは、マスター制御とAGV間の情報交換に以下のトピックを使用する。

トピック名 | 公開者 | 購読者 | 用途 | 実装 | スキーマ
---|---|---|---|---|---
order|マスター制御|AGV|マスター制御からAGVへの運転命令の通信|必須|order.schema
instantActions | マスター制御 | AGV | 即座に実行されるアクションの通信 | 必須 |instantActions.schema
state | AGV | マスター制御 | AGVの状態の通信 | 必須 | state.schema
visualization | AGV | 視覚化システム | 視覚化のみを目的とした位置情報を有する更新頻度の高いトピック | オプション | visualization.schema
connection | ブローカー/AGV | マスター制御 | AGV接続が失われたことを示す。車両の状態を確認するためにマスター制御で使用しない。MQTTプロトコルの接続レベルチェック用に追加された | 必須 | connection.schema 
factsheet | AGV | マスター制御 | マスター制御におけるAGVのセットアップを支援するパラメータまたはベンダー固有の情報 | 必須 | factsheet.schema


## 6.6 Topic: "order" (from master control to AGV)

The topic "order" is the MQTT topic via which the AGV receives a JSON encapsulated order.


### 6.6.1 Concept and logic

The basic structure of an order is a graph of nodes and edges.
The AGV is expected to traverse the nodes and edges to fulfill the order.
The full graph of all connected nodes and edges is held by master control.

The graph representation in the master control contains restrictions, e.g., which AGV is allowed to traverse which edge.
These restrictions will not be communicated to the AGV.
The master control only includes edges in an AGV order which the concerning AGV is allowed to traverse.

![Figure 3 Graph representation in master control and graph transmitted in orders](./assets/graph_representation_transmission.png)
>Figure 3 Graph representation in master control and graph transmitted in orders

The nodes and edges are passed as two lists in the order message.
The order of the nodes and edges within those lists also governs in which sequence the nodes and edges shall be traversed.

For a valid order, there shall be at least one node and the number of edges shall be equal to the number of nodes minus one.

The first node of an order shall be trivially reachable for the AGV.
This means either that the AGV is already standing on the node, or that the AGV is in the node's deviation range.

Nodes and edges both have a boolean attribute `released`.
If a node or edge is released, the AGV is expected to traverse it.
If a node or edge is not released, the AGV shall not traverse it.

An edge can be released only if both the start and the end node of the edge are released.

After an unreleased edge, no released nodes or edges can follow in the sequence.

The set of released nodes and edges are called the "base".
The set of unreleased nodes and edges are called the "horizon".

It is valid to send an order without a horizon.

An order message does not necessarily describe the full transport order.
For traffic control and to accommodate resource constrained vehicles, the full transport order (which might consist of many nodes and edges) can be split up into many sub-orders, which are connected via their `orderId` and `orderUpdateId`.
The process of updating an order is described in the next section.


### 6.6.2 Orders and order update

To support traffic management, master control can split the path communicated via order into two parts:

- *"Base"*: This is the defined route that the AGV is allowed to travel. All nodes and edges of the base route have already been released by the master control for the vehicle. The last node of the base is called decision point.
- *"Horizon"*: This is the route currently planned by master control for the AGV to travel after the decision point. The horizon route has not yet been released by the master control.

The AGV shall stop at the decision point if no further nodes and edges are added to the base. In order to ensure a fluent movement, the master control should extend the base before the AGV reaches the decision point, if the traffic situation allows for it.

Since MQTT is an asynchronous protocol and transmission via wireless networks is not reliable, the base cannot be changed. The master control shall therefore assume that the base has already been executed by the AGV. A later section describes a procedure to cancel an order, but this is also considered unreliable due to the communication limitations mentioned above.

The master control has the possibility to change the horizon by sending an updated route to the AGV which includes the changed list of nodes and edges. The procedure for changing the horizon route is shown in Figure 4.

![Figure 4 Procedure for changing the driving route "Horizon"](./assets/driving_route_horizon.png)
>Figure 4 Procedure for changing the driving route "Horizon"

In Figure 4, an initial job is first sent by the control panel at time t = 1.
Figure 5 shows the pseudocode of a possible job.
For the sake of readability, a complete JSON example has been omitted here.

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
>Figure 5 Pseudocode of an order.

At time t = 3, the order is updated by sending an extension of the order (see example in Figure 6).
Note that the `orderUpdateId` is incremented and that the first node of the order update corresponds to the last shared base node of the previous order message.

This ensures that the AGV can also perform the order update, i.e., that the first node of the job update is reachable by executing the edges already known to the AGV.

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
>Figure 6 Pseudocode of an order update. Note the change of the `orderUpdateId`.

This also aids in the event that an order update is lost (e.g., due to an unreliable wireless network).
The AGV can always check that the last known base node has the same `nodeId` (and `nodeSequenceId`, more on that later) as the first new base node.

Also note that node g is the only base node that is sent again.
Since the base cannot be changed, a retransmission of nodes f and d is not valid.

It is important, that the contents of the stitching node (node g in the example case) are not changed.
For actions, deviation range, etc., the AGV shall use the instructions provided in the first order (Figure 5, orderUpdateId 0).

![Figure 7 Regular update process - order extension](./assets/update_order_extension.png)
>Figure 7 Regular update process - order extension.

Figure 7 describes how an order should be extended.
It shows the information that is currently available on the AGV.
The `orderId` stays the same and the `orderUpdateId` is incremented.

The last node of the previous base is the first base node in the updated order.
With this node the AGV can add the updated order onto the current order (stitching).
The other nodes and edges from the previous base are not resent.

Master control has the option to make changes to the horizon by sending entirely different nodes as the new base.
The horizon can also be deleted.

To allow loops in orders (like going from node a to b and then back to a) a `sequenceId` is assigned to the node and edge objects.
This `sequenceId` runs over the nodes and edges (first node of an order receives a 0, the first edge then gets the 1, the second node then gets the 2, and so on).
This allows for easier tracking of the order progress.

Once a `sequenceId` is assigned, it does not change with order updates (see Figure 7).
This is necessary to determine on AGV side to which node the master control refers to.

Figure 8 describes the process of accepting an order or order update.

![Figure 8 The process of accepting an order or orderUpdate](./assets/process_order_update.png)
>Figure 8 The process of accepting an order or order update.

1)	**is received order valid?**:
All formatting and JSON data types are correct?

2)	**is received order new or an update of the current order?**:
Is `orderId` of the received order different to `orderId` of order the vehicle currently holds?

3)	**is vehicle still executing an order or waiting for an update?**:
Are `nodeStates` not empty or are `actionStates` containing states which are neither 'FAILED' nor 'FINISHED'? Nodes and edges and the corresponding action states of the order horizon are also included inside the state. Vehicle might still have a horizon and therefore waiting for an update and executing an order.

4) **is start of new order close enough to current position?**:	Is the vehicle already standing on the node, or is it in the node's deviation range (see Section [6.6.1 Concept and logic](#661-concept-and-logic))?

5) **is received order update deprecated?**: Is `orderUpdateId` smaller than the one currently on the vehicle?

6)	**is received order update currently on vehicle?**: Is `orderUpdateId` equal to the one currently on the vehicle?

7)	**is the received update a valid continuation of the currently still running order?**:	Is the first node of the received order equal to the current decision point (last node of the current base)? The vehicle is still moving or executing actions related to the base released in previous order updates or still has a horizon and is therefore waiting for a continuation of the order. In this case, the order update is only accepted if the first node of the new base is equal to the last node of the previous base.

8)	**is the received update a valid continuation of the previously completed order?**: Are `nodeId` and `sequenceId` of the first node of the received order update equal to `lastNodeId` and `lastNodeSequenceId`? The vehicle is not executing any actions anymore neither is it waiting for a continuation of the order (meaning that it has completed its base with all related actions and does not have a horizon). The order update is now accepted if it continues from the last traversed node, therefore the first node of the new base needs to match the vehicle's `lastNodeId` as well as `lastNodeSequenceId`.

9)	populate/append states refers to the `actionStates`/`nodeStates`/`edgeStates`.


### 6.6.3 Order cancellation (by master control)

In the event of an unplanned change in the base nodes, the order shall be canceled by using the instantAction `cancelOrder`.

After receiving the instantAction `cancelOrder`, the vehicle stops (based on its capabilities, e.g., right where it is or on the next node).

If there are actions scheduled, these actions shall be cancelled and report 'FAILED' in their `actionState`.
If there are running actions, those actions should be cancelled and also be reported as 'FAILED'.
If the action cannot be interrupted, the `actionState` of that action should reflect that by reporting 'RUNNING' while it is running, and after that the respective state ('FINISHED', if successful and 'FAILED', if not).
While actions are running, the cancelOrder action shall report 'RUNNING', until all actions are cancelled/finished.
After all movement of the vehicle and all of its actions are stopped, the `cancelOrder` action status shall report 'FINISHED'.

The `orderId` and `orderUpdateId` are kept.

Figure 9 shows the expected behavior for different AGV capabilities.

![Figure 9 Expected behavior after a cancelOrder](./assets/process_cancel_order.png)
>Figure 9 Expected behavior after a `cancelOrder`.


#### 6.6.3.1 Receiving a new order after cancellation

After the cancellation of an order, the vehicle shall be in a state to receive a new order.

In the case of an AGV that localizes itself on nodes via a tag, the new order has to begin on the node the AGV is now standing on (see also Figure 5).

In case of an AGV that can stop in between nodes, the choice is up to master control how the next order should be started.
The AGV shall accept both methods.

There are two options:

- Send an order, where the first node is a temporary node that is positioned where the AGV currently stands. The AGV shall then realize that this node is trivially reachable and accept the order.
- Send an order, where the first node is the last traversed node of the previous order but set the deviation range so large that the AGV is within this range. Thus, the AGV shall realize that this node shall be counted as traversed and accept the order.


#### 6.6.3.2 Receiving a cancelOrder action when AGV has no order

If the AGV receives a `cancelOrder` action but, the AGV currently has no order, or the previous order was canceled, the `cancelOrder` action shall be reported as 'FAILED'.

The AGV shall report a "noOrderToCancel" error with the `errorLevel` set to 'WARNING'.
The `actionId` of the `instantAction` shall be passed as an `errorReference`.


### 6.6.4 Order rejection

There are several scenarios, when an order shall be rejected.
These scenarios are shown in Figure 8 and described below.


#### 6.6.4.1 Vehicle gets a malformed new order

Resolution:

1. Vehicle does NOT take over the new order in its internal buffer.
2. The vehicle reports the warning "validationError"
3. The warning shall be reported until the vehicle has accepted a new order.


#### 6.6.4.2 Vehicle receives an order with actions it cannot perform or with fields that it cannot use

Examples:

- Non-executable actions: lifting height higher than maximum lifting height, lifting actions although no stroke is installed, etc.
- Non-useable fields: trajectory, etc.

Resolution:

1. Vehicle does NOT take over the new order in its internal buffer
2. Vehicle reports the warning "orderError" with the wrong fields as error references
3. The warning shall be reported until the vehicle has accepted a new order.


#### 6.6.4.3 Vehicle gets a new order with the same orderId, but a lower orderUpdateId than the current orderUpdateId

Resolution:

1. Vehicle does NOT take over the new order in its internal buffer.
2. Vehicle keeps the previous order in its buffer.
3. The vehicle reports the warning "orderUpdateError"
4. The vehicle continues with executing the previous order.

If the AGV receives an order with the same `orderId` and `orderUpdateId` twice, the second order will be ignored. 
This might happen, if the master control resends the order because the state message was received too late by master control and it could therefore not verify that the first order had been received.

### 6.6.5 Corridors

The optional `corridor` edge attribute allows the vehicle to deviate from the edge trajectory for obstacle avoidance and defines the boundaries within which the vehicle is allowed to operate.
To use the `corridor` attribute, a predefined trajectory is required that the vehicle would follow if no `corridor` attribute was defined. This can be either the trajectory defined on the vehicle known to the master control or the trajectory sent in an order. The behavior of a vehicle using the `corridor` attribute is still the behavior of a line-guided vehicle, except that it's allowed to temporarily deviate from a trajectory to avoid obstacles.

*Remark:
An edge inside an order defines a logical connection between two nodes and not necessarily the (real) trajectory that a vehicle follows when driving from the start node to the end node.
Depending on the vehicle type, the trajectory that a vehicle takes between the start and end nodes is either defined by master control via the trajectory edge attribute or assigned to the vehicle as a predefined trajectory.
Depending on the internal state of the vehicle, the selected trajectory may vary.*

![Figure 10 Edges with corridor attribute.](./assets/edges_with_corridors.png)
>Figure 10 Edges with a `corridor` attribute that defines the left and right boundaries within which a vehicle is allowed to deviate from its predefined trajectory to avoid obstacles. On the left, the kinematic center defines the allowed deviation, while on the right, the contour of the vehicle, possibly extended by the load, defines the allowed deviation. This is defined by the `corridorRefPoint` parameter.

The area in which the vehicle is allowed to navigate independently (and deviate from the original edge trajectory) is defined by a left and a right boundary.
The optional `corridorRefPoint` field specifies whether the vehicle control point or the vehicle contour should be inside the defined boundary.
The boundaries of the edges shall be defined in such a way that the vehicle is inside the boundaries of the new and now current edge as soon as it passes a node.
Instead of setting the corridor boundaries to zero, master control shall not use the `corridor` attribute if the vehicle shall not deviate from the trajectory.

The vehicle's motion control software shall constantly check that the vehicle is within the defined boundaries.
If not, the vehicle shall stop because it is out of the allowed navigation space and report an error.
The master control can decide if user interaction is required or if the vehicle can continue by canceling the current order and sending a new order to the vehicle with corridor information that allows the vehicle to move again.

*Remark: Allowing the vehicle to deviate from the trajectory increases the possible footprint of the vehicle during driving. This circumstance shall be considered during initial operation, and if the master control makes a traffic control decision based on the vehicle's footprint.*

See also Section [6.10.2 Traversal of nodes and entering/leaving edges](#6102-traversal-of-nodes-and-enteringleaving-edges-triggering-of-actions) for further information.


## 6.6.6 Implementation of the order message

Object structure | Unit | Data type | Description
---|---|---|---
headerId | | uint32 | Header ID of the message.<br> The header ID is defined per topic and incremented by 1 with each sent (but not necessarily received) message.
timestamp | | string | Timestamp (ISO 8601, UTC); YYYY-MM-DDTHH:mm:ss.ffZ (e.g., "2017-04-15T11:40:03.12Z")
version | | string | Version of the protocol [Major].[Minor].[Patch] (e.g., 1.3.2)
manufacturer | | string | Manufacturer of the AGV
serialNumber | | string | Serial number of the AGV
orderId | | string | Order identification.<br> This is to be used to identify multiple order messages that belong to the same order.
orderUpdateId | | uint32 | Order update identification.<br>Is unique per orderId.<br>If an order update is rejected, this field is to be passed in the rejection message.
*zoneSetId* | | string | Unique identifier of the zone set, that the AGV has to use for navigation or that was used by master control for planning. <br> <br> Optional: Some master control systems do not use zones.<br> Some AGVs do not understand zones.<br> Do not add to message, if no zones are used.
**nodes [node]** | | array | Array of node objects to be traversed for fulfilling the order. <br>One node is enough for a valid order. <br>Leave edge array empty in that case.
**edges [edge]** | | array | Array of edge objects to be traversed for fulfilling the order. <br>One node is enough for a valid order. <br>Leave edge array empty in that case.

Object structure | Unit | Data type | Description
---|---|---|---
**node** { | | JSON object|
nodeId | | string | Unique node identification
sequenceId | | uint32 | Number to track the sequence of nodes and edges in an order and to simplify order updates. <br>The main purpose is to distinguish between a node, which is passed more than once within one orderId. <br>The variable sequenceId runs across all nodes and edges of the same order and is reset when a new orderId is issued.
*nodeDescription* | | string | Additional information on the node
released | | boolean | "true" indicates that the node is part of the base. <br> "false" indicates that the node is part of the horizon.
***nodePosition*** | | JSON object | Node position. <br>Optional for vehicle types that do not require the node position (e.g., line-guided vehicles).
**actions [action]** <br> } | | array | Array of actions to be executed on a node. <br>Empty array, if no actions required.

Object structure | Unit | Data type | Description
---| --- |--- | ---
**nodePosition** { | | JSON object | Defines the position on a map in a global project-specific world coordinate system. <br>Each floor has its own map. <br>All maps shall use the same project-specific global origin.
x | m | float64 | X-position on the map in reference to the map coordinate system. <br>Precision is up to the specific implementation.
y | m | float64 | Y-position on the map in reference to the map coordinate system. <br>Precision is up to the specific implementation.
*theta* | rad | float64 | Range: [-Pi ... Pi] <br><br>Absolute orientation of the AGV on the node.<br> Optional: vehicle can plan the path by itself.<br>If defined, the AGV has to assume the theta angle on this node.<br>If previous edge disallows rotation, the AGV shall rotate on the node.<br>If following edge has a differing orientation defined but disallows rotation, the AGV is to rotate on the node to the edges desired rotation before entering the edge.
*allowedDeviationXY* | m | float64 | Indicates how precisely an AGV shall match the position of a node for it to be considered traversed. <br><br> If = 0.0: no deviation is allowed (no deviation means within the normal tolerance of the AGV manufacturer).<br><br> If > 0.0: allowed deviation radius in meters. <br>If the AGV passes a node within the deviation radius, the node can be considered traversed.
*allowedDeviationTheta* | rad | float64 | Range: [0.0 ... Pi] <br><br> Indicates how precise the orientation defined in theta has to be met on the node by the AGV. <br>The lowest acceptable angle is theta - allowedDeviationTheta and the highest acceptable angle is theta + allowedDeviationTheta.
mapId | | string | Unique identification of the map on which the position is referenced. <br> Each map has the same project-specific global origin of coordinates. <br>When an AGV uses an elevator, e.g., leading from a departure floor to a target floor, it will disappear off the map of the departure floor and spawn in the related lift node on the map of the target floor.
*mapDescription* <br> } | | string | Additional information on the map.

Object structure | Unit | Data type | Description
---|---|---|---
**action** { | | JSON object | Describes an action that the AGV can perform.
actionType | | string | Name of action as described in the first column of "Actions and Parameters". <br> Identifies the function of the action.
actionId | | string | Unique ID to identify the action and map them to the actionState in the state. <br>Suggestion: Use UUIDs.
*actionDescription* | | string | Additional information on the action
blockingType | | string | Enum {'NONE', 'SOFT', 'HARD'}: <br> 'NONE': allows driving and other actions;<br>'SOFT': allows other actions but not driving;<br>'HARD': is the only allowed action at that time.
***actionParameters [actionParameter]*** <br><br> } | | array | Array of actionParameter objects for the indicated action, e.g., "deviceId", "loadId", "external triggers". <br><br> An example implementation can be found in [7.2 Format of parameters](#72-format-of-parameters).

Object structure | Unit | Data type | Description
---|---|---|---
**edge** { | | JSON object | Directional connection between two nodes.
edgeId | | string | Unique edge identification.
sequenceId | | uint32 | Number to track the sequence of nodes and edges in an order and to simplify order updates. <br>The variable sequenceId runs across all nodes and edges of the same order and is reset when a new orderId is issued.
*edgeDescription* | | string | Additional information on the edge.
released | | boolean | "true" indicates that the edge is part of the base.<br>"false" indicates that the edge is part of the horizon. 
startNodeId | | string | nodeId of first node within the order.
endNodeId | | string | nodeId of the last node within the order.
*maxSpeed* | m/s | float64 | Permitted maximum speed on the edge. <br>Speed is defined by the fastest measurement of the vehicle.
*maxHeight* | m | float64 | Permitted maximum height of the vehicle, including the load, on the edge.
*minHeight* | m | float64 | Permitted minimal height of the load handling device on the edge.
*orientation* | rad | float64 | Orientation of the AGV on the edge. The value `orientationType` defines if it has to be interpreted relative to the global project-specific map coordinate system or tangential to the edge. In case of interpreted tangential to the edge, 0.0 denotes driving forwards and PI denotes driving backwards. <br>Example: orientation Pi/2 rad will lead to a rotation of 90 degrees.<br><br>If the AGV starts in a different orientation, rotate the vehicle on the edge to the desired orientation, if `rotationAllowed` is set to "true".<br>If `rotationAllowed` is "false", rotate before entering the edge.<br>If that is not possible, reject the order.<br><br>If no trajectory is defined, apply the rotation to the direct path between the two connecting nodes of the edge.<br>If a trajectory is defined for the edge, apply the orientation to the trajectory.
*orientationType* | | string | Enum {'GLOBAL', 'TANGENTIAL'}: <br>'GLOBAL': relative to the global project-specific map coordinate system;<br>'TANGENTIAL': tangential to the edge.<br><br>If not defined, the default value is 'TANGENTIAL'.
*direction* | | string | Sets direction at junctions for line-guided or wire-guided vehicles, to be defined initially (vehicle-individual).<br> Examples: "left", "right", "straight".
*rotationAllowed* | | boolean | "true": rotation is allowed on the edge.<br>"false": rotation is not allowed on the edge.<br><br>Optional:<br>No limit, if not set.
*maxRotationSpeed* | rad/s | float64| Maximum rotation speed<br><br>Optional:<br>No limit, if not set.
***trajectory*** | | JSON object | Trajectory JSON object for this edge as NURBS. <br>Defines the path, on which the AGV should move between the start node and the end node of the edge.<br><br>Optional:<br>Can be omitted, if the AGV cannot process trajectories or if the AGV plans its own trajectory.
*length* | m | float64 | Length of the path from the start node to the end node<br><br>Optional:<br>This value is used by line-guided AGVs to decrease their speed before reaching a stop position.
***corridor*** | | JSON object | Definition of boundaries in which a vehicle can deviate from its trajectory, e.g., to avoid obstacles.<br>
**action [action]**<br><br><br> } | | array | Array of actions to be executed on the edge. <br>Empty array, if no actions required. <br>An action triggered by an edge will only be active for the time that the AGV is traversing the edge which triggered the action. <br>When the AGV leaves the edge, the action will stop and the state before entering the edge will be restored.


Object structure | Unit | Data type | Description
---|---|---|---
**trajectory** { | | JSON object |
degree | | float64 | Range: [1.0 ... float64.max]<br><br>Degree of the NURBS curve defining the trajectory.<br><br>If not defined, the default value is 1.
**knotVector [float64]** | | array | Range: [0.0 ... 1.0]<br><br>Array of knot values of the NURBS.<br><br>knotVector has size of number of control points + degree + 1.
**controlPoints [controlPoint]**<br><br> } | | array | Array of controlPoint objects defining the control points of the NURBS, explicitly including the start and end point.

Object structure | Unit | Data type | Description
---|---|---|---
**controlPoint** { | | JSON object |
x | | float64 | X-coordinate described in the world coordinate system.
y | | float64 | Y-coordinate described in the world coordinate system.
*weight* | | float64 | Range: [0.0 ... float64.max]<br><br>The weight of the control point on the curve.<br>When not defined, the default will be 1.0.
} | | |

Object structure | Unit | Data type | Description
---|---|---|---
***corridor*** { | | JSON object |
leftWidth | m | float64 | Range: [0.0 ... float64.max]<br>Defines the width of the corridor in meters to the left related to the trajectory of the vehicle (see Figure 13).
rightWidth | m | float64 | Range: [0.0 ... float64.max]<br>Defines the width of the corridor in meters to the right related to the trajectory of the vehicle (see Figure 13).
*corridorRefPoint* <br><br>**}**| | string | Defines whether the boundaries are valid for the kinematic center or the contour of the vehicle. If not specified the boundaries are valid to the vehicles kinematic center.<br> Enum { 'KINEMATICCENTER' , 'CONTOUR' }

### 6.7 Maps

To ensure consistent navigation among different types of AGVs, the position is always specified in reference to the project-specific coordinate system (see Figure 11).
For the differentiation between different levels of a site or location, a unique `mapId` is used.
The map coordinate system is to be specified as a right-handed coordinate system with the z-axis pointing skywards.
A positive rotation therefore is to be understood as a counterclockwise rotation.
The vehicle coordinate system is also specified as a right-handed coordinate system with the x-axis pointing in the forward direction of the vehicle and the z-axis pointing upward. The vehicle reference point is defined as (0,0,0) in the vehicle reference frame, unless specified otherwise.
This is in accordance with Section 2.11 in DIN ISO 8855.

![Figure 11 Coordinate system with sample AGV and orientation](./assets/coordinate_system_vehicle_orientation.png)
>Figure 11 Coordinate system with sample AGV and orientation

The X, Y, and Z coordinates shall be given in meters. 
The orientation shall be in radians and shall be within +Pi and –Pi.

![Figure 12 Coordinate systems for map and vehicle](./assets/coordinate_system_vehicle_map.png)
>Figure 12 Coordinate systems for map and vehicle


### 6.7.1 Map distribution

To enable an automatic map distribution and intelligent management of restarting the vehicles if necessary, a standardized way to distribute maps is introduced.

The map files to be distributed are stored on a dedicated map server that is accessible by the vehicles. To ensure efficient transmission, each transmission should consist of a single file. If multiple maps or files are required, they should be bundled or packed into a single file. The process of transferring a map from the map server to a vehicle is a pull operation, initiated by the master control triggering a download command using an `instantAction`.

Each map is uniquely identified by a combination of a map identifier (field `mapId`) and a map version (field `mapVersion`). The map identifier describes a specific area of the vehicle's physical workspace, and the map version indicates updates to previous versions. Before accepting a new order, the vehicle shall check that there is a map on the vehicle for each map identifier in the requested order. It is the responsibility of the master control to ensure that the correct maps are activated to operate the vehicle.

In order to minimize downtime and make it easier for the master control to synchronize the activation of new maps, it is essential that maps are pre-loaded or buffered on the vehicles. The status of the maps on the vehicle can be accessed via the vehicle state channel. It's important to note that transferring a map to an AGV and then activating the map are different processes. To activate a pre-loaded map on a vehicle, the master control sends an instant action. In this case, any other map with the same map identifier but a different map version is automatically disabled. Maps can be deleted by the master control with another instant action. The result of this process is shown in the vehicle state.

The map distribution process is shown in Figure 13.

![Figure 13 Map distribution process](./assets/map_distribution_process.png)
>Figure 13 Communication required between master control, AGV and map server to download, enable, and delete a map.

#### 6.7.2 Maps in the vehicle state

The `mapId` field in the `agvPosition` of the state represents the currently active map. Information about the maps available on a vehicle is presented in the `maps` array, which is a component of the state message. Each entry in this array is a JSON object consisting of the mandatory fields `mapId`, `mapVersion`, and `mapStatus`, which can be either 'ENABLED' or 'DISABLED'. An 'ENABLED' map can be used by the vehicle if necessary. A 'DISABLED' map shall not be used. The status of the download process is indicated by the current action not being completed. Errors are also reported in the state.

Note that multiple maps with different `mapId` can be enabled at the same time. There can only be one version of maps with the same `mapId` enabled at a time. If the `maps` array is empty, this means that there are currently no maps available on the vehicle.


#### 6.7.3 Map download

The map download is triggered by the `downloadMap` instant action from the master control. This command contains the mandatory parameters `mapId` and `mapDownloadLink` under which the map is stored on the map server and which can be accessed by the vehicle.

The AGV sets the `actionStatus` to 'RUNNING' as soon as it starts downloading the map file. If the download is successful, the `actionStatus` is updated to 'FINISHED'. If the download is unsuccessful, the status is set to 'FAILED'. Once the download has been successfully completed, the map shall be added to the array of `maps` in the state. Maps shall not be reported in the state, before they are ready to be enabled.

It is important to ensure that the process of downloading a map does not modify, delete, enable, or disable any existing maps on the vehicle.
The vehicle shall reject the download of a map with a `mapId` and `mapVersion` that is already on the vehicle. An error shall be reported, and the status of the instant action shall be set to 'FAILED'. The master control shall first delete the map on the vehicle and then restart the download.


#### 6.7.4 Enable downloaded maps

There are two ways to enable a map on a vehicle:

1. **Master control enables map**: Use the `enableMap` instant action to set a map to 'ENABLED' on the vehicle. Other Versions of the same `mapId` with different `mapVersion` are set to 'DISABLED'.
2. **Manually enable a map on the vehicle**: In some cases, it might be necessary to enable the maps on the vehicle directly. The result shall be reported in the vehicle state.

It is the responsibility of the master control to ensure that the correct maps are activated on the vehicle when sending the corresponding `mapId` as part of a `nodePosition` in an order.
If the vehicle is to be set to a specific position on a new map, the `initPosition` instant action is used.


#### 6.7.5 Delete maps on vehicle
The master control can request the deletion of a specific map from a vehicle. This is done with the instant action `deleteMap`. When a vehicle runs out of memory, it should report this to the master control, which can then initiate the deletion of maps. The vehicle itself is not allowed to delete maps.
After successfully deleting a map, it is important to remove that map's entry from the vehicle's array of maps in the vehicle state.


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

Object structure | Unit | Data type | Description
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

Object structure | Unit | Data type | Description
---|---|---|---
**map**{ | | JSON object|
mapId | | string | ID of the map describing a defined area of the vehicle's workspace.
mapVersion | | string | Version of the map.
*mapDescription* | | string | Additional information on the map.
mapStatus <br>}| | string | Enum {'ENABLED', 'DISABLED'}<br>'ENABLED': Indicates this map is currently active / used on the AGV. At most one map with the same mapId can have its status set to 'ENABLED'.<br>'DISABLED': Indicates this map version is currently not enabled on the AGV and thus could be enabled or deleted by request.

Object structure | Unit | Data type | Description
---|---|---|---
**nodeState** { | JSON object | |
nodeId | | string | Unique node identification.
sequenceId | | uint32 | sequence ID to discern multiple nodes with same nodeId.
*nodeDescription* | | string | Additional information on the node.
released| | boolean | "true" indicates that the node is part of the base.<br>"false" indicates that the node is part of the horizon.
***nodePosition***<br><br>}| | JSON object | Node position. <br>The object is defined in Section [6.6 Topic "order"](#66-topic-order-from-master-control-to-agv) <br>Optional: <br>Master control has this information. <br>Can be sent additionally, e.g., for debugging purposes.

Object structure | Unit | Data type | Description
---|---|---|---
**edgeState** { | | JSON object | |
edgeId | | string | Unique edge identification.
sequenceId | | uint32 | sequence ID to differentiate between multiple edges with the same edgeId.
*edgeDescription* | | string | Additional information on the edge.
released | | boolean | "true" indicates that the edge is part of the base.<br>"false" indicates that the edge is part of the horizon.
***trajectory*** <br><br>} | | JSON object | The trajectory is to be communicated as NURBS and is defined in Section [6.6.6 Implementation of the order message](#666-implementation-of-the-order-message)<br><br>Trajectory segments start from the point, where the vehicle enters the edge, and terminate at the point, where the vehicle reports that the end node was traversed.

Object structure | Unit | Data type | Description
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

Object structure | Unit | Data type | Description
---|---|---|---
**velocity** { | | JSON object |
*vx* | m/s | float64 | The AGV's velocity in its X-direction.
*vy* | m/s | float64 | The AGV's velocity in its Y-direction.
*omega*<br>}| Rad/s | float64 | The AGV's turning speed around its Z-axis.

Object structure | Unit | Data type | Description
---|---|---|---
**load** { | | JSON object |
*loadId* | | string | Unique identification of the load (e.g., barcode or RFID).<br><br>Empty field, if the AGV can identify the load but didn't identify the load yet.<br><br>Optional if the AGV cannot identify the load.
*loadType* | | string | Type of load.
*loadPosition* | | string | Indicates, which load handling/carrying unit of the AGV is used, e.g., in case the AGV has multiple spots/positions to carry loads.<br><br>For example: "front", "back", "positionC1", etc.<br><br>Optional for vehicles with only one loadPosition
***boundingBoxReference*** | | JSON object | Point of reference for the location of the bounding box. <br>The point of reference is always the center of the bounding box's bottom surface (at height = 0) and is described in coordinates of the AGV's coordinate system.
***loadDimensions*** | | JSON object | Dimensions of the load's bounding box in meters.
*weight*<br>} | kg | float64 | Range: [0.0 ... float64.max]<br><br>Absolute weight of the load measured in kg.

Object structure | Unit | Data type | Description
---|---|---|---
**boundingBoxReference** { | | JSON object | Point of reference for the location of the bounding box. <br>The point of reference is always the center of the bounding box's bottom surface (at height = 0) and is described in coordinates of the AGV's coordinate system.
x | | float64 | X-coordinate of the point of reference.
y | | float64 | Y-coordinate of the point of reference.
z | | float 64 | Z-coordinate of the point of reference.
*theta*<br> } | | float64 | Orientation of the loads bounding box. <br>Important for tuggers, trains, etc.

Object structure | Unit | Data type | Description
---|---|---|---
**loadDimensions** { | | JSON object | Dimensions of the load's bounding box in meters.
length | m | float64 | Absolute length of the load's bounding box.
width | m | float64 | Absolute width of the load's bounding box.
*height* <br>}| m | float64 | Absolute height of the load's bounding box.<br><br>Optional:<br><br>Set value only if known.

Object structure | Unit | Data type | Description
---|---|---|---
**actionState** { | | JSON object |
actionId | |string | Unique identifier of the action.
*actionType* | | string | Type of the action.<br><br>Optional: Only for informational or visualization purposes. Master control is aware of action type as dispatched in the order.
*actionDescription* | | string | Additional information on the current action.
actionStatus | | string | Enum {'WAITING', 'INITIALIZING', 'RUNNING', 'PAUSED', 'FINISHED', 'FAILED'}<br><br>See Section [6.11 actionStates](#611-actionstates).
*resultDescription*<br>} | | string | Description of the result, e.g., the result of an RFID reading.<br><br>Errors will be transmitted in errors.

Object structure | Unit | Data type | Description
---|---|---|---
**batteryState** { | | JSON object | 
batteryCharge | % | float64 | State of Charge: <br> if AGV only provides values for good or bad battery levels, these will be indicated as 20% (bad) and 80% (good). 
*batteryVoltage* | V | float64 | Battery voltage.
*batteryHealth* | % | int8 | Range: [0 ... 100]<br><br>State describing the battery's health. 
charging | | boolean | “true”: charging in progress.<br>“false”: the AGV is currently not charging.
*reach* <br>}| m | uint32 | Range: [0 ... uint32.max]<br><br>Estimated reach with current state of charge. 

Object structure | Unit | Data type | Description
---|---|---|---
**error** { | | JSON object |
errorType | | string | Type/name of error
***errorReferences [errorReference]*** | | array | Array of references (e.g., nodeId, edgeId, orderId, actionId, etc.) to provide more information related to the error.<br>For additional information see [7 Best practice](#7-best-practice).
*errorDescription* | | string | Verbose description providing details and possible causes of the error.
*errorHint* | | string | Hint on how to approach or solve the reported error.
errorLevel <br> }| | string | Enum {'WARNING', 'FATAL'}<br><br>'WARNING': the AGV is ready to start (e.g., maintenance cycle expiration warning).<br>'FATAL': the AGV is not in running condition, user intervention required (e.g., laser scanner is contaminated).

Object structure | Unit | Data type | Description
---|---|---|---
**errorReference** { | | JSON object |
referenceKey | | string | Specifies the type of reference used (e.g., nodeId, edgeId, orderId, actionId, etc.).
referenceValue <br>} | | string | The value that belongs to the reference key. For example, the ID of the node where the error occurred.

Object structure | Unit | Data type | Description
---|---|---|---
**info** { | | JSON object |
infoType | | string | Type/name of information.
*infoReferences [infoReference]* | | array | Array of references.
*infoDescription* | | string | Description of the information.
infoLevel <br>}| | string | Enum {'DEBUG', 'INFO'}<br><br>'DEBUG': used for debugging.<br> 'INFO': used for visualization.

Object structure | Unit | Data type | Description
---|---|---|---
**infoReference** { | | JSON object |
referenceKey | | string | References the type of reference (e.g., headerId, orderId, actionId, etc.).
referenceValue <br>} | | string | References the value, which belongs to the reference key.

Object structure | Unit | Data type | Description
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

| **Field** | **data type** | **description** |
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

| **Field** | **data type** | **description** |
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

| **Field** | **data type** | **description** |
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

| **Field** | **data type** | **description** |
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

| **Field** | **data type** | **description** |
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

| **Field** | **data type** | **description** |
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

| **Field** | **data type** | **description** |
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

| **Field** | **data type** | **description** |
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


# 7 Best practice

This section includes additional information, which helps in facilitating a common understanding concurrent with the logic of the protocol.

## 7.1 Error reference

If an error occurs due to an erroneous order, the AGV should return a meaningful error reference in the field errorReferences (see Section [6.10.6 Implementation of the state message](#6106-implementation-of-the-state-message) of the state topic).
This can include the following information:

- `headerId`
- Topic (`order` or `instantAction`)
- `orderId` and `orderUpdateId` if error was caused by an order update
- `actionId` if error was caused by an action
- List of parameters if error was caused by erroneous action parameters

If an action cannot be completed because of external factors (e.g., no load at expected position), the actionId should be referenced.


## 7.2 Format of parameters

Parameters for errors, information and actions are designed as an array of JSON objects with key-value pairs.

| **Field** | **data type** | **description** |
|---|---|---|
**actionParameter** { | JSON object | actionParameter for the indicated action, e.g., deviceId, loadId, external triggers.
key | string | The key of the parameter.
value</br>} | One of:</br>array,</br>boolean,</br>number,</br>string,</br>object | The value of the parameter that belongs to the key.

Examples for the `actionParameter` of an action "someAction" with key-value pairs for stationType and loadType:

"actionParameters":[
{"key":"stationType", "value": "floor"},
{"key":"weight", "value": 8.5},
{"key": "loadType", "value": "pallet_eu"}
]

The reason for using the proposed scheme of "key": "actualKey", "value": "actualValue" is to keep the implementation generic. The "actualValue" can be of any possible JSON data type, such as float, bool, and even an object.


# 8 用語集


## 8.1 定義

コンセプト|説明
---|---
フリーナビゲーションAGV|地図を使用して自らの経路を計画する車両。<br>マスターコントロールは、出発地と目的地の座標のみを送信する。<br>車両は、その経路をマスターコントロールに送信する。<br>マスターコントロールとの接続が切れた場合でも、車両は走行を継続できる。<br>フリーナビゲーション車両は、局所的な障害物を回避することが許可される場合がある。<br>また、受け取り/払い出し位置の微調整を車両自身が行うことも可能である。
誘導車両（物理的または仮想的）|マスターコントロールから経路情報を受け取る車両。<br>経路の計算はマスターコントロールで行われる。<br>マスターコントロールとの通信が途切れた場合、車両はリリースされたノードとエッジ（「ベース」）を終了し、停止する。<br>誘導された車両は、局所的な障害物を回避することが許可される場合がある。<br>また、受け取り/払い出し位置の微調整が車両自身によって行われる場合もある。
中央マップ|マスターコントロールで一元管理されるマップ。<br>これは最初に作成され、その後使用される。

