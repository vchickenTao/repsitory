# ð **MySQL  å­å¨å¼æç¯**

---

## 1. æ¥çå­å¨å¼æ

* æ¥çmysqlæä¾ä»ä¹å­å¨å¼æ

```mysql
show engines;
```

![image-20220615223831995](MySQLæ¶æç¯.assets/image-20220615223831995.png)

## 2. è®¾ç½®ç³»ç»é»è®¤çå­å¨å¼æ

* æ¥çé»è®¤çå­å¨å¼æ

```mysql
show variables like '%storage_engine%';
#æ
SELECT @@default_storage_engine;
```

![](MySQLæ¶æç¯.assets/image-20220615224249491.png)

* ä¿®æ¹é»è®¤çå­å¨å¼æ

å¦æå¨åå»ºè¡¨çè¯­å¥ä¸­æ²¡ææ¾å¼æå®è¡¨çå­å¨å¼æçè¯ï¼é£å°±ä¼é»è®¤ä½¿ç¨ InnoDB ä½ä¸ºè¡¨çå­å¨å¼æã å¦ææä»¬æ³æ¹åè¡¨çé»è®¤å­å¨å¼æçè¯ï¼å¯ä»¥è¿æ ·åå¯å¨æå¡å¨çå½ä»¤è¡ï¼

```mysql
SET DEFAULT_STORAGE_ENGINE=MyISAM;
```

æèä¿®æ¹ my.cnf æä»¶ï¼

```mysql
default-storage-engine=MyISAM
# éå¯æå¡
systemctl restart mysqld.service
```

## 3. è®¾ç½®è¡¨çå­å¨å¼æ

å­å¨å¼ææ¯è´è´£å¯¹è¡¨ä¸­çæ°æ®è¿è¡æåååå¥å·¥ä½çï¼æä»¬å¯ä»¥ä¸º ä¸åçè¡¨è®¾ç½®ä¸åçå­å¨å¼æ ï¼ä¹å°±æ¯ è¯´ä¸åçè¡¨å¯ä»¥æä¸åçç©çå­å¨ç»æï¼ä¸åçæåååå¥æ¹å¼ã

### 3.1 åå»ºè¡¨æ¶æå®å­å¨å¼æ

æä»¬ä¹ååå»ºè¡¨çè¯­å¥é½æ²¡ææå®è¡¨çå­å¨å¼æï¼é£å°±ä¼ä½¿ç¨é»è®¤çå­å¨å¼æ InnoDB ãå¦ææä»¬æ³æ¾ å¼çæå®ä¸ä¸è¡¨çå­å¨å¼æï¼é£å¯ä»¥è¿ä¹åï¼

```mysql
CREATE TABLE è¡¨å(
å»ºè¡¨è¯­å¥;
) ENGINE = å­å¨å¼æåç§°;
```

### 3.2 ä¿®æ¹è¡¨çå­å¨å¼æ

å¦æè¡¨å·²ç»å»ºå¥½äºï¼æä»¬ä¹å¯ä»¥ä½¿ç¨ä¸è¾¹è¿ä¸ªè¯­å¥æ¥ä¿®æ¹è¡¨çå­å¨å¼æï¼

```mysql
ALTER TABLE è¡¨å ENGINE = å­å¨å¼æåç§°;
```

æ¯å¦æä»¬ä¿®æ¹ä¸ä¸ engine_demo_table è¡¨çå­å¨å¼æï¼

```mysql
mysql> ALTER TABLE engine_demo_table ENGINE = InnoDB;
```

è¿æ¶æä»¬åæ¥çä¸ä¸ engine_demo_table çè¡¨ç»æï¼

```mysql
mysql> SHOW CREATE TABLE engine_demo_table\G
*************************** 1. row ***************************
Table: engine_demo_table
Create Table: CREATE TABLE `engine_demo_table` (
`i` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.01 sec)
```

## 4. å¼æä»ç»

### 4.1 InnoDB å¼æï¼å·å¤å¤é®æ¯æåè½çäºå¡å­å¨å¼æ

