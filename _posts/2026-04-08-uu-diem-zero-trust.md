---
title: "Hệ thống Backup 5 tầng"
date: 2026-04-08 16:51:00 +0700
categories: [Cybersecurity, Backup]
tags: [siem, lab]
---
## Backup dữ liệu là ?

Backup dữ liệu là quá trình sao lưu, lưu trữ một bản sao của dữ liệu quan trọng để có thể khôi phục (restore) lại khi dữ liệu gốc bị mất, hỏng hóc, bị tấn công hoặc xảy ra sự cố. Mục đích chính của backup là đảm bảo an toàn dữ liệu và tính liên tục trong hoạt động kinh doanh hay công việc cá nhân. Bên cạnh đó còn hỗ trợ giảm thiểu về vấn đề ransomware và có khả năng hồi phục dữ liệu

## #Hệ thống backup

1. Production  : Nhận dữ liệu, tạo dữ liệu và phân loại dữ liệu 
2. Snapshort : Snapshot dữ liệu lưu trữ vào và chuyển tới cho backup storage để lưu trữ
3. Orchestrator : Trung tâm xử lý cấp token và đồng thời cũng là thành phần trung gian giữa vault và backup storage, ra mọi quyết định ( Phần đầu não )
4. Backup Storage : phần lưu trữ tạm thời của hệ thống và chờ đợi nhận nhiệm vụ để backup xuống vault 
5. Vault : Phần cứng nhất của hệ thống, khi mất vault thì chúng ta có thể mất hết.

## #Cấu trúc file của hệ thống

```jsx
Production
hospital-app/
├── token.jwt // khoa cua ochestrator cap 
├── data/ // vung chua du lieu 
│   └── input.txt // data do nguoi dung nhap
├── output/ // folder sau khi phan loai du lieu xong
│   ├── public.txt
│   ├── confidential.txt
│   └── sensitive.txt
├── /opt/orchestrator/ //file chua log
└── scripts/
    ├── data.sh // phan loai du lieu
    ├── policy.conf // policy cho cac kieu du lieu
    ├── export.sh // scripts moi kem hassh
    └── verify_jwt1.py // file xac thuc token
    
Snapshort/
/data/staging/app/
├── incoming/ // file chua data nhan tu production
├── logs/
│   ├── backup.log // ghi log khi snapshort chuyen toi tang 4
│   └── snapshot.log // log snapshort
├── manifests/ 
│   ├── app_snapshot_1773290613.json
│   ├── app_snapshot_1773292016.json
│   ├── ... (các file snapshot khác)
│   └── app_snapshot_1773813219.json
├── scripts/
│   ├── verify.sh // file xac nhan hash
│   └── k.sh // scripts thuc hien snapshort va delete file on incoming
├── snapshots/
│       └── app_snapshot_1773813219.tar.gz // luu tru cung nhu file snapshort 
│     
└──/usr/local/bin/
				 └──backup_worker.sh // file chuyen du lieu toi backupstorage

Orchestrator/
/opt/orchestrator/
├── clients.json       # Danh sách thông tin khách hàng/node
├── jwt_secret.key     # Khóa bí mật (Cực kỳ quan trọng!)
├── a.json // policy + time dung de setup // file moi cap token luu hash      
└── token_server1.py
├── backup/
│   ├── backup_policy_engine.py // file cap token cho backupstorage
│   ├── db/
│   │   └── backup_sessions.db // tao session backup
│   ├── logs/
│   │   └── (trống) // ghi log cua backup 
│   └── policies/
│       └── backup_policy.json
├──snapshot/
│   ├── snapshot_policy.json
│   ├── snapshot_policy_server.py // file cap token cho snapshort
└── vault/
		 ├── server.py // file lay token tu vault va chuyen den cho backup_storage
		 
Backup_storage/
├── app/
│   ├── app_snapshot_1773714759.tar.gz  # Backup ngày 17/03
│   └── app_snapshot_1773813219.tar.gz  # Backup ngày 18/03 (Mới nhất)
├── logs/
│   └── (Trống - Nơi lưu log của retention_manager)
├── retention_manager.sh                 # Script tự động xóa bản backup cũ
├── proccess.sh // file xu ly du lieu phan loai du lieu quan trong
├── push.sh // file run de nhan token tu vault thong qua orchestrator

Vault/
/var/lib/vault/ (Root Storage)
├── audit/           # Nhật ký truy xuất (Ai đã xem secret nào?)
├── auth/            # Các phương thức xác thực (LDAP, AppRole, GitHub...)
├── core/            # Dữ liệu cốt lõi của Vault (Cluster, Seal status)
├── logical/         # Nơi lưu trữ Secret Engines (KV, AWS, Database...)
├── sys/             # Cấu hình hệ thống và các chính sách chung
└── backup-policy.hcl # File định nghĩa chính sách backup (HCL format)
/var/log/vault/ (Nhật ký truy vết)
├── audit.log             # Nhật ký hoạt động (1.7 KB)
└── vault_audit.log       # Nhật ký chi tiết (53.4 KB - Hoạt động chính)

```

