# Shiraishi-Lab専用在庫管理システム提案書



## 課題
Shiraishi-Labの物品は，他の研究室と比べ多いと言われている．また，会社からの貸出品があるためデータ上で管理する必要性がある．完全に物品を把握している人が2名であるため，blackbox化している．
また，他の在庫管理サービスに依頼する程の量ではないため委託する必要性が低い．（月1000円以上であるサービスが多い）

　
## 全体構成
本システムは，以下の3つの主要なコンポーネントで構成する．
- フロントエンド（Reactアプリケーション）
		ユーザーインターフェースを提供し，在庫情報の表示や操作を行う．
		AWS S3にホスティングされ，静的コンテンツとして提供する．
- バックエンド（AWS Lambda + API Gateway）
  - 在庫データの追加，更新，削除，貸出，返却を行うREST APIを提供する．
  - 利用頻度に応じて課金されるサーバーレス構成を採用する．
- データベース（Amazon DynamoDB）
  在庫データを保存・管理するNoSQLデータベースを使用する．
  サーバーレスで自動スケーリングに対応しており，利用頻度が低い場合のコストを抑えられる．


## システム構成図
```scss
 [ユーザー
    ↓（HTTPS）
 [CloudFront (CDNで高速配信 + HTTPS対応)
    ↓（静的ファイル）
 [AWS S3 (Reactアプリケーションのホスティング)
    ↓（API呼び出し）
 [API Gateway (HTTPリクエストを受け付け)
    ↓
 [AWS Lambda (API処理)
    ↓
 [DynamoDB (在庫データを管理)
```



## 各コーポメントの詳細


### フロントエンド


 技術スタック

 
  フレームワーク：React
  スタイリング：Tailwind CSS
  ホスティング：AWS S3

 主な機能

 
  - 在庫リストの表示
  - 在庫の追加，更新，削除，貸出，返却
  - 備品の貸出・返却フォームを追加
  - 貸出状況の表示
  - 入力バリデーション(例：在庫数が負の値にならない)

### バックエンド


技術スタック

 
  - AWS Lambda: リクエストごとにAPIを実行。
  - API Gateway: REST APIを公開し，Lambdaと接続．

| メソッド | エンドポイント | 説明 |
|--------|--------------|------|
 GET|	/items	|全ての備品リストを取得
 GET|	/items/{id}	|特定の備品の詳細情報を取得
 POST|	/items	|新しい備品を追加
 PUT|	/items/{id}	|備品情報を更新（例: 場所や名称）
 DELETE|	/items/{id}	|備品を削除
 POST|	/items/{id}/borrow	|備品を貸出（借りる人の情報を登録）
 POST|	/items/{id}/return	|備品を返却（借り手情報を削除または更新）
 GET|	/items/{id}/borrowed	|貸出中の備品情報を取得


### データベース


 技術スタック

 
  - Amazon DynamoDB: 高速でスケーラブルなNoSQLデータベース．

 データモデル例
  ```json
   {
     "id": "123",                  
     "name": "プロジェクター",     
     "quantity": 10,               
     "location": "615",        
     "borrowed": [                
       {
         "borrower": "山田太郎",     
         "borrowedAt": "2024-11-21T10:00:00Z", 
         "dueDate": "2024-11-30T18:00:00Z"    
       },
       {
         "borrower": "佐藤花子",     
         "borrowedAt": "2024-11-20T14:00:00Z",
         "dueDate": "2024-11-25T17:00:00Z"
       }
	]
     ,
     "updatedAt": "2024-11-21T10:00:00Z"
   }

  ``` 



## コスト見積もり


- AWSのサービス利用におけるコスト見積もり（小規模運用を想定）

1.	AWS S3（静的サイトホスティング）
		ストレージ: 月1GB程度 → 約$0.023
		データ転送: 月10GB程度 → 約$0.09

2.	AWS Lambda（バックエンド）
		月間リクエスト: 1,000回 → 無料枠内で無料

3.	Amazon DynamoDB（データベース）
		月間リクエスト: 10,000回 → 無料枠内で無料

4.	CloudFront（HTTPS対応）
		月間データ転送: 10GB → 無料枠内で無料

### 合計: 月$1未満## （ほぼ無料運用）



## 今後の展開案

- PWA対応
		システムをProgressive Web Appに拡張し，オフライン利用やプッシュ通知を提供

- 認証機能の追加
		AWS Cognitoを活用して，ユーザーごとの認証・アクセス管理を実現




## 開発フロー

- フェーズ1: フロントエンド開発
  Reactアプリケーションを設計・実装
  モックデータを使用してデザインと機能を構築

- フェーズ2: バックエンド開発
  Lambda関数を作成し，API Gatewayでエンドポイントを公開
  DynamoDBテーブルを設定し，Lambdaから接続

- フェーズ3: 全体統合
  フロントエンドとバックエンドを連携
  S3にデプロイし，CloudFrontを設定

- フェーズ4: テストと公開
  スマートフォンとPCで動作確認
  本番環境で公開

  