* MySQLä»3.23.34aå¼å§å°±åå«InnoDBå­å¨å¼æã `å¤§äºç­äº5.5ä¹åï¼é»è®¤éç¨InnoDBå¼æ` ã
* InnoDBæ¯MySQLç é»è®¤äºå¡åå¼æ ï¼å®è¢«è®¾è®¡ç¨æ¥å¤çå¤§éçç­æ(short-lived)äºå¡ãå¯ä»¥ç¡®ä¿äºå¡çå®æ´æäº¤(Commit)ååæ»(Rollback)ã 
* é¤äºå¢å åæ¥è¯¢å¤ï¼è¿éè¦æ´æ°ãå é¤æä½ï¼é£ä¹ï¼åºä¼åéæ©InnoDBå­å¨å¼æã é¤éæéå¸¸ç¹å«çåå éè¦ä½¿ç¨å¶ä»çå­å¨å¼æï¼å¦ååºè¯¥ä¼åèèInnoDBå¼æã 
* æ°æ®æä»¶ç»æï¼ï¼å¨ãç¬¬02ç« _MySQLæ°æ®ç®å½ãç« èå·²è®²ï¼ 
  * è¡¨å.frm å­å¨è¡¨ç»æï¼MySQL8.0æ¶ï¼åå¹¶å¨è¡¨å.ibdä¸­ï¼ 
  * è¡¨å.ibd å­å¨æ°æ®åç´¢å¼ 
* InnoDBæ¯ ä¸ºå¤çå·¨å¤§æ°æ®éçæå¤§æ§è½è®¾è®¡ ã 
  * å¨ä»¥åççæ¬ä¸­ï¼å­å¸æ°æ®ä»¥åæ°æ®æä»¶ãéäºå¡è¡¨ç­æ¥å­å¨ãç°å¨è¿äºåæ°æ®æä»¶è¢«å é¤ äºãæ¯å¦ï¼ .frm ï¼ .par ï¼ .trn ï¼ .isl ï¼ .db.opt ç­é½å¨MySQL8.0ä¸­ä¸å­å¨äºã 
* å¯¹æ¯MyISAMçå­å¨å¼æï¼ InnoDBåçå¤çæçå·®ä¸äº ï¼å¹¶ä¸ä¼å ç¨æ´å¤çç£çç©ºé´ä»¥ä¿å­æ°æ®åç´¢å¼ã 
* MyISAMåªç¼å­ç´¢å¼ï¼ä¸ç¼å­çå®æ°æ®ï¼InnoDBä¸ä»ç¼å­ç´¢å¼è¿è¦ç¼å­çå®æ°æ®ï¼ å¯¹åå­è¦æ±è¾ é« ï¼èä¸åå­å¤§å°å¯¹æ§è½æå³å®æ§çå½±åã

### 4.2 MyISAM å¼æï¼ä¸»è¦çéäºå¡å¤çå­å¨å¼æ

* MyISAMæä¾äºå¤§éçç¹æ§ï¼åæ¬å¨æç´¢å¼ãåç¼©ãç©ºé´å½æ°(GIS)ç­ï¼ä½MyISAMä¸æ¯æäºå¡ãè¡çº§ éãå¤é® ï¼æä¸ä¸ªæ¯«æ çé®çç¼ºé·å°±æ¯å´©æºåæ æ³å®å¨æ¢å¤ ã
* 5.5ä¹åé»è®¤çå­å¨å¼æ 
* ä¼å¿æ¯è®¿é®çéåº¦å¿« ï¼å¯¹äºå¡å®æ´æ§æ²¡æè¦æ±æèä»¥SELECTãINSERTä¸ºä¸»çåºç¨ 
* éå¯¹æ°æ®ç»è®¡æé¢å¤çå¸¸æ°å­å¨ãæè count(*) çæ¥è¯¢æçå¾é« æ°æ®æä»¶ç»æï¼ï¼å¨ãç¬¬02ç« _MySQLæ°æ®ç®å½ãç« èå·²è®²ï¼ 
  * è¡¨å.frm å­å¨è¡¨ç»æ 
  * è¡¨å.MYD å­å¨æ°æ® (MYData) 
  * è¡¨å.MYI å­å¨ç´¢å¼ (MYIndex) 
* åºç¨åºæ¯ï¼åªè¯»åºç¨æèä»¥è¯»ä¸ºä¸»çä¸å¡

