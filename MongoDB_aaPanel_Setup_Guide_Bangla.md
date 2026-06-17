# Ubuntu 24.04 + aaPanel + MongoDB 7.0 সম্পূর্ণ সেটআপ গাইড

## এই গাইডে কী আছে

এই গাইড অনুসরণ করলে আপনি:

- Ubuntu 24.04 VPS-এ MongoDB 7.0 Install করতে পারবেন
- MongoDB Authentication Enable করতে পারবেন
- MongoDB User তৈরি করতে পারবেন
- aaPanel-এর সাথে MongoDB Connect করতে পারবেন
- Database তৈরি করতে পারবেন
- Node.js / Next.js / NestJS থেকে Connect করতে পারবেন

---

## ধাপ ১: MongoDB Repository যোগ করা

```bash
sudo apt update
sudo apt install curl gnupg -y
```

```bash
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg
```

```bash
echo "deb [ arch=amd64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

```bash
sudo apt update
```

```bash
apt-cache policy mongodb-org
```

---

## ধাপ ২: MongoDB Install

```bash
sudo apt install -y mongodb-org
```

---

## ধাপ ৩: MongoDB চালু করা

```bash
sudo systemctl daemon-reload
sudo systemctl enable mongod
sudo systemctl start mongod
sudo systemctl status mongod
```

---

## ধাপ ৪: Version Check

```bash
mongosh --eval "db.version()"
```

---

## ধাপ ৫: MongoDB Shell-এ প্রবেশ

```bash
mongosh
```

## Root User তৈরি

```javascript
use admin

db.createUser({
  user: "root",
  pwd: "rayhan",
  roles: [
    {
      role: "root",
      db: "admin"
    }
  ]
})
```

```javascript
db.getUsers()
exit
```

---

## ধাপ ৬: Authentication Enable

```bash
nano /etc/mongod.conf
```

ফাইলের শেষে যোগ করুন:

```yaml
security:
  authorization: enabled
```

Restart:

```bash
sudo systemctl restart mongod
```

---

## ধাপ ৭: Login Test

```bash
mongosh -u root -p rayhan --authenticationDatabase admin
```

---

## ধাপ ৮: MongoDB কোন IP-তে Listen করছে

```bash
ss -tlnp | grep 27017
```

Expected:

```text
127.0.0.1:27017
```

---

## ধাপ ৯: aaPanel-এর সাথে Connect

MongoDB → Add Remote DB

```text
DB Address: 127.0.0.1
Port: 27017
Username: root
Password: rayhan
```

---

## ধাপ ১০: Database তৈরি

Database Name:

```text
absacademy
```

---

## Node.js / Next.js URI

```env
MONGODB_URI=mongodb://root:rayhan@127.0.0.1:27017/absacademy?authSource=admin
```

---

## Public URI (প্রয়োজন হলে)

```env
mongodb://root:rayhan@72.60.38.253:27017/absacademy?authSource=admin
```

### Public Access Enable

```bash
nano /etc/mongod.conf
```

পরিবর্তন করুন:

```yaml
bindIp: 0.0.0.0
```

```bash
sudo systemctl restart mongod
sudo ufw allow 27017/tcp
```

---

## প্রয়োজনীয় Command Summary

```bash
sudo systemctl start mongod
sudo systemctl stop mongod
sudo systemctl restart mongod
sudo systemctl status mongod
```

```bash
mongosh -u root -p rayhan --authenticationDatabase admin
```

```javascript
show dbs
```

```javascript
use admin
db.getUsers()
```

---

## Final Working URI

```env
mongodb://root:rayhan@127.0.0.1:27017/absacademy?authSource=admin
```
