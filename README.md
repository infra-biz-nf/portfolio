# Infrastructure & Operations Portfolio

## 👤 Profile
インフラ／IT運用改善エンジニア。  
SIerとして As-Is 業務移管のリード、要件定義、詳細設計、ベンダー調整、運用定着まで一連の工程を担当。  
近年は「効率化」「ローカル作業の排除」を軸に、業務フロー標準化・自動化・生成AI活用に注力。

---

## 🛠 Core Strengths
- インフラ構築／運用（オンプレ・クラウド両方）
- 業務フロー改善（標準化、省力化、見える化）
- 移行設計・移行方式検討（OS/MW バージョンアップ含む）
- ベンダーコントロール／ステークホルダー調整
- ドキュメント整備（手順書、運用設計、パラメータシート）
- AI活用（ナレッジ検索・ドキュメント自動生成）

---

## 💼 Experience Summary
### ■ 業務移管・運用改善プロジェクト  
- 他社からの As-Is 業務移管のリード  
- ギャップ分析、交通整理、要件ヒアリング  
- 運用フロー再構築、ローカル作業排除、標準化推進  
- 定着化までの運用ガイド作成および教育

### ■ インフラ構築・運用  
- サーバ構築、パラメータ設計  
- 既存機器のリプレースおよび設定変更  
- 監視設計 / インシデント対応  
- OS・MW のバージョンアップ／移行設計

### ■ プロジェクトリード  
- 進捗管理 / リスク管理 / 品質管理  
- 調整業務（ユーザ部門、ベンダー、社内関係者）  
- プロジェクト計画策定・推進  
- チームマネジメント（新人教育・要員管理）

---

# AWS Internal FAQ Chatbot（Lambda + API Gateway）

AWS Lambda（Python）と API Gateway を使った、
社内問い合わせ・ヘルプデスク向けの簡易チャットボットのサンプルです。

パスワード / VPN / PC不具合などのよくある質問に対して、
キーワードマッチで自動応答を返します。

## 構成技術

- AWS Lambda（Python）
- Amazon API Gateway
- Amazon CloudWatch Logs

## 想定ユースケース

- 社内ポータルや Teams Bot からの問い合わせの一次回答
- ヘルプデスクの負荷軽減
- よくある FAQ の自動化

## リクエスト例

```json
{
  "userId": "taro",
  "message": "VPNが接続できません"
}
{
  "userId": "taro",
  "reply": "VPN接続ができない場合は、まずネットワーク環境を確認し、社内ヘルプページのVPNトラブルシューティング手順に沿ってご確認ください。",
  "timestamp": "2025-05-01T10:24:33"
}

### `lambda_function.py`

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
        "answer": "VPN接続ができない場合は、まずネットワーク環境を確認し、社内ヘルプページのVPNトラブルシューティング手順に沿ってご確認ください。"
    },
    {
        "keywords": ["オフィス", "出社", "勤務時間"],
        "answer": "オフィスの利用時間や出社ルールは、社内規程の「勤務・出社ルール」をご確認ください。"
    },
    {
        "keywords": ["PC", "パソコン", "故障"],
        "answer": "PC の不具合は電源・ケーブル接続を確認後、ヘルプデスクの問い合わせフォームからチケットを起票してください。"
    }
]

DEFAULT_ANSWER = (
    "すみません、回答が見つかりませんでした。\n"
    "キーワードを変えてもう一度お試しいただくか、ヘルプデスクまでお問い合わせください。"
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
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "internal server error"})
        }

# Azure Internal FAQ Chatbot（Azure Functions / Python）

Azure Functions（HTTP Trigger）で実装した、
社内向け FAQ チャットボット API のサンプルです。

AWS 版と同じ FAQ ロジックで、Teams や社内 Web から利用できます。

## 構成技術

- Azure Functions（Python）
- HTTP Trigger
- Azure Monitor（ログ）

## リクエスト例

```json
{"userId": "taro", "message": "パスワードを忘れました"}
}