### 4.3 Archive å¼æï¼ç¨äºæ°æ®å­æ¡£

* ä¸è¡¨å±ç¤ºäºARCHIVE å­å¨å¼æåè½

![](MySQLæ¶æç¯.assets/image-20220616124743732.png)

### 4.4 Blackhole å¼æï¼ä¸¢å¼åæä½ï¼è¯»æä½ä¼è¿åç©ºåå®¹

### 4.5 CSV å¼æï¼å­å¨æ°æ®æ¶ï¼ä»¥éå·åéåä¸ªæ°æ®é¡¹

ä½¿ç¨æ¡ä¾å¦ä¸

```mysql
mysql> CREATE TABLE test (i INT NOT NULL, c CHAR(10) NOT NULL) ENGINE = CSV;
Query OK, 0 rows affected (0.06 sec)
mysql> INSERT INTO test VALUES(1,'record one'),(2,'record two');
Query OK, 2 rows affected (0.05 sec)
Records: 2 Duplicates: 0 Warnings: 0
mysql> SELECT * FROM test;
+---+------------+
| i |      c     |
+---+------------+
| 1 | record one |
| 2 | record two |
+---+------------+
2 rows in set (0.00 sec)
```

åå»ºCSVè¡¨è¿ä¼åå»ºç¸åºçåæä»¶ ï¼ç¨äº å­å¨è¡¨çç¶æ å è¡¨ä¸­å­å¨çè¡æ° ãæ­¤æä»¶çåç§°ä¸è¡¨çåç§°ç¸ åï¼åç¼ä¸º CSM ãå¦å¾æç¤º

![image-20220616125342599](MySQLæ¶æç¯.assets/image-20220616125342599.png)

å¦ææ£æ¥ test.CSV éè¿æ§è¡ä¸è¿°è¯­å¥åå»ºçæ°æ®åºç®å½ä¸­çæä»¶ï¼å¶åå®¹ä½¿ç¨Notepad++æå¼å¦ä¸ï¼

```mysql
"1","record one"
"2","record two"
```

è¿ç§æ ¼å¼å¯ä»¥è¢« Microsoft Excel ç­çµå­è¡¨æ ¼åºç¨ç¨åºè¯»åï¼çè³åå¥ãä½¿ç¨Microsoft Excelæå¼å¦å¾æç¤º

![image-20220616125448555](MySQLæ¶æç¯.assets/image-20220616125448555.png)

### 4.6 Memory å¼æï¼ç½®äºåå­çè¡¨

**æ¦è¿°ï¼**

Memoryéç¨çé»è¾ä»è´¨æ¯åå­ ï¼ååºéåº¦å¾å¿« ï¼ä½æ¯å½mysqldå®æ¤è¿ç¨å´©æºçæ¶åæ°æ®ä¼ä¸¢å¤± ãå¦å¤ï¼è¦æ±å­å¨çæ°æ®æ¯æ°æ®é¿åº¦ä¸åçæ ¼å¼ï¼æ¯å¦ï¼BlobåTextç±»åçæ°æ®ä¸å¯ç¨(é¿åº¦ä¸åºå®ç)ã

**ä¸»è¦ç¹å¾ï¼**

* Memoryåæ¶ æ¯æåå¸ï¼HASHï¼ç´¢å¼ å B+æ ç´¢å¼ ã 
* Memoryè¡¨è³å°æ¯MyISAMè¡¨è¦å¿«ä¸ä¸ªæ°éçº§ ã 
* MEMORY è¡¨çå¤§å°æ¯åå°éå¶ çãè¡¨çå¤§å°ä¸»è¦åå³äºä¸¤ä¸ªåæ°ï¼åå«æ¯ max_rows å max_heap_table_size ãå¶ä¸­ï¼max_rowså¯ä»¥å¨åå»ºè¡¨æ¶æå®ï¼max_heap_table_sizeçå¤§å°é» è®¤ä¸º16MBï¼å¯ä»¥æéè¦è¿è¡æ©å¤§ã 
* æ°æ®æä»¶ä¸ç´¢å¼æä»¶åå¼å­å¨ã 
* ç¼ºç¹ï¼å¶æ°æ®æä¸¢å¤±ï¼çå½å¨æç­ãåºäºè¿ä¸ªç¼ºé·ï¼éæ©MEMORYå­å¨å¼ææ¶éè¦ç¹å«å°å¿ã

