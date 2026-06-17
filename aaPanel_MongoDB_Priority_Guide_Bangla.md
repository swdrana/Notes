# aaPanel দিয়ে MongoDB + Adminer + Node.js সম্পূর্ণ সেটআপ গাইড
## Ubuntu 24.04 VPS-এর জন্য

এই গাইডটি এমনভাবে লেখা হয়েছে যাতে একবার পড়ে ধাপে ধাপে অনুসরণ করলেই কাজ হয়ে যায়। এখানে **aaPanel-কে প্রধান পদ্ধতি** ধরা হয়েছে। যেখানে aaPanel যথেষ্ট নয়, শুধু সেখানে ছোট terminal step রাখা হয়েছে।

---

## আপনার বর্তমান লক্ষ্য

এই সেটআপে আপনি করতে পারবেন:

- aaPanel থেকে MongoDB manage
- database create
- username/password set
- Adminer দিয়ে MongoDB browse
- Node.js / Next.js / NestJS থেকে connect
- public URI বা local URI ব্যবহার
- PHP Adminer-এ `mongodb` extension চালু

---

# ১) MongoDB install করার আগে কী জানা দরকার

আপনার VPS:

- Ubuntu 24.04
- aaPanel installed
- MongoDB 7.0
- MongoDB server local mode-এ চলছে
- MongoDB user `admin` database-এর ভিতরে রাখা ভালো
- একই VPS-এর app-এর জন্য `127.0.0.1` সবচেয়ে নিরাপদ

---

# ২) MongoDB install করা

## ২.১ aaPanel থেকে চেষ্টা

aaPanel → App Store → MongoDB

যদি install button কাজ করে, সেটাই ব্যবহার করুন।

কিন্তু Ubuntu 24.04-এ aaPanel MongoDB installer অনেক সময় fail করতে পারে। তখন terminal দিয়ে install করতে হবে।

## ২.২ terminal দিয়ে MongoDB repository যোগ করা

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

## ২.৩ MongoDB install

```bash
sudo apt install -y mongodb-org
```

## ২.৪ service চালু করা

```bash
sudo systemctl daemon-reload
sudo systemctl enable mongod
sudo systemctl start mongod
sudo systemctl status mongod
```

## ২.৫ version check

```bash
mongosh --eval "db.version()"
```

---

# ৩) MongoDB user তৈরি করা

MongoDB shell খুলুন:

```bash
mongosh
```

তারপর:

```javascript
use admin

db.createUser({
  user: "root",
  pwd: "rayhan",
  roles: [
    { role: "root", db: "admin" }
  ]
})
```

User তৈরি হয়েছে কিনা দেখুন:

```javascript
db.getUsers()
```

Exit:

```javascript
exit
```

---

# ৪) Authentication enable করা

Config file খুলুন:

```bash
nano /etc/mongod.conf
```

ফাইলের শেষে যোগ করুন:

```yaml
security:
  authorization: enabled
```

Save করুন:

- `CTRL + X`
- `Y`
- `ENTER`

তারপর restart:

```bash
sudo systemctl restart mongod
```

Login test:

```bash
mongosh -u root -p rayhan --authenticationDatabase admin
```

---

# ৫) MongoDB কোথায় listen করছে দেখা

```bash
ss -tlnp | grep 27017
```

যদি দেখায়:

```text
127.0.0.1:27017
```

তাহলে MongoDB শুধু VPS-এর ভিতর থেকে access করা যাবে।

---

# ৬) aaPanel-এ MongoDB connect করা

aaPanel → MongoDB

## ৬.১ Database add করা

- Click: **Add DB**
- Database name: `absacademy`
- Username: `absacademy`
- Password: strong password

## ৬.২ Get DB from server

যদি aaPanel database detect না করে, click করুন:

- **Get DB from server**

## ৬.৩ Remote DB add করা

- **Add Remote DB**

তারপর দিন:

```text
Host: 127.0.0.1
Port: 27017
Username: root অথবা absacademy
Password: আপনার password
```

---

# ৭) aaPanel-এ Root password / Security authentication error এলে

কখনো কখনো aaPanel এই error দেখাতে পারে:

```text
Local login password is empty
```

এর মানে সাধারণত:

- MongoDB server খারাপ হয়নি
- aaPanel plugin-এর internal record খালি
- aaPanel UI feature পুরোপুরি sync হয়নি

এ অবস্থায়:

- **Get DB from server** দিন
- page refresh করুন
- database list verify করুন
- **Add DB** বা **Add Remote DB** দিয়ে database তৈরি/যোগ করুন

---

# ৮) Adminer-এর জন্য PHP mongodb extension চালু করা

Adminer 5.4.0 যদি এই error দেয়:

```text
None of the supported PHP extensions (mongodb) are available
```

তাহলে MongoDB server-এ সমস্যা না, PHP-তে `mongodb` extension নেই।

আপনার aaPanel PHP path সাধারণত:

```text
/www/server/php/83/
```

## ৮.১ aaPanel থেকে extension install করার চেষ্টা

aaPanel → App Store → PHP 8.3 → Extensions

Search করুন:

```text
mongodb
```

Install করুন।

## ৮.২ যদি aaPanel-এ না পাওয়া যায়, terminal দিয়ে install

```bash
apt update
apt install php8.3-dev php-pear gcc g++ make -y
```

তারপর:

```bash
pecl install mongodb
```

## ৮.৩ aaPanel PHP-তে extension enable করা

aaPanel PHP config:

```text
/www/server/php/83/etc/php.ini
```

এখানে শেষে যোগ করুন:

