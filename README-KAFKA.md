
# Kafka CLI Setup

## 📦 Install Kafka
- Download and install the **latest version of Kafka**.
- Unzip the downloaded archive.
- Navigate to the folder where you see `bin` and `config` directories.

## 🖥️ Open WSL CLI (For Windows Users)
- Open **WSL** in the Kafka directory path.
- Open **three separate WSL CLI windows**:
  1. One for **Zookeeper**.
  2. One for **Kafka server**.
  3. One for **Kafka CLI commands**.

---

## 🚀 Starting Kafka Services

### ✅ Start Kafka Zookeeper:
```bash
sh bin/zookeeper-server-start.sh config/zookeeper.properties
```

### ✅ Start Kafka Server:
```bash
sh bin/kafka-server-start.sh config/server.properties
```

---

## 🛠️ Kafka Miscellaneous CLI Commands

### 📌 Create a Topic:
```bash
sh bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic NewTopic --partitions 3 --replication-factor 1
```

### 📌 List All Topics:
```bash
sh bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

### 📌 Describe a Topic:
```bash
sh bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic NewTopic
```

### 📌 Produce Messages to a Topic:
```bash
sh bin/kafka-console-producer.sh --broker-list localhost:9092 --topic NewTopic
```

### 📌 Consume Messages from a Topic:
```bash
sh bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic NewTopic --from-beginning
```

---

## ⚙️ Note:
> If you are running Kafka server using **KRaft mode** (Kafka Raft mode), **you don't need to start Zookeeper**.

---

## 🔗 Configuring Kafka with Spring Boot (When Kafka Runs in WSL)

### ❗ Problem:
- **Spring Boot** application runs on **localhost (Windows)**.
- **Kafka Server** runs on **WSL**, which has a **different IP address**.
- Direct connection fails unless **port forwarding** is set up.

---

### ✅ Solution:

### 1. Find WSL Host IP Address:
Run this command inside WSL:
```bash
hostname -I
```

---

### 2. Port Forwarding (Windows PowerShell):
Run this command in **Windows PowerShell (Admin)**:
```powershell
netsh interface portproxy add v4tov4 listenaddress=127.0.0.1 listenport=9092 connectaddress=<WSL-IP> connectport=9092
```
> Replace `<WSL-IP>` with the IP address obtained from `hostname -I`.

---

### 3. Modify Kafka `server.properties` Configuration:
Open `config/server.properties` and make the following changes:
```properties
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://127.0.0.1:9092
```

➡️ **Restart Kafka server** after making these changes.

---

### 4. Verify Port Mapping:
Run this command in **PowerShell**:
```powershell
Test-NetConnection 127.0.0.1 -Port 9092
```

✅ If `TcpTestSucceeded : True`, Kafka is now accessible on `127.0.0.1:9092`.

---

### 5. Use Correct Kafka Broker in Spring Boot:
In `application.yml` or `application.properties`:
```yaml
spring:
  kafka:
    bootstrap-servers: 127.0.0.1:9092
```

✅ Now Spring Boot and Kafka (running in WSL) can communicate seamlessly.

---

### 🚀 You're all set! Kafka is ready to use with your Spring Boot application.
