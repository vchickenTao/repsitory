**<span style="font-size: 35px;">ð MySQL åºç¡ç¯ -Â æåºæ¬çSELECTè¯­å¥</span>**

---

# æåºæ¬çSELECTè¯­å¥

## 1. SQLè¯­è¨çè§ååè§è

### 1) åºæ¬è§å

* SQL å¯ä»¥åå¨ä¸è¡æèå¤è¡ãä¸ºäºæé«å¯è¯»æ§ï¼åå­å¥åè¡åï¼å¿è¦æ¶ä½¿ç¨ç¼©è¿ 
* æ¯æ¡å½ä»¤ä»¥ ; æ \g æ \G ç»æ 
* å³é®å­ä¸è½è¢«ç¼©åä¹ä¸è½åè¡ 
* å³äºæ ç¹ç¬¦å· 
  * å¿é¡»ä¿è¯ææç()ãåå¼å·ãåå¼å·æ¯æå¯¹ç»æç 
  * å¿é¡»ä½¿ç¨è±æç¶æä¸çåè§è¾å¥æ¹å¼ 
  * å­ç¬¦ä¸²ååæ¥ææ¶é´ç±»åçæ°æ®å¯ä»¥ä½¿ç¨åå¼å·ï¼' 'ï¼è¡¨ç¤º 
  * åçå«åï¼å°½éä½¿ç¨åå¼å·ï¼" "ï¼ï¼èä¸ä¸å»ºè®®çç¥as

### 2) SQLå¤§å°åè§èï¼å»ºè®®éµå®ï¼

* MySQL å¨ Windows ç¯å¢ä¸æ¯å¤§å°åä¸ææç 
* MySQL å¨ Linux ç¯å¢ä¸æ¯å¤§å°åææç 
  * æ°æ®åºåãè¡¨åãè¡¨çå«åãåéåæ¯ä¸¥æ ¼åºåå¤§å°åç 
  * å³é®å­ãå½æ°åãåå(æå­æ®µå)ãåçå«å(å­æ®µçå«å) æ¯å¿½ç¥å¤§å°åçã 
* æ¨èéç¨ç»ä¸çä¹¦åè§èï¼ 
  * æ°æ®åºåãè¡¨åãè¡¨å«åãå­æ®µåãå­æ®µå«åç­é½å°å 
  * SQL å³é®å­ãå½æ°åãç»å®åéç­é½å¤§å

### 3) æ³¨é

```mysql
åè¡æ³¨éï¼#æ³¨éæå­(MySQLç¹æçæ¹å¼)
åè¡æ³¨éï¼-- æ³¨éæå­(--åé¢å¿é¡»åå«ä¸ä¸ªç©ºæ ¼ã)
å¤è¡æ³¨éï¼/* æ³¨éæå­ */
```

### 4) å½åè§å