**ä½¿ç¨Memoryå­å¨å¼æçåºæ¯ï¼**

1. ç®æ æ°æ®æ¯è¾å° ï¼èä¸éå¸¸é¢ç¹çè¿è¡è®¿é® ï¼å¨åå­ä¸­å­æ¾æ°æ®ï¼å¦æå¤ªå¤§çæ°æ®ä¼é æåå­æº¢åº ãå¯ä»¥éè¿åæ° max_heap_table_size æ§å¶Memoryè¡¨çå¤§å°ï¼éå¶Memoryè¡¨çæå¤§çå¤§å°ã 
2. å¦ææ°æ®æ¯ä¸´æ¶ç ï¼èä¸å¿é¡»ç«å³å¯ç¨å¾å°ï¼é£ä¹å°±å¯ä»¥æ¾å¨åå­ä¸­ã
3. å­å¨å¨Memoryè¡¨ä¸­çæ°æ®å¦æçªç¶é´ä¸¢å¤±çè¯ä¹æ²¡æå¤ªå¤§çå³ç³» ã

### 4.7 Federated å¼æï¼è®¿é®è¿ç¨è¡¨

**Federatedå¼ææ¯è®¿é®å¶ä»MySQLæå¡å¨çä¸ä¸ª ä»£ç ï¼å°½ç®¡è¯¥å¼æçèµ·æ¥æä¾äºä¸ç§å¾å¥½ç è·¨æå¡ å¨ççµæ´»æ§ ï¼ä½ä¹ç»å¸¸å¸¦æ¥é®é¢ï¼å æ­¤ é»è®¤æ¯ç¦ç¨ç ã**

### 4.8 Mergeå¼æï¼ç®¡çå¤ä¸ªMyISAMè¡¨ææçè¡¨éå

### 4.9 NDBå¼æï¼MySQLéç¾¤ä¸ç¨å­å¨å¼æ

ä¹å«å NDB Cluster å­å¨å¼æï¼ä¸»è¦ç¨äº MySQL Cluster åå¸å¼éç¾¤ ç¯å¢ï¼ç±»ä¼¼äº Oracle ç RAC é ç¾¤ã

### 4.10 å¼æå¯¹æ¯

MySQLä¸­åä¸ä¸ªæ°æ®åºï¼ä¸åçè¡¨å¯ä»¥éæ©ä¸åçå­å¨å¼æãå¦ä¸è¡¨å¯¹å¸¸ç¨å­å¨å¼æååºäºå¯¹æ¯ã

![image-20220616125928861](MySQLæ¶æç¯.assets/image-20220616125928861.png)

![image-20220616125945304](MySQLæ¶æç¯.assets/image-20220616125945304.png)

å¶å®è¿äºä¸è¥¿å¤§å®¶æ²¡å¿è¦ç«å³å°±ç»è®°ä½ï¼ååºæ¥çç®çå°±æ¯æ³è®©å¤§å®¶æç½ä¸åçå­å¨å¼ææ¯æä¸åçåè½ã

å¶å®æä»¬æå¸¸ç¨çå°±æ¯ InnoDB å MyISAM ï¼ææ¶ä¼æä¸ä¸ Memory ãå¶ä¸­ InnoDB æ¯ MySQL é»è®¤çå­å¨å¼æã

## 5. MyISAMåInnoDB

å¾å¤äººå¯¹ InnoDB å MyISAM çåèå­å¨çé®ï¼å°åºéæ©åªä¸ªæ¯è¾å¥½å¢ï¼ 

MySQL5.5ä¹åçé»è®¤å­å¨å¼ææ¯MyISAMï¼5.5ä¹åæ¹ä¸ºäºInnoDBã



## åèèµæ

- [å°ç¡è°·-å®çº¢åº·èå¸ï¼MySQLæ°æ®åºæç¨å¤©è±æ¿ï¼mysqlå®è£å°mysqlé«çº§ï¼å¼ºï¼ç¡¬ï¼](https://www.bilibili.com/video/BV1iq4y1u7vj)