```ini
extension=mongodb.so
```

copy করতে হতে পারে:

```bash
cp /usr/lib/php/20230831/mongodb.so /www/server/php/83/lib/php/extensions/no-debug-non-zts-20230831/
```

test:

```bash
/www/server/php/83/bin/php -m | grep mongodb
```

---

# ৯) aaPanel PHP restart

aaPanel-এর নিজের PHP-FPM restart command server ভেদে আলাদা হতে পারে।

আগে দেখুন:

```bash
ls /etc/init.d/ | grep php
```

যদি `php-fpm-83` থাকে, তাহলে:

```bash
/etc/init.d/php-fpm-83 restart
```

না থাকলে aaPanel panel restart দিন:

```bash
bt
```

---

# ১০) MongoDB user/database verify করা

MongoDB shell-এ login করুন:

```bash
mongosh -u root -p rayhan --authenticationDatabase admin
```

তারপর:

```javascript
use admin
db.getUsers()
show dbs
```

---

# ১১) URI কীভাবে বানাবেন

## ১১.১ একই VPS-এর app-এর জন্য Local URI

```env
MONGODB_URI=mongodb://absacademy:YOUR_PASSWORD@127.0.0.1:27017/absacademy?authSource=absacademy
```

অথবা root user হলে:

```env
MONGODB_URI=mongodb://root:rayhan@127.0.0.1:27017/absacademy?authSource=admin
```

## ১১.২ Public URI

```env
MONGODB_URI=mongodb://absacademy:YOUR_PASSWORD@72.60.38.253:27017/absacademy?authSource=absacademy
```

অথবা root user হলে:

```env
MONGODB_URI=mongodb://root:rayhan@72.60.38.253:27017/absacademy?authSource=admin
```

---

# ১২) Public access চালু করা

**সতর্কতা:** public access শুধু দরকার হলে খুলবেন।

```bash
nano /etc/mongod.conf
```

এই লাইন:

```yaml
bindIp: 127.0.0.1
```

পরিবর্তন করুন:

```yaml
bindIp: 0.0.0.0
```

তারপর:

```bash
sudo systemctl restart mongod
ufw allow 27017/tcp
ss -tlnp | grep 27017
```

---

# ১৩) aaPanel দিয়ে database manage করার নিয়ম

## Add DB
নতুন database এবং user create করতে।

## Delete
পুরোনো database/user remove করতে।

## Password
বর্তমান database user-এর password বদলাতে।

## Import
server থেকে existing database sync করতে।

## Get DB from server
server-এর database list aaPanel-এ আনতে।

## Adminer
browser দিয়ে database দেখতে।

## Remote DB
local MongoDB server-কে aaPanel-এ manually add করতে।

---

# ১৪) Node.js / Next.js connection example

## npm package install

```bash
npm install mongoose
```

## connection file

```javascript
import mongoose from "mongoose";

const MONGODB_URI = process.env.MONGODB_URI;

if (!MONGODB_URI) {
  throw new Error("MONGODB_URI is missing");
}

let cached = global.mongoose;

if (!cached) {
  cached = global.mongoose = { conn: null, promise: null };
}

export async function connectDB() {
  if (cached.conn) return cached.conn;

  if (!cached.promise) {
    cached.promise = mongoose.connect(MONGODB_URI);
  }

  cached.conn = await cached.promise;
  return cached.conn;
}
```

## .env

```env
MONGODB_URI=mongodb://absacademy:YOUR_PASSWORD@127.0.0.1:27017/absacademy?authSource=absacademy
```

---

# ১৫) কাজ না করলে আগে কী চেক করবেন

## MongoDB service চলছে?

```bash
systemctl status mongod
```

## port open আছে?

```bash
ss -tlnp | grep 27017
```

## user list আছে?

```javascript
use admin
db.getUsers()
```

## database list আছে?

```javascript
show dbs
```

## PHP extension আছে?

```bash
/www/server/php/83/bin/php -m | grep mongodb
```

---

# ১৬) আপনার জন্য recommended final setup

সবচেয়ে ভালো setup:

- MongoDB server: installed and running
- Authentication: enabled
- Database: `absacademy`
- User: `absacademy`
- App URI: local `127.0.0.1`
- aaPanel: database manage করা
- Adminer: `mongodb` extension enabled
- Public IP: শুধু দরকার হলে

---

# ১৭) Final local URI

```env
mongodb://absacademy:YOUR_PASSWORD@127.0.0.1:27017/absacademy?authSource=absacademy
```

যদি `root` user ব্যবহার করেন:

```env
mongodb://root:rayhan@127.0.0.1:27017/absacademy?authSource=admin
```

---

# ১৮) Final public URI

```env
mongodb://absacademy:YOUR_PASSWORD@72.60.38.253:27017/absacademy?authSource=absacademy
```

অথবা:

```env
mongodb://root:rayhan@72.60.38.253:27017/absacademy?authSource=admin
```

---

# ১৯) সবচেয়ে জরুরি নিয়ম

যদি app একই VPS-এ থাকে, public IP ব্যবহার করবেন না।

একই VPS-এর জন্য:

```text
127.0.0.1
```

ই যথেষ্ট।

Public IP শুধু তখনই দরকার যখন বাইরের computer বা অন্য server থেকে connect করবেন।

---

# ২০) এক লাইনের সারাংশ

aaPanel দিয়ে database create করুন, MongoDB user set করুন, authentication চালু করুন, Adminer-এর জন্য PHP `mongodb` extension enable করুন, আর same VPS-এর app-এর জন্য `127.0.0.1` URI ব্যবহার করুন।
