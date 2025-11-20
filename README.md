# Infrastructure & Operations Portfolio

## 👤 Profile
インフラ／IT運用改善エンジニア。  
SIerとして As-Is 業務移管のリード、要件定義、詳細設計、ベンダー調整、運用定着まで一連の工程を担当。  
近年は「効率化」「ローカル作業の排除」を軸に、業務フロー標準化・自動化・生成AI活用に注力。

---

## 🛠 Core Strengths
- インフラ構築・運用（オンプレ／クラウド両方）
- 業務フロー改善（標準化、省力化、可視化）
- 移行設計・方式検討（OS/MW バージョンアップ含む）
- ベンダーコントロール・ステークホルダー調整
- ドキュメント整備（運用設計、手順書、パラメータシート）
- 生成AI活用（ナレッジ検索／自動ドキュメント生成）

---

## 💼 Experience Summary

### ■ 業務移管・運用改善プロジェクト
- 他社からの As-Is 業務移管リード  
- ギャップ分析・要件ヒアリング  
- 運用フロー再構築、ローカル作業排除、標準化推進  
- 運用ガイド作成、教育、定着化支援  

### ■ インフラ構築・運用
- サーバ構築、パラメータ設計  
- 既存機器リプレース、設定変更  
- 監視設計、インシデント対応  
- OS/MW バージョンアップ、移行設計  

### ■ プロジェクトリード
- 進捗・品質・リスク管理  
- ユーザ部門／ベンダー調整  
- プロジェクト計画策定  
- チームマネジメント、新人教育  

---

# 🔧 AWS Internal FAQ Chatbot  
**（AWS Lambda + API Gateway）**

社内問い合わせ（VPN、PW、PC不具合など）に対して  
キーワードマッチで自動回答する簡易 FAQ API。

## 技術構成
- AWS Lambda（Python）
- Amazon API Gateway
- Amazon CloudWatch Logs

## Request / Response 例

```json
{
  "userId": "taro",
  "message": "VPNが接続できません"
}
```

```json
{
  "userId": "taro",
  "reply": "VPN接続ができない場合は、まずネットワーク環境をご確認ください。",
  "timestamp": "2025-05-01T10:24:33"
}
```

---

## `lambda_function.py`

```python
import json
from datetime import datetime, timezone, timedelta

FAQ_DATA = [
    {
        "keywords": ["パスワード", "ログイン", "PW"],
        "answer": "パスワードをお忘れの場合は、社内ポータルの「パスワードリセット」から再発行してください。"
    },
    {
        "keywords": ["VPN", "接続できない", "リモート"],
        "answer": "VPN接続ができない場合は、まずネットワーク環境をご確認ください。"
    },
    {
        "keywords": ["オフィス", "出社", "勤務時間"],
        "answer": "オフィスの出社ルールは社内規程をご確認ください。"
    },
    {
        "keywords": ["PC", "パソコン", "故障"],
        "answer": "PC の不具合は電源・ケーブル接続を確認後、ヘルプデスクにチケットを起票してください。"
    }
]

DEFAULT_ANSWER = (
    "すみません、回答が見つかりませんでした。\n"
    "キーワードを変えてもう一度お試しください。"
)

JST = timezone(timedelta(hours=9))


def find_answer(message: str) -> str:
    text = message.lower()
    best_score = 0
    best_answer = None

    for faq in FAQ_DATA:
        score = sum(1 for kw in faq["keywords"] if kw.lower() in text)
        if score > best_score:
            best_score = score
            best_answer = faq["answer"]

    return best_answer or DEFAULT_ANSWER


def lambda_handler(event, context):
    try:
        body = json.loads(event.get("body", "{}"))
        message = body.get("message", "")
        user = body.get("userId", "unknown")

        answer = find_answer(message)
        now = datetime.now(JST).isoformat(timespec="seconds")

        return {
            "statusCode": 200,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps(
                {"userId": user, "reply": answer, "timestamp": now},
                ensure_ascii=False
            )
        }

    except Exception as e:
        print("Error:", e)
        return {"statusCode": 500, "body": json.dumps({"error": "internal server error"})}
```

---

# 🔧 Azure Internal FAQ Chatbot  
**（Azure Functions / Python）**

AWS版と同じロジックで Azure Functions (HTTP Trigger) を実装。

## 技術構成
- Azure Functions（Python）
- HTTP Trigger
- Azure Monitor（ログ）

## Request / Response 例

```json
{"userId": "taro", "message": "パスワードを忘れました"}
```

```json
{
  "userId": "taro",
  "reply": "パスワードをお忘れの場合は、社内ポータルの「パスワードリセット」から再発行してください。",
  "timestamp": "2025-05-01T09:12:33"
}
```

---

# 🔧 Asset Management Automation  
**（DynamoDB + S3）**

PC／モニタなどの資産棚卸しを API 経由で更新し、  
DynamoDB に反映 → S3 にバックアップする仕組み。

---

# 🔧 PC Repair Invoice Automation  
**（請求書自動生成：S3 + SES）**

PC修理の完了データを受け取り  
請求書を自動生成 → JSON を S3 保存 → SES で送信するフロー。

---