## #Các lệnh cần biết khi chạy vault

```jsx
vault server -config=/etc/vault.d/vault.hcl //lenh chay cau hinh cua vault
export VAULT_ADDR='http://192.168.56.20:8200' // tao bien moi truong cho no
vault operator init // lenh nhan Unseal key va Root Token
vault operator unseal // lenh unseal vault

Cau hinh auth + policy
vault auth enable approle
vault policy write backup-policy policy.hcl // ban phai tao file policy.hcl roi viet vao do policy cua ban thi moi apply duoc
vault write auth/approle/role/backup-role \
    token_policies="backup-policy" \
    token_ttl=5m \
    token_max_ttl=10m
// lenh nay giup chung ta tao role va vault se cap token
vault read auth/approle/role/backup-role/role-id
vault write -f auth/approle/role/backup-role/secret-id // hai lenh  nay la lay credential dung de xac thuc thi
v// vault moi cap token duoc
```

## Mục tiêu bài lab

Mục đích mình thực hiện bài lab này vì hiện tại mình đang có một dự án nghiên cứu khoa học và tên đề tài của nó là “Xây dựng phương pháp ngăn chặn ransomware bảo vệ dữ liệu y tế số. Bên cạnh đó mình muốn hiểu được hệ thống nó sẽ hoạt động như thế nào và nguy cơ về độ bảo mật khi nó xảy ra giảm thiểu sự cố là bao nhiêu. Hệ thống có thể sẽ có lỗi sau khi triển khai nhưng nó vẫn sẽ mang lại một điều gì đó hết sức cần thiết cho data hospital. Bên cạch đó khi chúng ta backup hay store data thì nó luôn verify credential liên tục cho chúng ta hạn chế và giảm thiểu tối đa vầ ddie

## Gốc nhìn tổng quan

Hiện nay trên thế giới chúng ta đã và đang thấy có rất nhiều vụ ransomware và loss data xảy ra, gây nên thiệt hại rất lớn về chi phí cũng như con người. Bên cạnh đó chúng ta có thể thấy được đó không phải là do vấn đề vì con người mà vấn đề nằm ở bảo mật và vấn đề đó sẽ phải giải quyết ra sao nếu chúng ta không có một cái gì đó để khiến điều này xảy ra trong scope mà chúng ta có thể kiểm soát được và mất dữ liệu có thể chấp nhận được.

## Những điều cần chuẩn bị

Vì có hệ thống có 5 tầng nên bạn cần chuẩn 5-6 máy ảo nếu là máy bạn đủ mạnh và đủ khỏe để làm những việc này, cón nếu không thì bạn có thể kết hợp tầng 1 + tầng 2 vào một máy ảo, và tầng 4 + tầng 5 vào một máy ảo còn tầng 3 giữ nguyên, recommend bạn nên setup wazuh cho mỗi máy ảo riêng, hiện tại mình đang cấu hình ip tĩnh cho vm nên mình sẽ sử dụng logwatch và chuyển tất cả các log về trung tâm làm việc, còn mình sử dụng 5 máy áo và sử dụng lần lượt khi data đã được backup và thể hiện flow data của nó chi tiết. Về phần vault, bạn có thể sử dụng tool Vault, nhưng khá khó chịu là khi setup lại server thì bạn phải làm lại từ đầu để nó cấp Role_id và Secrect_id, nhưng đây mới là cái khó khiến ransomware không thể tấn công vào được

## Về phần thực hành