{
  "userId": "taro",
  "reply": "パスワードをお忘れの場合は、社内ポータルの「パスワードリセット」から再発行してください。",
  "timestamp": "2025-05-01T09:12:33"
}


### `__init__.py`

```python
import logging
import json
from datetime import datetime, timezone, timedelta
import azure.functions as func

FAQ_DATA = [
    {
        "keywords": ["パスワード", "ログイン"],
        "answer": "パスワードをお忘れの場合は、社内ポータルの「パスワードリセット」から再発行してください。"
    },
    {
        "keywords": ["VPN", "接続できない"],
        "answer": "VPN接続ができない場合は、まずネットワーク環境をご確認ください。"
    },
    {
        "keywords": ["オフィス", "出社"],
        "answer": "出社ルールは、社内規程の「勤務・出社ルール」をご確認ください。"
    },
    {
        "keywords": ["PC", "故障"],
        "answer": "PC不具合はヘルプデスクからチケットを起票してください。"
    }
]

DEFAULT_ANSWER = "回答を見つけられませんでした。ヘルプデスクへお問い合わせください。"

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


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info("Chatbot request received.")

    try:
        body = req.get_json()
        user = body.get("userId", "unknown")
        message = body.get("message", "")

        answer = find_answer(message)
        now = datetime.now(JST).isoformat(timespec="seconds")

        return func.HttpResponse(
            json.dumps(
                {"userId": user, "reply": answer, "timestamp": now},
                ensure_ascii=False
            ),
            mimetype="application/json",
            status_code=200
        )

    except Exception as e:
        logging.error(e)
        return func.HttpResponse(
            json.dumps({"error": "internal server error"}, ensure_ascii=False),
            status_code=500
        )
# AWS Asset Management Automation（資産管理台帳フロー）

PC・周辺機器の「貸出・返却・廃棄」などの更新を、
DynamoDB に自動反映し、同時に S3 にバックアップ保存するサンプルです。

## 構成技術

- Amazon API Gateway
- AWS Lambda（Python）
- Amazon DynamoDB
- Amazon S3（バックアップ）

## 主な項目

- assetId（例：PC-001）
- assetType（Laptop / Monitor など）
- serialNumber
- status（in-use / returned / disposed）
- holder（利用者）
- location（設置場所）
- history（更新履歴）

import json
from datetime import datetime, timezone, timedelta
import boto3

dynamodb = boto3.resource("dynamodb")
s3 = boto3.client("s3")

TABLE_NAME = "AssetManagementTable"
BACKUP_BUCKET = "asset-management-backups"
JST = timezone(timedelta(hours=9))


def build_response(body, status_code=200):
    return {
        "statusCode": status_code,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(body, ensure_ascii=False)
    }


def update_asset(record):
    now = datetime.now(JST).isoformat(timespec="seconds")
    table = dynamodb.Table(TABLE_NAME)

    existing = table.get_item(Key={"assetId": record["assetId"]}).get("Item")
    new_history = {
        "timestamp": now,
        "status": record.get("status"),
        "holder": record.get("holder"),
        "location": record.get("location"),
        "note": record.get("note"),
        "updateUser": record.get("updateUser")
    }

    history = existing.get("history", []) if existing else []
    history.append(new_history)

    updated_item = {
        "assetId": record["assetId"],
        "assetType": record.get("assetType"),
        "serialNumber": record.get("serialNumber"),
        "model": record.get("model"),
        "status": record.get("status"),
        "holder": record.get("holder"),
        "location": record.get("location"),
        "note": record.get("note"),
        "updatedAt": now,
        "history": history
    }

    table.put_item(Item=updated_item)
    return updated_item


def backup_to_s3(asset):
    key = f"backups/{asset['assetId']}.json"
    body = json.dumps(asset, ensure_ascii=False, indent=2)
    s3.put_object(
        Bucket=BACKUP_BUCKET,
        Key=key,
        Body=body.encode("utf-8"),
        ContentType="application/json"
    )
    return key


def lambda_handler(event, context):
    try:
        body = json.loads(event.get("body", "{}"))

        updated = update_asset(body)
        key = backup_to_s3(updated)

        return build_response({
            "assetId": updated["assetId"],
            "status": updated["status"],
            "backupS3Key": key,
            "message": "asset updated & backup stored"
        })

    except Exception as e:
        print("Error:", e)
        return build_response({"error": "internal server error"}, 500)

# PC Repair Invoice Automation（PC修理請求書フロー）

PC修理の完了データを受け取り、
請求書データを自動生成 → S3 に保存 → SES でメール送信するサンプルです。

## 構成技術

- AWS Lambda（Python）
- Amazon API Gateway
- Amazon S3（請求データ保存）
- Amazon SES（請求メール送信）

## 入力例

- ticketId
- customerName / customerEmail
- deviceName
- repairContent
- repairFee / partsFee / taxRate（消費税率）
import json
import uuid
from datetime import datetime, timezone, timedelta
import boto3

s3 = boto3.client("s3")
ses = boto3.client("ses")

S3_BUCKET_NAME = "pc-repair-invoices-bucket"
SENDER_EMAIL = "no-reply@example.com"
JST = timezone(timedelta(hours=9))


def build_invoice(data):
    now = datetime.now(JST)

    invoice_id = f"INV-{now.strftime('%Y%m%d')}-{uuid.uuid4().hex[:8]}"
    repair_fee = int(data.get("repairFee", 0))
    parts_fee = int(data.get("partsFee", 0))
    tax_rate = float(data.get("taxRate", 0.1))

    subtotal = repair_fee + parts_fee
    tax = int(subtotal * tax_rate)
    total = subtotal + tax

    return {
        "invoiceId": invoice_id,
        "ticketId": data.get("ticketId"),
        "issueDate": now.strftime("%Y-%m-%d"),
        "customerName": data.get("customerName"),
        "customerEmail": data.get("customerEmail"),
        "deviceName": data.get("deviceName"),
        "repairContent": data.get("repairContent"),
        "repairFee": repair_fee,
        "partsFee": parts_fee,
        "subtotal": subtotal,
        "tax": tax,
        "total": total
    }


def save_to_s3(invoice):
    key = f"invoices/{invoice['invoiceId']}.json"
    s3.put_object(
        Bucket=S3_BUCKET_NAME,
        Key=key,
        Body=json.dumps(invoice, ensure_ascii=False, indent=2).encode(),
        ContentType="application/json"
    )
    return key


def send_email(invoice):
    if not invoice.get("customerEmail"):
        print("No customerEmail. Skip SES send.")
        return

    subject = f"[請求書] PC修理のご請求 ({invoice['invoiceId']})"
    body = f"""
{invoice['customerName']} 様

PC修理のご利用ありがとうございます。
請求書を発行しました。

合計金額: {invoice['total']:,} 円

詳細は請求書データをご確認ください。
"""

    ses.send_email(
        Source=SENDER_EMAIL,
        Destination={"ToAddresses": [invoice["customerEmail"]]},
        Message={
            "Subject": {"Data": subject, "Charset": "UTF-8"},
            "Body": {"Text": {"Data": body, "Charset": "UTF-8"}
            }
        },
    )


def lambda_handler(event, context):
    try:
        body = json.loads(event.get("body", "{}"))
        invoice = build_invoice(body)
        key = save_to_s3(invoice)
        send_email(invoice)

        return {
            "statusCode": 200,
            "body": json.dumps(
                {
                    "invoiceId": invoice["invoiceId"],
                    "total": invoice["total"],
                    "s3Key": key,
                    "message": "invoice generated & email sent"
                },
                ensure_ascii=False
            )
        }

    except Exception as e:
        print("Error:", e)
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "internal server error"})
        }