* æ°æ®åºãè¡¨åä¸å¾è¶è¿30ä¸ªå­ç¬¦ï¼åéåéå¶ä¸º29ä¸ª 
* å¿é¡»åªè½åå« AâZ, aâz, 0â9, _å±63ä¸ªå­ç¬¦ 
* æ°æ®åºåãè¡¨åãå­æ®µåç­å¯¹è±¡åä¸­é´ä¸è¦åå«ç©ºæ ¼ åä¸ä¸ªMySQLè½¯ä»¶ä¸­ï¼æ°æ®åºä¸è½ååï¼åä¸ä¸ªåºä¸­ï¼è¡¨ä¸è½éåï¼
* åä¸ä¸ªè¡¨ä¸­ï¼å­æ®µä¸è½éå å¿é¡»ä¿è¯ä½ çå­æ®µæ²¡æåä¿çå­ãæ°æ®åºç³»ç»æå¸¸ç¨æ¹æ³å²çªãå¦æåæä½¿ç¨ï¼è¯·å¨SQLè¯­å¥ä¸­ä½¿ ç¨`ï¼çéå·ï¼å¼èµ·æ¥ 
* ä¿æå­æ®µååç±»åçä¸è´æ§ï¼å¨å½åå­æ®µå¹¶ä¸ºå¶æå®æ°æ®ç±»åçæ¶åä¸å®è¦ä¿è¯ä¸è´æ§ãåå¦æ°æ® ç±»åå¨ä¸ä¸ªè¡¨éæ¯æ´æ°ï¼é£å¨å¦ä¸ä¸ªè¡¨éå¯å°±å«åæå­ç¬¦åäº

## 2. åºæ¬çSELECTè¯­å¥

### 1) SELECT ... FROM

* è¯­æ³

```mysql
SELECT æ è¯éæ©åªäºå
FROM æ è¯ä»åªä¸ªè¡¨ä¸­éæ©
```

* éæ©å¨é¨å

```mysql
SELECT *
FROM departments;
```

* éæ©ç¹å®çåï¼

```mysql
SELECT department_id, location_id
FROM departments;
```

### 2) åçå«å

* éå½åä¸ä¸ªå 
* ä¾¿äºè®¡ç® 
* ç´§è·ååï¼ä¹å¯ä»¥å¨åååå«åä¹é´å å¥å³é®å­ASï¼å«åä½¿ç¨åå¼å·ï¼ä»¥ä¾¿å¨å«åä¸­åå«ç©ºæ ¼æç¹ æ®çå­ç¬¦å¹¶åºåå¤§å°åã 
* AS å¯ä»¥çç¥ 
* å»ºè®®å«åç®ç­ï¼è§åç¥æ 
* ä¸¾ä¾ï¼

```mysql
SELECT last_name AS name, commission_pct comm
FROM employees;
```

### 3) å»é¤éå¤è¡

DISTINCTå³é®å­

```mysql
SELECT DISTINCT department_id FROM employees;
```

### 4) ç©ºå¼åä¸è¿ç®

ç©ºå¼ï¼null ( ä¸ç­åäº0, â â, ânullâ )

å®éé®é¢çè§£å³æ¹æ¡ï¼å¼å¥IFNULL

```mysql
SELECT employee_id, salary "æå·¥èµ", salary * (1 + IFNULL(commission_pct, 0)) * 12 "å¹´å·¥èµ" FROM employees;
```

è¿éä½ ä¸å®è¦æ³¨æï¼å¨ MySQL éé¢ï¼ ç©ºå¼ä¸ç­äºç©ºå­ç¬¦ä¸²ãä¸ä¸ªç©ºå­ç¬¦ä¸²çé¿åº¦æ¯ 0ï¼èä¸ä¸ªç©ºå¼çé¿ åº¦æ¯ç©ºãèä¸ï¼å¨ MySQL éé¢ï¼ç©ºå¼æ¯å ç¨ç©ºé´çã

### 5) çéå· ``

å¿é¡»ä¿è¯ä½ çå­æ®µæ²¡æåä¿çå­ãæ°æ®åºç³»ç»æå¸¸è§æ¹æ³å²çªã

å¦æåæä½¿ç¨ï¼å¨SQLè¯­å¥ä¸­ä½¿ç¨ \` \` å¼èµ·æ¥ã

```mysql
SELECT * FROM `order`;
```

### 6) æ¥è¯¢å¸¸æ°

```mysql
SELECT 'å°å¼ ç§æ' as "å¬å¸å", employee_id, last_name FROM employees;
```

## 3. æ¾ç¤ºè¡¨ç»æ

æ¾ç¤ºè¡¨ä¸­å­æ®µçè¯¦ç»ä¿¡æ¯

```mysql
DESCRIBE employees;
æ
DESC employees;
```

```mysql
mysql> desc employees;
+----------------+-------------+------+-----+---------+-------+
| Field | Type | Null | Key | Default | Extra |
+----------------+-------------+------+-----+---------+-------+
| employee_id | int(6) | NO | PRI | 0 | |
| first_name | varchar(20) | YES | | NULL | |
| last_name | varchar(25) | NO | | NULL | |
| email | varchar(25) | NO | UNI | NULL | |
| phone_number | varchar(20) | YES | | NULL | |
| hire_date | date | NO | | NULL | |
| job_id | varchar(10) | NO | MUL | NULL | |
| salary | double(8,2) | YES | | NULL | |
| commission_pct | double(2,2) | YES | | NULL | |
| manager_id | int(6) | YES | MUL | NULL | |
| department_id | int(4) | YES | MUL | NULL | |
+----------------+-------------+------+-----+---------+-------+
11 rows in set (0.00 sec)
```

å¶ä¸­ï¼åä¸ªå­æ®µçå«ä¹åå«è§£éå¦ä¸ï¼ 

* Fieldï¼è¡¨ç¤ºå­æ®µåç§°ã 
* Typeï¼è¡¨ç¤ºå­æ®µç±»åï¼è¿é barcodeãgoodsname æ¯ææ¬åçï¼price æ¯æ´æ°ç±»åçã 
* Nullï¼è¡¨ç¤ºè¯¥åæ¯å¦å¯ä»¥å­å¨NULLå¼ã 
* Keyï¼è¡¨ç¤ºè¯¥åæ¯å¦å·²ç¼å¶ç´¢å¼ã
* PRIè¡¨ç¤ºè¯¥åæ¯è¡¨ä¸»é®çä¸é¨åï¼
* UNIè¡¨ç¤ºè¯¥åæ¯UNIQUEç´¢å¼çä¸ é¨åï¼
* MULè¡¨ç¤ºå¨åä¸­æä¸ªç»å®å¼åè®¸åºç°å¤æ¬¡ã 
* Defaultï¼è¡¨ç¤ºè¯¥åæ¯å¦æé»è®¤å¼ï¼å¦ææï¼é£ä¹å¼æ¯å¤å°ã 
* Extraï¼è¡¨ç¤ºå¯ä»¥è·åçä¸ç»å®åæå³çéå ä¿¡æ¯ï¼ä¾å¦AUTO_INCREMENTç­ã

## 4. è¿æ»¤æ°æ®

* è¯­æ³ï¼

```mysql
SELECT å­æ®µ1,å­æ®µ2
FROM è¡¨å
WHERE è¿æ»¤æ¡ä»¶
```

ä½¿ç¨WHERE å­å¥ï¼å°ä¸æ»¡è¶³æ¡ä»¶çè¡è¿æ»¤æãWHEREå­å¥ç´§é FROMå­å¥ã

* ä¸¾ä¾ï¼

```mysql
SELECT employee_id, last_name, job_id, department_id
FROM employees
WHERE department_id = 90;
```

# åèèµæ

- [å°ç¡è°·-å®çº¢åº·èå¸ï¼MySQLæ°æ®åºæç¨å¤©è±æ¿ï¼mysqlå®è£å°mysqlé«çº§ï¼å¼ºï¼ç¡¬ï¼](https://www.bilibili.com/video/BV1iq4y1u7vj)