**Note : dữ liệu này em hiện đang mô phỏng và em chưa lấy bất kì dữ liệu nào ngoài đời thực hay bất kỳ một dữ liệu sensitive data nào áp dụng cho hệ thống này** 

Đầu tiên khi production được nhận dữ liệu từ agent thì nó sẽ vào file data và chưa có phân loại chỉ là một file tổng hợp lại tất cả các dữ liệu đã được lưu trữ. Giờ chúng ta bắt đầu thực hiện một cách chi tiết và cụ thể nhất

## Production :

![image.png](./assets/backups/q.png)

Đây là những gì có trong file của production, phần data là dữ liệu được đưa vào, phần ouput thì khi phân loại dữ liệu xong nó sẽ gửi qua đó và chúng ta có thể nhận biết được data nào sẽ quan trọng dự vào policy mà chúng ta đã cấu hình

![image.png](./)

thì trong file scripts có những cái mà chúng ta cần thiết, thì phần production mình sẽ để ở bên dưới để các bạn có thể thấy rõ hơn

[https://www.notion.so/Production-334332f1e38180dfb616f76318ee778e?source=copy_link](https://www.notion.so/Production-334332f1e38180dfb616f76318ee778e?pvs=21)

Orchestrator :

Note : thực hiện hành động giám sát cũng như cấp token bên cạnh đó cấp key cho production thực hiện backup file

```jsx
jwt_secret.key =  // file chua key secret

client.json // file chua thong so va chinh sach cua nhung user nao duoc quyen cap token
{
  "prod-app-01": {
    "role": "app",
    "purpose": "tier1_export"
  },
  "prod-db-01": {
    "role": "db",
    "purpose": "tier1_export"
  },
  "prod-file-01": {
    "role": "file",
    "purpose": "tier1_export"
  }
}

b.py // file tao server va va cap token cho backup, production va snapshort
#!/usr/bin/env python3

from flask import Flask, request, jsonify
import jwt, datetime, json, uuid

app = Flask(__name__)

SECRET_KEY = open("/opt/orchestrator/jwt_secret.key").read().strip()
CLIENTS = json.load(open("/opt/orchestrator/a.json"))

HASH_DB = "/opt/orchestrator/hash.log"

def is_allowed_time(start, end):
    now = datetime.datetime.utcnow().time()
    s = datetime.datetime.strptime(start, "%H:%M").time()
    e = datetime.datetime.strptime(end, "%H:%M").time()
    return s <= now <= e if s <= e else (now >= s or now <= e)

@app.route("/get-token")
def get_token():
    host = request.args.get("host")
    role = request.args.get("role")
    purpose = request.args.get("purpose")

    if host not in CLIENTS:
        return jsonify({"error":"unknown host"}),403

    c = CLIENTS[host]

    if c["role"] != role or c["purpose"] != purpose:
        return jsonify({"error":"policy mismatch"}),403

    if "allowed_time" in c:
        if not is_allowed_time(c["allowed_time"]["start"], c["allowed_time"]["end"]):
            return jsonify({"error":"not allowed time"}),403

    now = datetime.datetime.utcnow()

    payload = {
        "iss":"orch-01",
        "aud":"backup_client",
        "host":host,
        "role":role,
        "purpose":purpose,
        "iat":now,
        "nbf":now,
        "exp":now + datetime.timedelta(minutes=10),
        "jti":str(uuid.uuid4())
    }

    token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")
    return jsonify({"token":token})

@app.route("/submit-hash", methods=["POST"])
def submit_hash():
    d = request.get_json()

    try:
        jwt.decode(d["token"], SECRET_KEY, algorithms=["HS256"], audience="backup_client")
    except:
        return jsonify({"error":"invalid token"}),403

    with open(HASH_DB,"a") as f:
        f.write(f"{datetime.datetime.utcnow()} | {d['host']} | {d['filename']} | {d['hash']}\n")

    return jsonify({"status":"ok"})

@app.route("/get-hash")
def get_hash():
    filename = request.args.get("filename")

    for line in open(HASH_DB):
        if filename in line:
            return jsonify({"hash":line.strip().split("|")[-1].strip()})

    return jsonify({"error":"not found"}),404

@app.route("/approve-snapshot", methods=["POST"])
def approve_snapshot():
    try:
        jwt.decode(request.json["token"], SECRET_KEY, algorithms=["HS256"], audience="backup_client")
    except Exception as e:
        return jsonify({"status":"denied","error":str(e)}),403

    return jsonify({"status":"approved"})

@app.route("/backup/request")
def backup_request():
    return jsonify({
        "approved": True,
        "session": str(uuid.uuid4()),
        "retention_days": 7
    })

@app.route("/backup/complete", methods=["POST"])
def backup_complete():
    return jsonify({"status":"ok"})

app.run(host="192.168.56.10", port=8080)
```

khi đã chạy xong hoàn tất thì file được zip sẽ chuyển qua cho staging để thực hiện snapshot 

## #Snapshot

khi file đã được đẩy từ production qua snapshot thì chúng ta sẽ thực hiện snapshot

![image.png](attachment:91e6d5fe-8119-44cd-b2ae-7901907c6845:image.png)

khi data được đẩy vào incoming thì chúng ta sẽ thực hiện snapshot theo như đúng quy trình của chúng ta

theo cấu trúc như file như trên thì chúng ta sẽ thực hiện snapshot. Em sẽ chạy file **k.sh** trong folder scripts của snapshot để thực hiện snapshot 

![image.png](attachment:a1b84dc1-2cf6-4c4c-a42d-2f7308486187:image.png)

khi thực hiện xong thì file trong incoming sẽ bị xóa và snapshot sẽ được ghi vào folder snapshot và log thì sẽ được ghi vào file snapshot.log chúng ta có thể kiểm tra nó ở phía trên em đang liệt kê. Còn file manifest sẽ lưu lại file nào được snapshot và id của file đó

![image.png](attachment:05a441f0-b347-41a0-ad74-08ea5ddd2ff5:image.png)

vậy là quá trình snapshot của chúng ta đã xong, dưới đây là những gì có trong quá trình này từ log cho đến snapshot

```jsx

snapshot_policy.json // day la policy cua snapshot 
{
 "snapshot_enabled": true,
 "allowed_hosts": [
  "stage1",
  "192.168.56.12"
 ],
 "max_snapshot_per_hour": 10
}

Ve phan snapshot thi co file
k.sh // file nay de thuc hien xin token va snapshot
#!/bin/bash

INCOMING="/data/staging/app/incoming"
SNAPSHOTS="/data/staging/app/snapshots"
MANIFESTS="/data/staging/app/manifests"
LOG="/data/staging/app/logs/snapshot.log"

ORCHESTRATOR="http://192.168.56.10:8080"

for file in $INCOMING/.tar.gz /// them dau * vao day
do

 if [ ! -f "$file" ]; then
  continue
 fi

 echo "$(date) found archive $file" >> $LOG

 echo "$(date) requesting snapshot approval" >> $LOG

 RESPONSE=$(curl -s -X POST $ORCHESTRATOR/approve-snapshot)

 if echo $RESPONSE | grep approved > /dev/null
 then

  TIMESTAMP=$(date +%s)

  SNAPSHOT="$SNAPSHOTS/app_snapshot_${TIMESTAMP}.tar.gz"

  mv "$file" "$SNAPSHOT"

  echo "$(date) snapshot created $SNAPSHOT" >> $LOG

  MANIFEST="$MANIFESTS/app_snapshot_${TIMESTAMP}.json"

  cat <<EOF > $MANIFEST
{
 "snapshot_id":"$TIMESTAMP",
 "snapshot_file":"$(basename $SNAPSHOT)",
 "created_at":"$(date)"
}
EOF

  echo "$(date) manifest created $MANIFEST" >> $LOG

 else

  echo "$(date) snapshot denied by orchestrator" >> $LOG

 fi

done
```

Vậy là phần snapshot của chúng ta đẫ hoàn thành, giờ chúng ta sẽ đến phần đẩy snapshot lên backup_storage, đây là cầu nối trong hệ thống của em, lý do  mà em không xóa phần snapshot là vì hệ thống của em nếu backup_storage bị compromise thì còn có snapshot, đây là một trong những điều tiên quyết của hệ thống này, mặc dù mọi thứ bị cô lập nhưng chúng luôn liên quan đến nhau và có sự kết nối với nhau.

Đẩy snapshot lên backup và lưu trữ tạm thời :

```jsx
Orchestrator/backup/ // trong nay se chua policy cung nhu file cap token cho staging
// ben canh do no con chua file luu backup session
backup_policy_engine.py
import datetime
import sqlite3
import json
import time

app = Flask(__name__)

POLICY_FILE="/opt/orchestrator/backup/policies/backup_policy.json"
DB="/opt/orchestrator/backup/db/backup_sessions.db"

def load_policy():
    with open(POLICY_FILE) as f:
        return json.load(f)

def db():
    return sqlite3.connect(DB)

@app.route("/backup/request")

def backup_request():

    host=request.args.get("host")
    role=request.args.get("role")
    snapshot=request.args.get("snapshot")

    if not host or not role or not snapshot:
        return jsonify({"approved":False})

    policy=load_policy()

    if host not in policy["allowed_hosts"]:
        return jsonify({"approved":False})

    if role not in policy["allowed_roles"]:
        return jsonify({"approved":False})

    if policy["snapshot_pattern"] not in snapshot:
        return jsonify({"approved":False})

    hour=datetime.datetime.now().hour
    start=policy["backup_window"]["start"]
    end=policy["backup_window"]["end"]

    if hour < start or hour > end:
        return jsonify({"approved":False})

    session=str(int(time.time()))

    conn=db()
    c=conn.cursor()

    c.execute(
        "INSERT INTO sessions(host,snapshot,session_id,status,created_at) VALUES (?,?,?,?,?)",
        (host,snapshot,session,"approved",datetime.datetime.utcnow())
    )

    conn.commit()

    return jsonify({
        "approved":True,
        "session":session,
        "retention_days":policy["retention_days"]
    })

@app.route("/backup/complete",methods=["POST"])

def complete():

    session=request.json["session"]

    conn=db()
    c=conn.cursor()

    c.execute(
        "UPDATE sessions SET status='completed' WHERE session_id=?",
        (session,)
    )

    conn.commit()

    return jsonify({"status":"recorded"})

app.run(host="192.168.56.10",port=8090)

backup_policy.json // file chua policy backup
{
  "allowed_hosts": [
    "staging"
  ],
  "allowed_roles": [
    "staging"
  ],
  "snapshot_pattern": "snapshot_",
  "backup_window": {
    "start": 0, // thoi gian cho phep backup
    "end": 23
  },
  "retention_days": 7

Staging // chua file xin token va day len backup /usr/local/bin/backup_worker.sh
backup_worker.sh
#!/usr/bin/env bash

HOST=$(hostname -s)

ROLE="staging"

ORCH="http://192.168.56.10:8090"

SNAPDIR="/data/staging/app/snapshots"

BACKUP_SERVER="192.168.56.13"

BACKUP_PATH="/backup/storage/app"

LOG="/data/staging/app/logs/backup.log"

log(){
echo "$(date) | $HOST | backup | $1" >> $LOG
}

while true
do

for file in $SNAPDIR/.tar.gz
do

[ -e "$file" ] || continue

SNAP=$(basename "$file")

log "request backup permission $SNAP"

RESP=$(curl -s "$ORCH/backup/request?host=$HOST&role=$ROLE&snapshot=$SNAP")

APPROVED=$(echo $RESP | jq -r '.approved')

SESSION=$(echo $RESP | jq -r '.session')

RETENTION=$(echo $RESP | jq -r '.retention_days')

if [ "$APPROVED" != "true" ]
then

log "backup denied by policy"

continue

fi

log "backup approved session=$SESSION retention=$RETENTION"

scp "$file" root@$BACKUP_SERVER:$BACKUP_PATH/

curl -X POST "$ORCH/backup/complete" \
-H "Content-Type: application/json" \
-d "{\"session\":\"$SESSION\"}"

log "backup completed $SNAP"

done

sleep 15

done
```

Em sẽ thực hiện luồng dữ liệu của em

![image.png](attachment:e0672c7a-07f9-409c-896a-8f32258c5c97:image.png)

đây là file dữ liệu chưa có gì và sau khi em thực hiện backup 

![image.png](attachment:0c78b0b0-a672-48df-9443-4c876b46d9f3:image.png)

đây là tổng quan của hệ thống em khi thực hiện đẩy backup hoàn tất, staging yêu cầu phiên backup và orchestrator cấp token backup rồi đẩy file lên backup_storage, đây chỉ mới là luồng backup còn khá là cơ bản và thiếu chuyên nghiệp có thể một vài ngày tới em sẽ nâng cấp hệ thống của mình và thực hiện lại những điều này.

## #Backup store phân loại data và đẩy data sensitive vào vault

```jsx
Process.sh //
#!/bin/bash
set -euo pipefail

INPUT=$1
WORK=/tmp/work
OUT=/tmp/processed.tar.gz

echo "[+] Cleaning workspace..."
rm -rf "$WORK"
mkdir -p "$WORK"

echo "[+] Extracting $INPUT..."
tar -xzf "$INPUT" -C "$WORK"

echo "[+] Filtering important data..."

IMPORTANT="$WORK/important"
mkdir -p "$IMPORTANT"

# 🔥 chỉ tìm file (không lấy thư mục), tránh loop
find "$WORK" -type f ! -path "$IMPORTANT/*" -print0 | while IFS= read -r -d '' f
do
    if grep -q "PATIENT" "$f" 2>/dev/null; then
        cp "$f" "$IMPORTANT/"
    fi
done

# 🔥 fallback nếu không có dữ liệu quan trọng
if [ -z "$(ls -A "$IMPORTANT")" ]; then
    echo "[!] No important data found → backup all"

    find "$WORK" -mindepth 1 ! -path "$IMPORTANT" ! -path "$IMPORTANT/*" -exec cp -r {} "$IMPORTANT/" \;
fi

echo "[+] Repacking..."

tar -czf "$OUT" -C "$IMPORTANT" .

echo "[+] Done → $OUT"

push.sh //

#!/bin/bash

DIR="/backup/storage/app"
LATEST=$(ls -t $DIR/.tar.gz | head -n 1) // thieu dau *

if [ -z "$LATEST" ]; then
    echo "[-] No backup file found"
    exit 1
fi

echo "[+] Latest file: $LATEST"

# xử lý
./process.sh "$LATEST"

# lấy token từ orchestrator
echo "[+] Getting token..."
TOKEN=$(curl -s http://192.168.56.10:5000/get-token | jq -r .auth.client_token)

if [ "$TOKEN" == "null" ]; then
    echo "[-] Cannot get token"
    exit 1
fi

echo "[+] Encoding..."

DATA=$(base64 -w 0 /tmp/processed.tar.gz)

KEY=$(basename $LATEST | cut -d'.' -f1)

echo "[+] Uploading to Vault..."

curl -s \
  --header "X-Vault-Token: $TOKEN" \
  --request POST \
  --data "{\"data\":{\"file\":\"$DATA\"}}" \
  http://192.168.56.20:8200/v1/backup/data/$KEY

echo
echo "[+] SUCCESS → Stored as: $KEY"
```

về phần setup vault thì em đã có một bài lab làm về điều này và em sẽ thực hiện backup luôn chứ không cần phải setup lại thêm một lệnh nào nữa, vì vault là một server phải kích hoạt khi khởi tạo nên em sẽ kích hoạt nó

```jsx
Duoi day la nhung lenh dung de kich hoat vault
vault server-config=/etc/vault.d/vault.hcl // lenh kich hoat server cua vault
export VAULT_ADDR='http://192.168.56.20:8200' // lenh khoi tao moi truong
vault operator init // lenh giup vault gui key cho em
vault operator unseal // unseal vault den buoc nay em nhap key ma vault da give cho em
vault login <ROOT_TOKEN> // nhap key root ma vault da dua
vault policy write backup-policy backup-policy.hcl // viet backup vao vault
vault write -f auth/approle/role/backup-role/secret-id //xin vault cap lai secrect_id

```

đó là những lệnh chúng ta cần chạy khi kích hoạt vault và giờ em sẽ thực hiện backup 

![image.png](attachment:59ffdc48-161d-4886-83f3-82af0bc3b42b:image.png)

thì Orchestrator chính là thằng trung gian xin token từ vault và cấp về backup, khi backup xong thì bên vault sẽ cấp key cho chúng ta, key mà em vừa storage về vautl đó là key = app_snapshot_1773907850, key này em dùng để restore lại nếu data khi bị encryption hoặc bị tấn công.

[Production ](https://www.notion.so/Production-334332f1e38180dfb616f76318ee778e?pvs=21)
