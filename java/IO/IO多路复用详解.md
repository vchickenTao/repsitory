**<span style="font-size: 35px;">ð Java IO - IOå¤è·¯å¤ç¨è¯¦è§£</span>**

---



## IOå¤è·¯å¤ç¨è¯¦è§£

### ç°å®åºæ¯

æä»¬è¯æ³ä¸ä¸è¿æ ·çç°å®åºæ¯:

ä¸ä¸ªé¤ååæ¶æ100ä½å®¢äººå°åºï¼å½ç¶å°åºåç¬¬ä¸ä»¶è¦åçäºæå°±æ¯ç¹èãä½æ¯é®é¢æ¥äºï¼é¤åèæ¿ä¸ºäºèçº¦äººåææ¬ç®ååªæä¸ä½å¤§å æå¡åæ¿çå¯ä¸çä¸æ¬èåç­å¾å®¢äººè¿è¡æå¡ã

- é£ä¹æç¬¨(ä½æ¯æç®å)çæ¹æ³æ¯(æ¹æ³A)ï¼æ è®ºæå¤å°å®¢äººç­å¾ç¹é¤ï¼æå¡åé½æä»æçä¸ä»½èåéç»å¶ä¸­ä¸ä½å®¢äººï¼ç¶åç«å¨å®¢äººèº«æç­å¾è¿ä¸ªå®¢äººå®æç¹èè¿ç¨ãå¨è®°å½å®¢äººç¹èåå®¹åï¼æç¹èè®°å½äº¤ç»åå å¨å¸ãç¶åæ¯ç¬¬äºä½å®¢äººããããç¶åæ¯ç¬¬ä¸ä½å®¢äººãå¾ææ¾ï¼åªæèè¢è¢«é¨å¤¹è¿çèæ¿ï¼æä¼è¿æ ·è®¾ç½®æå¡æµç¨ãå ä¸ºéåç80ä½å®¢äººï¼åç­å¾è¶æ¶åå°±ä¼ç¦»åº(è¿ä¼ç»å·®è¯)ã
- äºæ¯è¿æä¸ç§åæ³(æ¹æ³B)ï¼èæ¿é©¬ä¸æ°éä½£99åæå¡åï¼åæ¶å°å¶99æ¬æ°çèåãæ¯ä¸åæå¡åææä¸æ¬èåè´è´£ä¸ä½å®¢äºº(å³é®ä¸åªå¨äºæå¡åï¼è¿å¨äºèåãå ä¸ºæ²¡æèåå®¢äººä¹æ æ³ç¹è)ãå¨å®¢äººç¹å®èåï¼è®°å½ç¹èåå®¹äº¤ç»åå å¨å¸(å½ç¶ä¸ºäºæ´é«æï¼åå å¨å¸æå¥½ä¹æ100å)ãè¿æ ·æ¯ä¸ä½å®¢äººäº«åçå°±æ¯VIPæå¡å¯ï¼å½ç¶å®¢äººä¸ä¼èµ°ï¼ä½æ¯äººåææ¬å¯æ¯ä¸ä¸ªå¤§å¤´å¦(äºæ­»ä½ )ã
- å¦å¤ä¸ç§åæ³(æ¹æ³C)ï¼å°±æ¯æ¹è¿ç¹èçæ¹å¼ï¼å½å®¢äººå°åºåï¼èªå·±ç³è¯·ä¸æ¬èåãæ³å¥½èªå·±è¦ç¹çæåï¼å°±å¼å«æå¡åãæå¡åç«å¨èªå·±èº«è¾¹åè®°å½å®¢äººçèååå®¹ãå°èåéç»å¨å¸çè¿ç¨ä¹è¦è¿è¡æ¹è¿ï¼å¹¶ä¸æ¯æ¯ä¸ä»½èåè®°å½å¥½ä»¥åï¼é½è¦äº¤ç»åå å¨å¸ãæå¡åå¯ä»¥è®°å½å·å¤ä»½èååï¼åæ¶äº¤ç»å¨å¸å°±è¡äºãé£ä¹è¿ç§æ¹å¼ï¼å¯¹äºèæ¿æ¥è¯´äººåææ¬æ¯æä½çï¼å¯¹äºå®¢äººæ¥è¯´ï¼è½ç¶ä¸åäº«åVIPæå¡å¹¶ä¸è¦è¿è¡ä¸å®çç­å¾ï¼ä½æ¯è¿äºé½æ¯å¯æ¥åçï¼å¯¹äºæå¡åæ¥è¯´ï¼åºæ¬ä¸å¥¹çæ¶é´é½æ²¡ææµªè´¹ï¼åºæ¬ä¸è¢«èæ¿åæäºæåä¸æ»´æ²¹æ°´ã

å¦ææ¨æ¯èæ¿ï¼æ¨ä¼éç¨åªç§æ¹å¼å¢?

å°åºæåµ: å¹¶åéãå°åºæåµä¸çæ³æ¶ï¼ä¸ä¸ªæå¡åä¸æ¬èåï¼å½ç¶æ¯è¶³å¤äºãæä»¥ä¸åçèæ¿å¨ä¸åçåºåä¸ï¼å°ä¼çµæ´»éæ©æå¡ååèåçéç½®ã

- å®¢äºº: å®¢æ·ç«¯è¯·æ±
- ç¹é¤åå®¹: å®¢æ·ç«¯åéçå®éæ°æ®
- èæ¿: æä½ç³»ç»
- äººåææ¬: ç³»ç»èµæº
- èå: æä»¶ç¶ææè¿°ç¬¦ãæä½ç³»ç»å¯¹äºä¸ä¸ªè¿ç¨è½å¤åæ¶ææçæä»¶ç¶ææè¿°ç¬¦çä¸ªæ°æ¯æéå¶çï¼å¨linuxç³»ç»ä¸­$ulimit -næ¥çè¿ä¸ªéå¶å¼ï¼å½ç¶ä¹æ¯å¯ä»¥(å¹¶ä¸åºè¯¥)è¿è¡åæ ¸åæ°è°æ´çã
- æå¡å: æä½ç³»ç»åæ ¸ç¨äºIOæä½ççº¿ç¨(åæ ¸çº¿ç¨)
- å¨å¸: åºç¨ç¨åºçº¿ç¨(å½ç¶å¨æ¿å°±æ¯åºç¨ç¨åºè¿ç¨å¯)
- é¤åä¼ éæ¹å¼: åæ¬äºé»å¡å¼åéé»å¡å¼ä¸¤ç§ã
  - æ¹æ³A: é»å¡å¼/éé»å¡å¼ åæ­¥IO
  - æ¹æ³B: ä½¿ç¨çº¿ç¨è¿è¡å¤çç é»å¡å¼/éé»å¡å¼ åæ­¥IO
  - æ¹æ³C: é»å¡å¼/éé»å¡å¼ å¤è·¯å¤ç¨IO

### å¸åçå¤è·¯å¤ç¨IOå®ç°

ç®åæµç¨çå¤è·¯å¤ç¨IOå®ç°ä¸»è¦åæ¬åç§: `select`ã`poll`ã`epoll`ã`kqueue`ãä¸è¡¨æ¯ä»ä»¬çä¸äºéè¦ç¹æ§çæ¯è¾:

| IOæ¨¡å | ç¸å¯¹æ§è½ | å³é®æè·¯         | æä½ç³»ç»      | JAVAæ¯ææåµ                                                 |
| ------ | -------- | ---------------- | ------------- | ------------------------------------------------------------ |
| select | è¾é«     | Reactor          | windows/Linux | æ¯æ,Reactoræ¨¡å¼(ååºå¨è®¾è®¡æ¨¡å¼)ãLinuxæä½ç³»ç»ç kernels 2.4åæ ¸çæ¬ä¹åï¼é»è®¤ä½¿ç¨selectï¼èç®åwindowsä¸å¯¹åæ­¥IOçæ¯æï¼é½æ¯selectæ¨¡å |
| poll   | è¾é«     | Reactor          | Linux         | Linuxä¸çJAVA NIOæ¡æ¶ï¼Linux kernels 2.6åæ ¸çæ¬ä¹åä½¿ç¨pollè¿è¡æ¯æãä¹æ¯ä½¿ç¨çReactoræ¨¡å¼ |
| epoll  | é«       | Reactor/Proactor | Linux         | Linux kernels 2.6åæ ¸çæ¬åä»¥åä½¿ç¨epollè¿è¡æ¯æï¼Linux kernels 2.6åæ ¸çæ¬ä¹åä½¿ç¨pollè¿è¡æ¯æï¼å¦å¤ä¸å®æ³¨æï¼ç±äºLinuxä¸æ²¡æWindowsä¸çIOCPææ¯æä¾çæ­£ç å¼æ­¥IO æ¯æï¼æä»¥Linuxä¸ä½¿ç¨epollæ¨¡æå¼æ­¥IO |
| kqueue | é«       | Proactor         | Linux         | ç®åJAVAççæ¬ä¸æ¯æ                                         |

å¤è·¯å¤ç¨IOææ¯æéç¨çæ¯âé«å¹¶åâåºæ¯ï¼æè°é«å¹¶åæ¯æ1æ¯«ç§åè³å°åæ¶æä¸åä¸ªè¿æ¥è¯·æ±åå¤å¥½ãå¶ä»æåµä¸å¤è·¯å¤ç¨IOææ¯åæ¥ä¸åºæ¥å®çä¼å¿ãå¦ä¸æ¹é¢ï¼ä½¿ç¨JAVA NIOè¿è¡åè½å®ç°ï¼ç¸å¯¹äºä¼ ç»çSocketå¥æ¥å­å®ç°è¦å¤æä¸äºï¼æä»¥å®éåºç¨ä¸­ï¼éè¦æ ¹æ®èªå·±çä¸å¡éæ±è¿è¡ææ¯éæ©ã

### Reactoræ¨¡ååProactoræ¨¡å

#### ä¼ ç»IOæ¨¡å

å¯¹äºä¼ ç»IOæ¨¡åï¼å¶ä¸»è¦æ¯ä¸ä¸ªServerå¯¹æ¥Nä¸ªå®¢æ·ç«¯ï¼å¨å®¢æ·ç«¯è¿æ¥ä¹åï¼ä¸ºæ¯ä¸ªå®¢æ·ç«¯é½åéä¸ä¸ªæ§è¡çº¿ç¨ãå¦ä¸å¾æ¯è¯¥æ¨¡åçä¸ä¸ªæ¼ç¤ºï¼

![img](https://pdai.tech/_images/io/java-io-reactor-1.png)

ä»å¾ä¸­å¯ä»¥çåºï¼ä¼ ç»IOçç¹ç¹å¨äºï¼

- æ¯ä¸ªå®¢æ·ç«¯è¿æ¥å°è¾¾ä¹åï¼æå¡ç«¯ä¼åéä¸ä¸ªçº¿ç¨ç»è¯¥å®¢æ·ç«¯ï¼è¯¥çº¿ç¨ä¼å¤çåæ¬è¯»åæ°æ®ï¼è§£ç ï¼ä¸å¡è®¡ç®ï¼ç¼ç ï¼ä»¥ååéæ°æ®æ´ä¸ªè¿ç¨ï¼
- åä¸æ¶å»ï¼æå¡ç«¯çååéä¸æå¡å¨ææä¾ççº¿ç¨æ°éæ¯åçº¿æ§å³ç³»çã

è¿ç§è®¾è®¡æ¨¡å¼å¨å®¢æ·ç«¯è¿æ¥ä¸å¤ï¼å¹¶åéä¸å¤§çæåµä¸æ¯å¯ä»¥è¿è¡å¾å¾å¥½çï¼ä½æ¯å¨æµ·éå¹¶åçæåµä¸ï¼è¿ç§æ¨¡å¼å°±æ¾å¾åä¸ä»å¿äºãè¿ç§æ¨¡å¼ä¸»è¦å­å¨çé®é¢æå¦ä¸å ç¹ï¼

- æå¡å¨çå¹¶åéå¯¹æå¡ç«¯è½å¤åå»ºççº¿ç¨æ°æå¾å¤§çä¾èµå³ç³»ï¼ä½æ¯æå¡å¨çº¿ç¨å´æ¯ä¸è½æ éå¢é¿çï¼
- æå¡ç«¯æ¯ä¸ªçº¿ç¨ä¸ä»è¦è¿è¡IOè¯»åæä½ï¼èä¸è¿éè¦è¿è¡ä¸å¡è®¡ç®ï¼
- æå¡ç«¯å¨è·åå®¢æ·ç«¯è¿æ¥ï¼è¯»åæ°æ®ï¼ä»¥ååå¥æ°æ®çè¿ç¨é½æ¯é»å¡ç±»åçï¼å¨ç½ç»ç¶åµä¸å¥½çæåµä¸ï¼è¿å°æå¤§çéä½æå¡å¨æ¯ä¸ªçº¿ç¨çå©ç¨çï¼ä»èéä½æå¡å¨ååéã

#### Reactoräºä»¶é©±å¨æ¨¡å

å¨ä¼ ç»IOæ¨¡åä¸­ï¼ç±äºçº¿ç¨å¨ç­å¾è¿æ¥ä»¥åè¿è¡IOæä½æ¶é½ä¼é»å¡å½åçº¿ç¨ï¼è¿é¨åæèæ¯éå¸¸å¤§çãå èjdk 1.4ä¸­å°±æä¾äºä¸å¥éé»å¡IOçAPIãè¯¥APIæ¬è´¨ä¸æ¯ä»¥äºä»¶é©±å¨æ¥å¤çç½ç»äºä»¶çï¼èReactoræ¯åºäºè¯¥APIæåºçä¸å¥IOæ¨¡åãå¦ä¸æ¯Reactoräºä»¶é©±å¨æ¨¡åçç¤ºæå¾ï¼

![img](https://pdai.tech/_images/io/java-io-reactor-2.png)

ä»å¾ä¸­å¯ä»¥çåºï¼å¨Reactoræ¨¡åä¸­ï¼ä¸»è¦æåä¸ªè§è²ï¼å®¢æ·ç«¯è¿æ¥ï¼Reactorï¼AcceptoråHandlerãè¿éAcceptorä¼ä¸æ­å°æ¥æ¶å®¢æ·ç«¯çè¿æ¥ï¼ç¶åå°æ¥æ¶å°çè¿æ¥äº¤ç±Reactorè¿è¡ååï¼æåæå·ä½çHandlerè¿è¡å¤çãæ¹è¿åçReactoræ¨¡åç¸å¯¹äºä¼ ç»çIOæ¨¡åä¸»è¦æå¦ä¸ä¼ç¹ï¼

- ä»æ¨¡åä¸æ¥è®²ï¼å¦æä»ä»è¿æ¯åªä½¿ç¨ä¸ä¸ªçº¿ç¨æ± æ¥å¤çå®¢æ·ç«¯è¿æ¥çç½ç»è¯»åï¼ä»¥åä¸å¡è®¡ç®ï¼é£ä¹Reactoræ¨¡åä¸ä¼ ç»IOæ¨¡åå¨æçä¸å¹¶æ²¡æä»ä¹æåãä½æ¯Reactoræ¨¡åæ¯ä»¥äºä»¶è¿è¡é©±å¨çï¼å¶è½å¤å°æ¥æ¶å®¢æ·ç«¯è¿æ¥ï¼+ ç½ç»è¯»åç½ç»åï¼ä»¥åä¸å¡è®¡ç®è¿è¡æåï¼ä»èæå¤§çæåå¤çæçï¼
- Reactoræ¨¡åæ¯å¼æ­¥éé»å¡æ¨¡åï¼å·¥ä½çº¿ç¨å¨æ²¡æç½ç»äºä»¶æ¶å¯ä»¥å¤çå¶ä»çä»»å¡ï¼èä¸ç¨åä¼ ç»IOé£æ ·å¿é¡»é»å¡ç­å¾ã

#### Reactoræ¨¡å----ä¸å¡å¤çä¸IOåç¦»

å¨ä¸é¢çReactoræ¨¡åä¸­ï¼ç±äºç½ç»è¯»ååä¸å¡æä½é½å¨åä¸ä¸ªçº¿ç¨ä¸­ï¼å¨é«å¹¶åæåµä¸ï¼è¿éçç³»ç»ç¶é¢ä¸»è¦å¨ä¸¤æ¹é¢ï¼

- é«é¢ççç½ç»è¯»åäºä»¶å¤çï¼
- å¤§éçä¸å¡æä½å¤çï¼

åºäºä¸è¿°ä¸¤ä¸ªé®é¢ï¼è¿éå¨åçº¿ç¨Reactoræ¨¡åçåºç¡ä¸æåºäºä½¿ç¨çº¿ç¨æ± çæ¹å¼å¤çä¸å¡æä½çæ¨¡åãå¦ä¸æ¯è¯¥æ¨¡åçç¤ºæå¾ï¼

![img](https://pdai.tech/_images/io/java-io-reactor-3.png)

ä»å¾ä¸­å¯ä»¥çåºï¼å¨å¤çº¿ç¨è¿è¡ä¸å¡æä½çæ¨¡åä¸ï¼è¯¥æ¨¡å¼ä¸»è¦å·æå¦ä¸ç¹ç¹ï¼

- ä½¿ç¨ä¸ä¸ªçº¿ç¨è¿è¡å®¢æ·ç«¯è¿æ¥çæ¥æ¶ä»¥åç½ç»è¯»åäºä»¶çå¤çï¼
- å¨æ¥æ¶å°å®¢æ·ç«¯è¿æ¥ä¹åï¼å°è¯¥è¿æ¥äº¤ç±çº¿ç¨æ± è¿è¡æ°æ®çç¼è§£ç ä»¥åä¸å¡è®¡ç®ã

è¿ç§æ¨¡å¼ç¸è¾äºåé¢çæ¨¡å¼æ§è½æäºå¾å¤§çæåï¼ä¸»è¦å¨äºå¨è¿è¡ç½ç»è¯»åçåæ¶ï¼ä¹è¿è¡äºä¸å¡è®¡ç®ï¼ä»èå¤§å¤§æåäºç³»ç»çååéãä½æ¯è¿ç§æ¨¡å¼ä¹æå¶ä¸è¶³ï¼ä¸»è¦å¨äºï¼

- ç½ç»è¯»åæ¯ä¸ä¸ªæ¯è¾æ¶èCPUçæä½ï¼å¨é«å¹¶åçæåµä¸ï¼å°ä¼æå¤§éçå®¢æ·ç«¯æ°æ®éè¦è¿è¡ç½ç»è¯»åï¼æ­¤æ¶ä¸ä¸ªçº¿ç¨å°ä¸è¶³ä»¥å¤çè¿ä¹å¤è¯·æ±ã

#### Reactoræ¨¡å----å¹¶åè¯»å

å¯¹äºä½¿ç¨çº¿ç¨æ± å¤çä¸å¡æä½çæ¨¡åï¼ç±äºç½ç»è¯»åå¨é«å¹¶åæåµä¸ä¼æä¸ºç³»ç»çä¸ä¸ªç¶é¢ï¼å èéå¯¹è¯¥æ¨¡åè¿éæåºäºä¸ç§æ¹è¿åçæ¨¡åï¼å³ä½¿ç¨çº¿ç¨æ± è¿è¡ç½ç»è¯»åï¼èä»ä»åªä½¿ç¨ä¸ä¸ªçº¿ç¨ä¸é¨æ¥æ¶å®¢æ·ç«¯è¿æ¥ãå¦ä¸æ¯è¯¥æ¨¡åçç¤ºæå¾ï¼

![img](https://pdai.tech/_images/io/java-io-reactor-4.png)

å¯ä»¥çå°ï¼æ¹è¿åçReactoræ¨¡åå°Reactoræåä¸ºäºmainReactoråsubReactorãè¿émainReactorä¸»è¦è¿è¡å®¢æ·ç«¯è¿æ¥çå¤çï¼å¤çå®æä¹åå°è¯¥è¿æ¥äº¤ç±subReactorä»¥å¤çå®¢æ·ç«¯çç½ç»è¯»åãè¿éçsubReactoråæ¯ä½¿ç¨ä¸ä¸ªçº¿ç¨æ± æ¥æ¯æçï¼å¶è¯»åè½åå°ä¼éççº¿ç¨æ°çå¢å¤èå¤§å¤§å¢å ãå¯¹äºä¸å¡æä½ï¼è¿éä¹æ¯ä½¿ç¨ä¸ä¸ªçº¿ç¨æ± ï¼èæ¯ä¸ªä¸å¡è¯·æ±é½åªéè¦è¿è¡ç¼è§£ç åä¸å¡è®¡ç®ãéè¿è¿ç§æ¹å¼ï¼æå¡å¨çæ§è½å°ä¼å¤§å¤§æåï¼å¨å¯è§æåµä¸ï¼å¶åºæ¬ä¸å¯ä»¥æ¯æç¾ä¸è¿æ¥ã

#### Reactoræ¨¡åç¤ºä¾

å¯¹äºä¸è¿°Reactoræ¨¡åï¼æå¡ç«¯ä¸»è¦æä¸ä¸ªè§è²ï¼Reactorï¼AcceptoråHandlerãè¿éåºäºDoug Leaçææ¡£å¯¹å¶è¿è¡äºå®ç°ï¼å¦ä¸æ¯Reactorçå®ç°ä»£ç ï¼

```java
public class Reactor implements Runnable {
  private final Selector selector;
  private final ServerSocketChannel serverSocket;

  public Reactor(int port) throws IOException {
    serverSocket = ServerSocketChannel.open();  // åå»ºæå¡ç«¯çServerSocketChannel
    serverSocket.configureBlocking(false);  // è®¾ç½®ä¸ºéé»å¡æ¨¡å¼
    selector = Selector.open();  // åå»ºä¸ä¸ªSelectorå¤è·¯å¤ç¨å¨
    SelectionKey key = serverSocket.register(selector, SelectionKey.OP_ACCEPT);
    serverSocket.bind(new InetSocketAddress(port));  // ç»å®æå¡ç«¯ç«¯å£
    key.attach(new Acceptor(serverSocket));  // ä¸ºæå¡ç«¯Channelç»å®ä¸ä¸ªAcceptor
  }

  @Override
  public void run() {
    try {
      while (!Thread.interrupted()) {
        selector.select();  // æå¡ç«¯ä½¿ç¨ä¸ä¸ªçº¿ç¨ä¸æ­ç­å¾å®¢æ·ç«¯çè¿æ¥å°è¾¾
        Set<SelectionKey> keys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = keys.iterator();
        while (iterator.hasNext()) {
          dispatch(iterator.next());  // çå¬å°å®¢æ·ç«¯è¿æ¥äºä»¶åå°å¶ååç»Acceptor
          iterator.remove();
        }

        selector.selectNow();
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

  private void dispatch(SelectionKey key) throws IOException {
    // è¿éçattachementä¹å³åé¢ä¸ºæå¡ç«¯Channelç»å®çAcceptorï¼è°ç¨å¶run()æ¹æ³è¿è¡
    // å®¢æ·ç«¯è¿æ¥çè·åï¼å¹¶ä¸è¿è¡åå
    Runnable attachment = (Runnable) key.attachment();
    attachment.run();
  }
}
```

è¿éReactoré¦åå¼å¯äºä¸ä¸ªServerSocketChannelï¼ç¶åå°å¶ç»å®å°æå®çç«¯å£ï¼å¹¶ä¸æ³¨åå°äºä¸ä¸ªå¤è·¯å¤ç¨å¨ä¸ãæ¥çå¨ä¸ä¸ªçº¿ç¨ä¸­ï¼å¶ä¼å¨å¤è·¯å¤ç¨å¨ä¸ç­å¾å®¢æ·ç«¯è¿æ¥ãå½æå®¢æ·ç«¯è¿æ¥å°è¾¾åï¼Reactorå°±ä¼å°å¶æ´¾åç»ä¸ä¸ªAcceptorï¼ç±è¯¥Acceptorä¸é¨è¿è¡å®¢æ·ç«¯è¿æ¥çè·åãä¸é¢æä»¬ç»§ç»­çä¸ä¸Acceptorçä»£ç ï¼

```java
public class Acceptor implements Runnable {
  private final ExecutorService executor = Executors.newFixedThreadPool(20);

  private final ServerSocketChannel serverSocket;

  public Acceptor(ServerSocketChannel serverSocket) {
    this.serverSocket = serverSocket;
  }

  @Override
  public void run() {
    try {
      SocketChannel channel = serverSocket.accept();  // è·åå®¢æ·ç«¯è¿æ¥
      if (null != channel) {
        executor.execute(new Handler(channel));  // å°å®¢æ·ç«¯è¿æ¥äº¤ç±çº¿ç¨æ± å¤ç
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

è¿éå¯ä»¥çå°ï¼å¨Acceptorè·åå°å®¢æ·ç«¯è¿æ¥ä¹åï¼å¶å°±å°å¶äº¤ç±çº¿ç¨æ± è¿è¡ç½ç»è¯»åäºï¼èè¿éçä¸»çº¿ç¨åªæ¯ä¸æ­çå¬å®¢æ·ç«¯è¿æ¥äºä»¶ãä¸é¢æä»¬ççHandlerçå·ä½é»è¾ï¼

```java
public class Handler implements Runnable {
  private volatile static Selector selector;
  private final SocketChannel channel;
  private SelectionKey key;
  private volatile ByteBuffer input = ByteBuffer.allocate(1024);
  private volatile ByteBuffer output = ByteBuffer.allocate(1024);

  public Handler(SocketChannel channel) throws IOException {
    this.channel = channel;
    channel.configureBlocking(false);  // è®¾ç½®å®¢æ·ç«¯è¿æ¥ä¸ºéé»å¡æ¨¡å¼
    selector = Selector.open();  // ä¸ºå®¢æ·ç«¯åå»ºä¸ä¸ªæ°çå¤è·¯å¤ç¨å¨
    key = channel.register(selector, SelectionKey.OP_READ);  // æ³¨åå®¢æ·ç«¯Channelçè¯»äºä»¶
  }

  @Override
  public void run() {
    try {
      while (selector.isOpen() && channel.isOpen()) {
        Set<SelectionKey> keys = select();  // ç­å¾å®¢æ·ç«¯äºä»¶åç
        Iterator<SelectionKey> iterator = keys.iterator();
        while (iterator.hasNext()) {
          SelectionKey key = iterator.next();
          iterator.remove();

          // å¦æå½åæ¯è¯»äºä»¶ï¼åè¯»åæ°æ®
          if (key.isReadable()) {
            read(key);
          } else if (key.isWritable()) {
           // å¦æå½åæ¯åäºä»¶ï¼ååå¥æ°æ®
            write(key);
          }
        }
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  // è¿éå¤ççä¸»è¦ç®çæ¯å¤çJdkçä¸ä¸ªbugï¼è¯¥bugä¼å¯¼è´Selectorè¢«æå¤è§¦åï¼ä½æ¯å®éä¸æ²¡æä»»ä½äºä»¶å°è¾¾ï¼
  // æ­¤æ¶çå¤çæ¹å¼æ¯æ°å»ºä¸ä¸ªSelectorï¼ç¶åéæ°å°å½åChannelæ³¨åå°è¯¥Selectorä¸
  private Set<SelectionKey> select() throws IOException {
    selector.select();
    Set<SelectionKey> keys = selector.selectedKeys();
    if (keys.isEmpty()) {
      int interestOps = key.interestOps();
      selector = Selector.open();
      key = channel.register(selector, interestOps);
      return select();
    }

    return keys;
  }

  // è¯»åå®¢æ·ç«¯åéçæ°æ®
  private void read(SelectionKey key) throws IOException {
    channel.read(input);
    if (input.position() == 0) {
      return;
    }

    input.flip();
    process();  // å¯¹è¯»åçæ°æ®è¿è¡ä¸å¡å¤ç
    input.clear();
    key.interestOps(SelectionKey.OP_WRITE);  // è¯»åå®æåçå¬åå¥äºä»¶
  }

  private void write(SelectionKey key) throws IOException {
    output.flip();
    if (channel.isOpen()) {
      channel.write(output);  // å½æåå¥äºä»¶æ¶ï¼å°ä¸å¡å¤ççç»æåå¥å°å®¢æ·ç«¯Channelä¸­
      key.channel();
      channel.close();
      output.clear();
    }
  }
    
  // è¿è¡ä¸å¡å¤çï¼å¹¶ä¸è·åå¤çç»æãæ¬è´¨ä¸ï¼åºäºReactoræ¨¡åï¼å¦æè¿éæä¸ºå¤çç¶é¢ï¼
  // åç´æ¥å°å¶å¤çè¿ç¨æ¾å¥çº¿ç¨æ± å³å¯ï¼å¹¶ä¸ä½¿ç¨ä¸ä¸ªFutureè·åå¤çç»æï¼æååå¥å®¢æ·ç«¯Channel
  private void process() {
    byte[] bytes = new byte[input.remaining()];
    input.get(bytes);
    String message = new String(bytes, CharsetUtil.UTF_8);
    System.out.println("receive message from client: \n" + message);

    output.put("hello client".getBytes());
  }
}
```

å¨Handlerä¸­ï¼ä¸»è¦è¿è¡çå°±æ¯ä¸ºæ¯ä¸ä¸ªå®¢æ·ç«¯Channelåå»ºä¸ä¸ªSelectorï¼å¹¶ä¸çå¬è¯¥Channelçç½ç»è¯»åäºä»¶ãå½æäºä»¶å°è¾¾æ¶ï¼è¿è¡æ°æ®çè¯»åï¼èä¸å¡æä½è¿äº¤ç±å·ä½çä¸å¡çº¿ç¨æ± å¤çã

### JAVAå¯¹å¤è·¯å¤ç¨IOçæ¯æ

![img](https://pdai.tech/_images/io/java-io-nio-1.png)

#### éè¦æ¦å¿µ: Channel

ééï¼è¢«å»ºç«çä¸ä¸ªåºç¨ç¨åºåæä½ç³»ç»äº¤äºäºä»¶ãä¼ éåå®¹çæ¸ é(æ³¨ææ¯è¿æ¥å°æä½ç³»ç»)ãä¸ä¸ªééä¼æä¸ä¸ªä¸å±çæä»¶ç¶ææè¿°ç¬¦ãé£ä¹æ¢ç¶æ¯åæä½ç³»ç»è¿è¡åå®¹çä¼ éï¼é£ä¹è¯´æåºç¨ç¨åºå¯ä»¥éè¿ééè¯»åæ°æ®ï¼ä¹å¯ä»¥éè¿ééåæä½ç³»ç»åæ°æ®ã

JDK APIä¸­çChannelçæè¿°æ¯:

> A channel represents an open connection to an entity such as a hardware device, a file, a network socket, or a program component that is capable of performing one or more distinct I/O operations, for example reading or writing.

> A channel is either open or closed. A channel is open upon creation, and once closed it remains closed. Once a channel is closed, any attempt to invoke an I/O operation upon it will cause a ClosedChannelException to be thrown. Whether or not a channel is open may be tested by invoking its isOpen method.

JAVA NIO æ¡æ¶ä¸­ï¼èªæçChannelééåæ¬:

![img](https://pdai.tech/_images/io/java-io-nio-2.png)

ææè¢«Selector(éæ©å¨)æ³¨åçééï¼åªè½æ¯ç»§æ¿äºSelectableChannelç±»çå­ç±»ãå¦ä¸å¾æç¤º

- ServerSocketChannel: åºç¨æå¡å¨ç¨åºççå¬ééãåªæéè¿è¿ä¸ªééï¼åºç¨ç¨åºæè½åæä½ç³»ç»æ³¨åæ¯æâå¤è·¯å¤ç¨IOâçç«¯å£çå¬ãåæ¶æ¯æUDPåè®®åTCPåè®®ã
- ScoketChannel: TCP Socketå¥æ¥å­ççå¬ééï¼ä¸ä¸ªSocketå¥æ¥å­å¯¹åºäºä¸ä¸ªå®¢æ·ç«¯IP: ç«¯å£ å° æå¡å¨IP: ç«¯å£çéä¿¡è¿æ¥ã
- DatagramChannel: UDP æ°æ®æ¥æççå¬ééã

#### éè¦æ¦å¿µ: Buffer

æ°æ®ç¼å­åº: å¨JAVA NIO æ¡æ¶ä¸­ï¼ä¸ºäºä¿è¯æ¯ä¸ªééçæ°æ®è¯»åéåº¦JAVA NIO æ¡æ¶ä¸ºæ¯ä¸ç§éè¦æ¯ææ°æ®è¯»åçéééæäºBufferçæ¯æã

è¿å¥è¯æä¹çè§£å¢? ä¾å¦ServerSocketChannelééå®åªæ¯æå¯¹OP_ACCEPTäºä»¶ççå¬ï¼æä»¥å®æ¯ä¸è½ç´æ¥è¿è¡ç½ç»æ°æ®åå®¹çè¯»åçãæä»¥ServerSocketChannelæ¯æ²¡æéæBufferçã

Bufferæä¸¤ç§å·¥ä½æ¨¡å¼: åæ¨¡å¼åè¯»æ¨¡å¼ãå¨è¯»æ¨¡å¼ä¸ï¼åºç¨ç¨åºåªè½ä»Bufferä¸­è¯»åæ°æ®ï¼ä¸è½è¿è¡åæä½ãä½æ¯å¨åæ¨¡å¼ä¸ï¼åºç¨ç¨åºæ¯å¯ä»¥è¿è¡è¯»æä½çï¼è¿å°±è¡¨ç¤ºå¯è½ä¼åºç°èè¯»çæåµãæä»¥ä¸æ¦æ¨å³å®è¦ä»Bufferä¸­è¯»åæ°æ®ï¼ä¸å®è¦å°Bufferçç¶ææ¹ä¸ºè¯»æ¨¡å¼ã

å¦ä¸å¾:

![img](https://pdai.tech/_images/io/java-io-nio-3.png)

- position: ç¼å­åºç®åè¿å¨æä½çæ°æ®åä½ç½®
- limit: ç¼å­åºæå¤§å¯ä»¥è¿è¡æä½çä½ç½®ãç¼å­åºçè¯»åç¶ææ­£å¼ç±è¿ä¸ªå±æ§æ§å¶çã
- capacity: ç¼å­åºçæå¤§å®¹éãè¿ä¸ªå®¹éæ¯å¨ç¼å­åºåå»ºæ¶è¿è¡æå®çãç±äºé«å¹¶åæ¶ééæ°éå¾å¾ä¼å¾åºå¤§ï¼æä»¥æ¯ä¸ä¸ªç¼å­åºçå®¹éæå¥½ä¸è¦è¿å¤§ã

å¨ä¸æJAVA NIOæ¡æ¶çä»£ç å®ä¾ä¸­ï¼æä»¬å°è¿è¡Bufferç¼å­åºæä½çæ¼ç¤ºã

#### éè¦æ¦å¿µ: Selector

Selectorçè±æå«ä¹æ¯âéæ©å¨âï¼ä¸è¿æ ¹æ®æä»¬è¯¦ç»ä»ç»çSelectorçå²ä½èè´£ï¼æ¨å¯ä»¥æå®ç§°ä¹ä¸ºâè½®è¯¢ä»£çå¨âãâäºä»¶è®¢éå¨âãâchannelå®¹å¨ç®¡çæºâé½è¡ã

- äºä»¶è®¢éåChannelç®¡ç

åºç¨ç¨åºå°åSelectorå¯¹è±¡æ³¨åéè¦å®å³æ³¨çChannelï¼ä»¥åå·ä½çæä¸ä¸ªChannelä¼å¯¹åªäºIOäºä»¶æå´è¶£ãSelectorä¸­ä¹ä¼ç»´æ¤ä¸ä¸ªâå·²ç»æ³¨åçChannelâçå®¹å¨ãä»¥ä¸ä»£ç æ¥èªWindowsSelectorImplå®ç°ç±»ä¸­ï¼å¯¹å·²ç»æ³¨åçChannelçç®¡çå®¹å¨:

```java
// Initial capacity of the poll array
private final int INIT_CAP = 8;
// Maximum number of sockets for select().
// Should be INIT_CAP times a power of 2
private final static int MAX_SELECTABLE_FDS = 1024;

// The list of SelectableChannels serviced by this Selector. Every mod
// MAX_SELECTABLE_FDS entry is bogus, to align this array with the poll
// array,  where the corresponding entry is occupied by the wakeupSocket
private SelectionKeyImpl[] channelArray = new SelectionKeyImpl[INIT_CAP];
```

- è½®è¯¢ä»£ç

åºç¨å±ä¸åéè¿é»å¡æ¨¡å¼æèéé»å¡æ¨¡å¼ç´æ¥è¯¢é®æä½ç³»ç»âäºä»¶ææ²¡æåçâï¼èæ¯ç±Selectorä»£å¶è¯¢é®ã

- å®ç°ä¸åæä½ç³»ç»çæ¯æ

ä¹åå·²ç»æå°è¿ï¼å¤è·¯å¤ç¨IOææ¯ æ¯éè¦æä½ç³»ç»è¿è¡æ¯æçï¼å¶ç¹ç¹å°±æ¯æä½ç³»ç»å¯ä»¥åæ¶æ«æåä¸ä¸ªç«¯å£ä¸ä¸åç½ç»è¿æ¥çäºä»¶ãæä»¥ä½ä¸ºä¸å±çJVMï¼å¿é¡»è¦ä¸º ä¸åæä½ç³»ç»çå¤è·¯å¤ç¨IOå®ç° ç¼åä¸åçä»£ç ãåæ ·æä½¿ç¨çæµè¯ç¯å¢æ¯Windowsï¼å®å¯¹åºçå®ç°ç±»æ¯sun.nio.ch.WindowsSelectorImpl:

![img](https://pdai.tech/_images/io/java-io-nio-4.png)

#### JAVA NIO æ¡æ¶ç®è¦è®¾è®¡åæ

éè¿ä¸æçæè¿°ï¼æä»¬ç¥éäºå¤è·¯å¤ç¨IOææ¯æ¯æä½ç³»ç»çåæ ¸å®ç°ãå¨ä¸åçæä½ç³»ç»ï¼çè³åä¸ç³»åæä½ç³»ç»ççæ¬ä¸­æå®ç°çå¤è·¯å¤ç¨IOææ¯é½æ¯ä¸ä¸æ ·çãé£ä¹ä½ä¸ºè·¨å¹³å°çJAVA JVMæ¥è¯´å¦ä½éåºå¤ç§å¤æ ·çå¤è·¯å¤ç¨IOææ¯å®ç°å¢? é¢åå¯¹è±¡çå¨åå°±æ¾ç°åºæ¥äº: æ è®ºä½¿ç¨åªç§å®ç°æ¹å¼ï¼ä»ä»¬é½ä¼æâéæ©å¨âãâééâãâç¼å­âè¿å ä¸ªæä½è¦ç´ ï¼é£ä¹å¯ä»¥ä¸ºä¸åçå¤è·¯å¤ç¨IOææ¯åå»ºä¸ä¸ªç»ä¸çæ½è±¡ç»ï¼å¹¶ä¸ä¸ºä¸åçæä½ç³»ç»è¿è¡å·ä½çå®ç°ãJAVA NIOä¸­å¯¹åç§å¤è·¯å¤ç¨IOçæ¯æï¼ä¸»è¦çåºç¡æ¯java.nio.channels.spi.SelectorProvideræ½è±¡ç±»ï¼å¶ä¸­çå ä¸ªä¸»è¦æ½è±¡æ¹æ³åæ¬:

- public abstract DatagramChannel openDatagramChannel(): åå»ºåè¿ä¸ªæä½ç³»ç»å¹éçUDP ééå®ç°ã
- public abstract AbstractSelector openSelector(): åå»ºåè¿ä¸ªæä½ç³»ç»å¹éçNIOéæ©å¨ï¼å°±åä¸ææè¿°ï¼ä¸åçæä½ç³»ç»ï¼ä¸åççæ¬æé»è®¤æ¯æçNIOæ¨¡åæ¯ä¸ä¸æ ·çã
- public abstract ServerSocketChannel openServerSocketChannel(): åå»ºåè¿ä¸ªNIOæ¨¡åå¹éçæå¡å¨ç«¯ééã
- public abstract SocketChannel openSocketChannel(): åå»ºåè¿ä¸ªNIOæ¨¡åå¹éçTCP Socketå¥æ¥å­éé(ç¨æ¥åæ å®¢æ·ç«¯çTCPè¿æ¥)

ç±äºJAVA NIOæ¡æ¶çæ´ä¸ªè®¾è®¡æ¯å¾å¤§çï¼æä»¥æä»¬åªè½è¿åä¸é¨åæä»¬å³å¿çé®é¢ãè¿éæä»¬ä»¥JAVA NIOæ¡æ¶ä¸­å¯¹äºä¸åå¤è·¯å¤ç¨IOææ¯çéæ©å¨ è¿è¡å®ä¾ååå»ºçæ¹å¼ä½ä¸ºä¾å­ï¼ä»¥ç¹çª¥è±¹è§å¨å±:

![img](https://pdai.tech/_images/io/java-io-nio-5.png)

å¾ææ¾ï¼ä¸åçSelectorProviderå®ç°å¯¹åºäºä¸åç éæ©å¨ãç±å·ä½çSelectorProviderå®ç°è¿è¡åå»ºãå¦å¤è¯´æä¸ä¸ï¼å®éä¸nettyåºå±ä¹æ¯éè¿è¿ä¸ªè®¾è®¡è·å¾å·ä½ä½¿ç¨çNIOæ¨¡åï¼æä»¬åæè®²è§£Nettyæ¶ï¼ä¼è®²å°è¿ä¸ªé®é¢ãä»¥ä¸ä»£ç æ¯Netty 4.0ä¸­NioServerSocketChannelè¿è¡å®ä¾åæ¶çæ ¸å¿ä»£ç çæ®µ:

```java
private static ServerSocketChannel newSocket(SelectorProvider provider) {
    try {
        /**
            *  Use the {@link SelectorProvider} to open {@link SocketChannel} and so remove condition in
            *  {@link SelectorProvider#provider()} which is called by each ServerSocketChannel.open() otherwise.
            *
            *  See <a href="See https://github.com/netty/netty/issues/2308">#2308</a>.
            */
        return provider.openServerSocketChannel();
    } catch (IOException e) {
        throw new ChannelException(
                "Failed to open a server socket.", e);
    }
}
```

#### JAVAå®ä¾

ä¸é¢ï¼æä»¬ä½¿ç¨JAVA NIOæ¡æ¶ï¼å®ç°ä¸ä¸ªæ¯æå¤è·¯å¤ç¨IOçæå¡å¨ç«¯(å®éä¸å®¢æ·ç«¯æ¯å¦ä½¿ç¨å¤è·¯å¤ç¨IOææ¯ï¼å¯¹æ´ä¸ªç³»ç»æ¶æçæ§è½æåç¸å³æ§ä¸å¤§):

```java
package testNSocket;

import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.nio.ByteBuffer;
import java.nio.channels.SelectableChannel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.log4j.BasicConfigurator;

public class SocketServer1 {

    static {
        BasicConfigurator.configure();
    }

    /**
     * æ¥å¿
     */
    private static final Log LOGGER = LogFactory.getLog(SocketServer1.class);

    public static void main(String[] args) throws Exception {
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);
        ServerSocket serverSocket = serverChannel.socket();
        serverSocket.setReuseAddress(true);
        serverSocket.bind(new InetSocketAddress(83));

        Selector selector = Selector.open();
        //æ³¨æãæå¡å¨ééåªè½æ³¨åSelectionKey.OP_ACCEPTäºä»¶
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);

        try {
            while(true) {
                //å¦ææ¡ä»¶æç«ï¼è¯´ææ¬æ¬¡è¯¢é®selectorï¼å¹¶æ²¡æè·åå°ä»»ä½åå¤å¥½çãæå´è¶£çäºä»¶
                //javaç¨åºå¯¹å¤è·¯å¤ç¨IOçæ¯æä¹åæ¬äºé»å¡æ¨¡å¼ åéé»å¡æ¨¡å¼ä¸¤ç§ã
                if(selector.select(100) == 0) {
                    //================================================
                    //      è¿éè§ä¸å¡æåµï¼å¯ä»¥åä¸äºç¶å¹¶åµçäºæ
                    //================================================
                    continue;
                }
                //è¿éå°±æ¯æ¬æ¬¡è¯¢é®æä½ç³»ç»ï¼æè·åå°çâæå³å¿çäºä»¶âçäºä»¶ç±»å(æ¯ä¸ä¸ªééé½æ¯ç¬ç«ç)
                Iterator<SelectionKey> selecionKeys = selector.selectedKeys().iterator();

                while(selecionKeys.hasNext()) {
                    SelectionKey readyKey = selecionKeys.next();
                    //è¿ä¸ªå·²ç»å¤ççreadyKeyä¸å®è¦ç§»é¤ãå¦æä¸ç§»é¤ï¼å°±ä¼ä¸ç´å­å¨å¨selector.selectedKeyséåä¸­
                    //å¾å°ä¸ä¸æ¬¡selector.select() > 0æ¶ï¼è¿ä¸ªreadyKeyåä¼è¢«å¤çä¸æ¬¡
                    selecionKeys.remove();

                    SelectableChannel selectableChannel = readyKey.channel();
                    if(readyKey.isValid() && readyKey.isAcceptable()) {
                        SocketServer1.LOGGER.info("======channelééå·²ç»åå¤å¥½=======");
                        /*
                         * å½server socket channelééå·²ç»åå¤å¥½ï¼å°±å¯ä»¥ä»server socket channelä¸­è·åsocketchanneläº
                         * æ¿å°socket channelåï¼è¦åçäºæå°±æ¯é©¬ä¸å°selectoræ³¨åè¿ä¸ªsocket channelæå´è¶£çäºæã
                         * å¦åæ æ³çå¬å°è¿ä¸ªsocket channelå°è¾¾çæ°æ®
                         * */
                        ServerSocketChannel serverSocketChannel = (ServerSocketChannel)selectableChannel;
                        SocketChannel socketChannel = serverSocketChannel.accept();
                        registerSocketChannel(socketChannel , selector);

                    } else if(readyKey.isValid() && readyKey.isConnectable()) {
                        SocketServer1.LOGGER.info("======socket channel å»ºç«è¿æ¥=======");
                    } else if(readyKey.isValid() && readyKey.isReadable()) {
                        SocketServer1.LOGGER.info("======socket channel æ°æ®åå¤å®æï¼å¯ä»¥å»è¯»==è¯»å=======");
                        readSocketChannel(readyKey);
                    }
                }
            }
        } catch(Exception e) {
            SocketServer1.LOGGER.error(e.getMessage() , e);
        } finally {
            serverSocket.close();
        }
    }

    /**
     * å¨server socket channelæ¥æ¶å°/åå¤å¥½ ä¸ä¸ªæ°ç TCPè¿æ¥åã
     * å°±ä¼åç¨åºè¿åä¸ä¸ªæ°çsocketChannelã<br>
     * ä½æ¯è¿ä¸ªæ°çsocket channelå¹¶æ²¡æå¨selectorâéæ©å¨/ä»£çå¨âä¸­æ³¨åï¼
     * æä»¥ç¨åºè¿æ²¡æ³éè¿selectoréç¥è¿ä¸ªsocket channelçäºä»¶ã
     * äºæ¯æä»¬æ¿å°æ°çsocket channelåï¼è¦åçç¬¬ä¸ä¸ªäºæå°±æ¯å°selectorâéæ©å¨/ä»£çå¨âä¸­æ³¨åè¿ä¸ª
     * socket channelæå´è¶£çäºä»¶
     * @param socketChannel æ°çsocket channel
     * @param selector selectorâéæ©å¨/ä»£çå¨â
     * @throws Exception
     */
    private static void registerSocketChannel(SocketChannel socketChannel , Selector selector) throws Exception {
        socketChannel.configureBlocking(false);
        //socketééå¯ä»¥ä¸åªå¯ä»¥æ³¨åä¸ç§äºä»¶SelectionKey.OP_READ | SelectionKey.OP_WRITE | SelectionKey.OP_CONNECT
        socketChannel.register(selector, SelectionKey.OP_READ , ByteBuffer.allocate(2048));
    }

    /**
     * è¿ä¸ªæ¹æ³ç¨äºè¯»åä»å®¢æ·ç«¯ä¼ æ¥çä¿¡æ¯ã
     * å¹¶ä¸è§å¯ä»å®¢æ·ç«¯è¿æ¥çsocket channelå¨ç»è¿å¤æ¬¡ä¼ è¾åï¼æ¯å¦å®æä¼ è¾ã
     * å¦æä¼ è¾å®æï¼åè¿åä¸ä¸ªtrueçæ è®°ã
     * @param socketChannel
     * @throws Exception
     */
    private static void readSocketChannel(SelectionKey readyKey) throws Exception {
        SocketChannel clientSocketChannel = (SocketChannel)readyKey.channel();
        //è·åå®¢æ·ç«¯ä½¿ç¨çç«¯å£
        InetSocketAddress sourceSocketAddress = (InetSocketAddress)clientSocketChannel.getRemoteAddress();
        Integer resoucePort = sourceSocketAddress.getPort();

        //æ¿å°è¿ä¸ªsocket channelä½¿ç¨çç¼å­åºï¼åå¤è¯»åæ°æ®
        //å¨åæï¼å°è¯¦ç»è®²è§£ç¼å­åºçç¨æ³æ¦å¿µï¼å®éä¸éè¦çå°±æ¯ä¸ä¸ªåç´ capacity,positionålimitã
        ByteBuffer contextBytes = (ByteBuffer)readyKey.attachment();
        //å°ééçæ°æ®åå¥å°ç¼å­åºï¼æ³¨ææ¯åå¥å°ç¼å­åºã
        //ç±äºä¹åè®¾ç½®äºByteBufferçå¤§å°ä¸º2048 byteï¼æä»¥å¯ä»¥å­å¨åå¥ä¸å®çæåµ
        //æ²¡å³ç³»ï¼æä»¬åé¢æ¥è°æ´ä»£ç ãè¿éæä»¬ææ¶çè§£ä¸ºä¸æ¬¡æ¥åå¯ä»¥å®æ
        int realLen = -1;
        try {
            realLen = clientSocketChannel.read(contextBytes);
        } catch(Exception e) {
            //è¿éæåºäºå¼å¸¸ï¼ä¸è¬å°±æ¯å®¢æ·ç«¯å ä¸ºæç§åå ç»æ­¢äºãæä»¥å³é­channelå°±è¡äº
            SocketServer1.LOGGER.error(e.getMessage());
            clientSocketChannel.close();
            return;
        }

        //å¦æç¼å­åºä¸­æ²¡æä»»ä½æ°æ®(ä½å®éä¸è¿ä¸ªä¸å¤ªå¯è½ï¼å¦åå°±ä¸ä¼è§¦åOP_READäºä»¶äº)
        if(realLen == -1) {
            SocketServer1.LOGGER.warn("====ç¼å­åºæ²¡ææ°æ®? ====");
            return;
        }

        //å°ç¼å­åºä»åç¶æåæ¢ä¸ºè¯»ç¶æ(å®éä¸è¿ä¸ªæ¹æ³æ¯è¯»åæ¨¡å¼äºåæ¢)ã
        //è¿æ¯java nioæ¡æ¶ä¸­çè¿ä¸ªsocket channelçåè¯·æ±å°å¨é¨ç­å¾ã
        contextBytes.flip();
        //æ³¨æä¸­æä¹±ç çé®é¢ï¼æä¸ªäººåå¥½æ¯ä½¿ç¨URLDecoder/URLEncoderï¼è¿è¡è§£ç¼ç ã
        //å½ç¶java nioæ¡æ¶æ¬èº«ä¹æä¾ç¼è§£ç æ¹å¼ï¼çä¸ªäººå¯
        byte[] messageBytes = contextBytes.array();
        String messageEncode = new String(messageBytes , "UTF-8");
        String message = URLDecoder.decode(messageEncode, "UTF-8");

        //å¦ææ¶å°äºâoverâå³é®å­ï¼æä¼æ¸ç©ºbufferï¼å¹¶ååæ°æ®ï¼
        //å¦åä¸æ¸ç©ºç¼å­ï¼è¿è¦è¿åbufferçâåç¶æâ
        if(message.indexOf("over") != -1) {
            //æ¸ç©ºå·²ç»è¯»åçç¼å­ï¼å¹¶ä»æ°åæ¢ä¸ºåç¶æ(è¿éè¦æ³¨æclear()åcapacity()ä¸¤ä¸ªæ¹æ³çåºå«)
            contextBytes.clear();
            SocketServer1.LOGGER.info("ç«¯å£:" + resoucePort + "å®¢æ·ç«¯åæ¥çä¿¡æ¯======message : " + message);

            //======================================================
            //          å½ç¶æ¥åå®æåï¼å¯ä»¥å¨è¿éæ­£å¼å¤çä¸å¡äº        
            //======================================================

            //ååæ°æ®ï¼å¹¶å³é­channel
            ByteBuffer sendBuffer = ByteBuffer.wrap(URLEncoder.encode("ååå¤çç»æ", "UTF-8").getBytes());
            clientSocketChannel.write(sendBuffer);
            clientSocketChannel.close();
        } else {
            SocketServer1.LOGGER.info("ç«¯å£:" + resoucePort + "å®¢æ·ç«¯ä¿¡æ¯è¿æªæ¥åå®ï¼ç»§ç»­æ¥å======message : " + message);
            //è¿æ¯ï¼limitåcapacityçå¼ä¸è´ï¼positionçä½ç½®æ¯realLençä½ç½®
            contextBytes.position(realLen);
            contextBytes.limit(contextBytes.capacity());
        }
    }
}
```

ä»£ç ä¸­çæ³¨éæ¯æ¯è¾æ¸æ¥çï¼ä½æ¯è¿æ¯è¦å¯¹å ä¸ªå³é®ç¹è¿è¡ä¸ä¸è®²è§£:

- serverChannel.register(Selector sel, int ops, Object att): å®éä¸register(Selector sel, int ops, Object att)æ¹æ³æ¯ServerSocketChannelç±»çç¶ç±»AbstractSelectableChannelæä¾çä¸ä¸ªæ¹æ³ï¼è¡¨ç¤ºåªè¦ç»§æ¿äºAbstractSelectableChannelç±»çå­ç±»é½å¯ä»¥æ³¨åå°éæ©å¨ä¸­ãéè¿è§å¯æ´ä¸ªAbstractSelectableChannelç»§æ¿å³ç³»ï¼ä¸å¾ä¸­çè¿äºç±»å¯ä»¥è¢«æ³¨åå°éæ©å¨ä¸­:

![img](https://pdai.tech/_images/io/java-io-nio-6.png)

- SelectionKey.OP_ACCEPT: ä¸åçChannelå¯¹è±¡å¯ä»¥æ³¨åçâæå³å¿çäºä»¶âæ¯ä¸ä¸æ ·çãä¾å¦ServerSocketChannelé¤äºè½å¤è¢«åè®¸å³æ³¨OP_ACCEPTäºä»¶å¤ï¼ä¸åè®¸åå³å¿å¶ä»äºä»¶äº(å¦åè¿è¡æ¶ä¼æåºå¼å¸¸)ãä»¥ä¸æ¢³çäºå¸¸ä½¿ç¨çAbstractSelectableChannelå­ç±»å¯ä»¥æ³¨åçäºä»¶åè¡¨:
- 

| ééç±»              | ééä½ç¨     | å¯å³æ³¨çäºä»¶                                                 |
| ------------------- | ------------ | ------------------------------------------------------------ |
| ServerSocketChannel | æå¡å¨ç«¯éé | SelectionKey.OP_ACCEPT                                       |
| DatagramChannel     | UDPåè®®éé  | SelectionKey.OP_READãSelectionKey.OP_WRITE                  |
| SocketChannel       | TCPåè®®éé  | SelectionKey.OP_READãSelectionKey.OP_WRITEãSelectionKey.OP_CONNECT |

å®éä¸éè¿æ¯ä¸ä¸ªAbstractSelectableChannelå­ç±»æå®ç°çpublic final int validOps()æ¹æ³ï¼å°±å¯ä»¥æ¥çè¿ä¸ªééâå¯ä»¥å³å¿çIOäºä»¶âã

selector.selectedKeys().iterator(): å½éæ©å¨Selectoræ¶å°æä½ç³»ç»çIOæä½äºä»¶åï¼å®çselectedKeyså°å¨ä¸ä¸æ¬¡è½®è¯¢æä½ä¸­ï¼æ¶å°è¿äºäºä»¶çå³é®æè¿°å­(ä¸åçchannelï¼å°±ç®å³é®å­ä¸æ ·ï¼ä¹ä¼å­å¨æä¸¤ä¸ªå¯¹è±¡)ãä½æ¯æ¯ä¸ä¸ªâäºä»¶å³é®å­âè¢«å¤çåé½å¿é¡»ç§»é¤ï¼å¦åä¸ä¸æ¬¡è½®è¯¢æ¶ï¼è¿ä¸ªäºä»¶ä¼è¢«éå¤å¤çã

> Returns this selectorâs selected-key set. Keys may be removed from, but not directly added to, the selected-key set. Any attempt to add an object to the key set will cause an UnsupportedOperationException to be thrown. The selected-key set is not thread-safe.

#### JAVAå®ä¾æ¹è¿

ä¸é¢çä»£ç ä¸­ï¼æä»¬ä¸ºäºè®²è§£selectorçä½¿ç¨ï¼å¨ç¼å­ä½¿ç¨ä¸å°±è¿è¡äºç®åãå®éçåºç¨ä¸­ï¼ä¸ºäºèçº¦åå­èµæºï¼æä»¬ä¸è¬ä¸ä¼ä¸ºä¸ä¸ªééåéé£ä¹å¤çç¼å­ç©ºé´ãä¸é¢çä»£ç æä»¬ä¸»è¦å¯¹å¶ä¸­çç¼å­æä½è¿è¡äºä¼å:

```java
package testNSocket;

import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.URLDecoder;
import java.net.URLEncoder;

import java.nio.ByteBuffer;
import java.nio.channels.SelectableChannel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;

import java.util.Iterator;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.log4j.BasicConfigurator;

public class SocketServer2 {

    static {
        BasicConfigurator.configure();
    }

    /**
     * æ¥å¿
     */
    private static final Log LOGGER = LogFactory.getLog(SocketServer2.class);

    /**
     * æ¹è¿çjava nio serverçä»£ç ä¸­ï¼ç±äºbufferçå¤§å°è®¾ç½®çæ¯è¾å°ã
     * æä»¬ä¸åæä¸ä¸ªclientéè¿socket channelå¤æ¬¡ä¼ ç»æå¡å¨çä¿¡æ¯ä¿å­å¨beffä¸­äº(å ä¸ºæ ¹æ¬å­ä¸ä¸)<br>
     * æä»¬ä½¿ç¨socketchanelçhashcodeä½ä¸ºkey(å½ç¶æ¨ä¹å¯ä»¥èªå·±ç¡®å®ä¸ä¸ªid)ï¼ä¿¡æ¯çstringbufferä½ä¸ºvalueï¼å­å¨å°æå¡å¨ç«¯çä¸ä¸ªåå­åºåMESSAGEHASHCONTEXTã
     * 
     * å¦ææ¨ä¸æ¸æ¥ConcurrentHashMapçä½ç¨åå·¥ä½åçï¼è¯·èªè¡ç¾åº¦/Google
     */
    private static final ConcurrentMap<Integer, StringBuffer> MESSAGEHASHCONTEXT = new ConcurrentHashMap<Integer , StringBuffer>();

    public static void main(String[] args) throws Exception {
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);
        ServerSocket serverSocket = serverChannel.socket();
        serverSocket.setReuseAddress(true);
        serverSocket.bind(new InetSocketAddress(83));

        Selector selector = Selector.open();
        //æ³¨æãæå¡å¨ééåªè½æ³¨åSelectionKey.OP_ACCEPTäºä»¶
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);

        try {
            while(true) {
                //å¦ææ¡ä»¶æç«ï¼è¯´ææ¬æ¬¡è¯¢é®selectorï¼å¹¶æ²¡æè·åå°ä»»ä½åå¤å¥½çãæå´è¶£çäºä»¶
                //javaç¨åºå¯¹å¤è·¯å¤ç¨IOçæ¯æä¹åæ¬äºé»å¡æ¨¡å¼ åéé»å¡æ¨¡å¼ä¸¤ç§ã
                if(selector.select(100) == 0) {
                    //================================================
                    //      è¿éè§ä¸å¡æåµï¼å¯ä»¥åä¸äºç¶å¹¶åµçäºæ
                    //================================================
                    continue;
                }
                //è¿éå°±æ¯æ¬æ¬¡è¯¢é®æä½ç³»ç»ï¼æè·åå°çâæå³å¿çäºä»¶âçäºä»¶ç±»å(æ¯ä¸ä¸ªééé½æ¯ç¬ç«ç)
                Iterator<SelectionKey> selecionKeys = selector.selectedKeys().iterator();

                while(selecionKeys.hasNext()) {
                    SelectionKey readyKey = selecionKeys.next();
                    //è¿ä¸ªå·²ç»å¤ççreadyKeyä¸å®è¦ç§»é¤ãå¦æä¸ç§»é¤ï¼å°±ä¼ä¸ç´å­å¨å¨selector.selectedKeyséåä¸­
                    //å¾å°ä¸ä¸æ¬¡selector.select() > 0æ¶ï¼è¿ä¸ªreadyKeyåä¼è¢«å¤çä¸æ¬¡
                    selecionKeys.remove();

                    SelectableChannel selectableChannel = readyKey.channel();
                    if(readyKey.isValid() && readyKey.isAcceptable()) {
                        SocketServer2.LOGGER.info("======channelééå·²ç»åå¤å¥½=======");
                        /*
                         * å½server socket channelééå·²ç»åå¤å¥½ï¼å°±å¯ä»¥ä»server socket channelä¸­è·åsocketchanneläº
                         * æ¿å°socket channelåï¼è¦åçäºæå°±æ¯é©¬ä¸å°selectoræ³¨åè¿ä¸ªsocket channelæå´è¶£çäºæã
                         * å¦åæ æ³çå¬å°è¿ä¸ªsocket channelå°è¾¾çæ°æ®
                         * */
                        ServerSocketChannel serverSocketChannel = (ServerSocketChannel)selectableChannel;
                        SocketChannel socketChannel = serverSocketChannel.accept();
                        registerSocketChannel(socketChannel , selector);

                    } else if(readyKey.isValid() && readyKey.isConnectable()) {
                        SocketServer2.LOGGER.info("======socket channel å»ºç«è¿æ¥=======");
                    } else if(readyKey.isValid() && readyKey.isReadable()) {
                        SocketServer2.LOGGER.info("======socket channel æ°æ®åå¤å®æï¼å¯ä»¥å»è¯»==è¯»å=======");
                        readSocketChannel(readyKey);
                    }
                }
            }
        } catch(Exception e) {
            SocketServer2.LOGGER.error(e.getMessage() , e);
        } finally {
            serverSocket.close();
        }
    }

    /**
     * å¨server socket channelæ¥æ¶å°/åå¤å¥½ ä¸ä¸ªæ°ç TCPè¿æ¥åã
     * å°±ä¼åç¨åºè¿åä¸ä¸ªæ°çsocketChannelã<br>
     * ä½æ¯è¿ä¸ªæ°çsocket channelå¹¶æ²¡æå¨selectorâéæ©å¨/ä»£çå¨âä¸­æ³¨åï¼
     * æä»¥ç¨åºè¿æ²¡æ³éè¿selectoréç¥è¿ä¸ªsocket channelçäºä»¶ã
     * äºæ¯æä»¬æ¿å°æ°çsocket channelåï¼è¦åçç¬¬ä¸ä¸ªäºæå°±æ¯å°selectorâéæ©å¨/ä»£çå¨âä¸­æ³¨åè¿ä¸ª
     * socket channelæå´è¶£çäºä»¶
     * @param socketChannel æ°çsocket channel
     * @param selector selectorâéæ©å¨/ä»£çå¨â
     * @throws Exception
     */
    private static void registerSocketChannel(SocketChannel socketChannel , Selector selector) throws Exception {
        socketChannel.configureBlocking(false);
        //socketééå¯ä»¥ä¸åªå¯ä»¥æ³¨åä¸ç§äºä»¶SelectionKey.OP_READ | SelectionKey.OP_WRITE | SelectionKey.OP_CONNECT
        //æåä¸ä¸ªåæ°è§ä¸º ä¸ºè¿ä¸ªsocketchanneåéçç¼å­åº
        socketChannel.register(selector, SelectionKey.OP_READ , ByteBuffer.allocate(50));
    }

    /**
     * è¿ä¸ªæ¹æ³ç¨äºè¯»åä»å®¢æ·ç«¯ä¼ æ¥çä¿¡æ¯ã
     * å¹¶ä¸è§å¯ä»å®¢æ·ç«¯è¿æ¥çsocket channelå¨ç»è¿å¤æ¬¡ä¼ è¾åï¼æ¯å¦å®æä¼ è¾ã
     * å¦æä¼ è¾å®æï¼åè¿åä¸ä¸ªtrueçæ è®°ã
     * @param socketChannel
     * @throws Exception
     */
    private static void readSocketChannel(SelectionKey readyKey) throws Exception {
        SocketChannel clientSocketChannel = (SocketChannel)readyKey.channel();
        //è·åå®¢æ·ç«¯ä½¿ç¨çç«¯å£
        InetSocketAddress sourceSocketAddress = (InetSocketAddress)clientSocketChannel.getRemoteAddress();
        Integer resoucePort = sourceSocketAddress.getPort();

        //æ¿å°è¿ä¸ªsocket channelä½¿ç¨çç¼å­åºï¼åå¤è¯»åæ°æ®
        //å¨åæï¼å°è¯¦ç»è®²è§£ç¼å­åºçç¨æ³æ¦å¿µï¼å®éä¸éè¦çå°±æ¯ä¸ä¸ªåç´ capacity,positionålimitã
        ByteBuffer contextBytes = (ByteBuffer)readyKey.attachment();
        //å°ééçæ°æ®åå¥å°ç¼å­åºï¼æ³¨ææ¯åå¥å°ç¼å­åºã
        //è¿æ¬¡ï¼ä¸ºäºæ¼ç¤ºbuffçä½¿ç¨æ¹å¼ï¼æä»¬ææç¼©å°äºbuffçå®¹éå¤§å°å°50byteï¼
        //ä»¥ä¾¿æ¼ç¤ºchannelå¯¹buffçå¤æ¬¡è¯»åæä½
        int realLen = 0;
        StringBuffer message = new StringBuffer();
        //è¿å¥è¯çæææ¯ï¼å°ç®åééä¸­çæ°æ®åå¥å°ç¼å­åº
        //æå¤§å¯åå¥çæ°æ®éå°±æ¯buffçå®¹é
        while((realLen = clientSocketChannel.read(contextBytes)) != 0) {

            //ä¸å®è¦æbufferåæ¢æâè¯»âæ¨¡å¼ï¼å¦åç±äºlimit = capacity
            //å¨readæ²¡æåæ»¡çæåµä¸ï¼å°±ä¼å¯¼è´å¤è¯»
            contextBytes.flip();
            int position = contextBytes.position();
            int capacity = contextBytes.capacity();
            byte[] messageBytes = new byte[capacity];
            contextBytes.get(messageBytes, position, realLen);

            //è¿ç§æ¹å¼ä¹æ¯å¯ä»¥è¯»åæ°æ®çï¼èä¸ä¸ç¨å³å¿positionçä½ç½®ã
            //å ä¸ºæ¯ç®åcontextBytesææçæ°æ®å¨é¨è½¬åºä¸ºä¸ä¸ªbyteæ°ç»ã
            //ä½¿ç¨è¿ç§æ¹å¼æ¶ï¼ä¸å®è¦èªå·±æ§å¶å¥½è¯»åçæç»ä½ç½®(realLenå¾éè¦)
            //byte[] messageBytes = contextBytes.array();

            //æ³¨æä¸­æä¹±ç çé®é¢ï¼æä¸ªäººåå¥½æ¯ä½¿ç¨URLDecoder/URLEncoderï¼è¿è¡è§£ç¼ç ã
            //å½ç¶java nioæ¡æ¶æ¬èº«ä¹æä¾ç¼è§£ç æ¹å¼ï¼çä¸ªäººå¯
            String messageEncode = new String(messageBytes , 0 , realLen , "UTF-8");
            message.append(messageEncode);

            //ååæ¢æâåâæ¨¡å¼ï¼ç´æ¥æåµç¼å­çæ¹å¼ï¼æå¿«æ·
            contextBytes.clear();
        }

        //å¦æåç°æ¬æ¬¡æ¥æ¶çä¿¡æ¯ä¸­æoverå³é®å­ï¼è¯´æä¿¡æ¯æ¥æ¶å®äº
        if(URLDecoder.decode(message.toString(), "UTF-8").indexOf("over") != -1) {
            //åä»messageHashContextä¸­ï¼ååºä¹åå·²ç»æ¶å°çä¿¡æ¯ï¼ç»åæå®æ´çä¿¡æ¯
            Integer channelUUID = clientSocketChannel.hashCode();
            SocketServer2.LOGGER.info("ç«¯å£:" + resoucePort + "å®¢æ·ç«¯åæ¥çä¿¡æ¯======message : " + message);
            StringBuffer completeMessage;
            //æ¸ç©ºMESSAGEHASHCONTEXTä¸­çåå²è®°å½
            StringBuffer historyMessage = MESSAGEHASHCONTEXT.remove(channelUUID);
            if(historyMessage == null) {
                completeMessage = message;
            } else {
                completeMessage = historyMessage.append(message);
            }
            SocketServer2.LOGGER.info("ç«¯å£:" + resoucePort + "å®¢æ·ç«¯åæ¥çå®æ´ä¿¡æ¯======completeMessage : " + URLDecoder.decode(completeMessage.toString(), "UTF-8"));

            //======================================================
            //          å½ç¶æ¥åå®æåï¼å¯ä»¥å¨è¿éæ­£å¼å¤çä¸å¡äº        
            //======================================================

            //ååæ°æ®ï¼å¹¶å³é­channel
            ByteBuffer sendBuffer = ByteBuffer.wrap(URLEncoder.encode("ååå¤çç»æ", "UTF-8").getBytes());
            clientSocketChannel.write(sendBuffer);
            clientSocketChannel.close();
        } else {
            //å¦ææ²¡æåç°æâoverâå³é®å­ï¼è¯´æè¿æ²¡ææ¥åå®ï¼åå°æ¬æ¬¡æ¥åå°çä¿¡æ¯å­å¥messageHashContext
            SocketServer2.LOGGER.info("ç«¯å£:" + resoucePort + "å®¢æ·ç«¯ä¿¡æ¯è¿æªæ¥åå®ï¼ç»§ç»­æ¥å======message : " + URLDecoder.decode(message.toString(), "UTF-8"));
            //æ¯ä¸ä¸ªchannelå¯¹è±¡é½æ¯ç¬ç«çï¼æä»¥å¯ä»¥ä½¿ç¨å¯¹è±¡çhashå¼ï¼ä½ä¸ºå¯ä¸æ ç¤º
            Integer channelUUID = clientSocketChannel.hashCode();

            //ç¶åè·åè¿ä¸ªchannelä¸ä»¥åå·²ç»è¾¾å°çmessageä¿¡æ¯
            StringBuffer historyMessage = MESSAGEHASHCONTEXT.get(channelUUID);
            if(historyMessage == null) {
                historyMessage = new StringBuffer();
                MESSAGEHASHCONTEXT.put(channelUUID, historyMessage.append(message));
            }
        }
    }
}
```

ä»¥ä¸ä»£ç åºè¯¥æ²¡æè¿å¤éè¦è®²è§£çäºãå½ç¶ï¼æ¨è¿æ¯å¯ä»¥å å¥çº¿ç¨æ± ææ¯ï¼è¿è¡å·ä½çä¸å¡å¤çãæ³¨æï¼ä¸å®æ¯çº¿ç¨æ± ï¼å ä¸ºè¿æ ·å¯ä»¥ä¿è¯çº¿ç¨è§æ¨¡çå¯æ§æ§ã

### å¤è·¯å¤ç¨IOçä¼ç¼ºç¹

- ä¸ç¨åä½¿ç¨å¤çº¿ç¨æ¥è¿è¡IOå¤çäº(åæ¬æä½ç³»ç»åæ ¸IOç®¡çæ¨¡åååºç¨ç¨åºè¿ç¨èè¨)ãå½ç¶å®éä¸å¡çå¤çä¸­ï¼åºç¨ç¨åºè¿ç¨è¿æ¯å¯ä»¥å¼å¥çº¿ç¨æ± ææ¯ç
- åä¸ä¸ªç«¯å£å¯ä»¥å¤çå¤ç§åè®®ï¼ä¾å¦ï¼ä½¿ç¨ServerSocketChannelæµæµçæå¡å¨ç«¯å£çå¬ï¼æ¢å¯ä»¥å¤çTCPåè®®åå¯ä»¥å¤çUDPåè®®ã
- æä½ç³»ç»çº§å«çä¼å: å¤è·¯å¤ç¨IOææ¯å¯ä»¥æ¯æä½ç³»ç»çº§å«å¨ä¸ä¸ªç«¯å£ä¸è½å¤åæ¶æ¥åå¤ä¸ªå®¢æ·ç«¯çIOäºä»¶ãåæ¶å·æä¹åæä»¬è®²å°çé»å¡å¼åæ­¥IOåéé»å¡å¼åæ­¥IOçææç¹ç¹ãSelectorçä¸é¨åä½ç¨æ´ç¸å½äºâè½®è¯¢ä»£çå¨âã
- é½æ¯åæ­¥IO: ç®åæä»¬ä»ç»ç é»å¡å¼IOãéé»å¡å¼IOçè³åæ¬å¤è·¯å¤ç¨IOï¼è¿äºé½æ¯åºäºæä½ç³»ç»çº§å«å¯¹âåæ­¥IOâçå®ç°ãæä»¬ä¸ç´å¨è¯´âåæ­¥IOâï¼ä¸ç´é½æ²¡æè¯¦ç»è¯´ï¼ä»ä¹å«åâåæ­¥IOâãå®éä¸ä¸å¥è¯å°±å¯ä»¥è¯´æ¸æ¥: åªæä¸å±(åæ¬ä¸å±çæç§ä»£çæºå¶)ç³»ç»è¯¢é®ææ¯å¦ææä¸ªäºä»¶åçäºï¼å¦åæä¸ä¼ä¸»å¨åè¯ä¸å±ç³»ç»äºä»¶åçäº:



## åèæç« 

- è½¬è½½[Java å¨æ ç¥è¯ä½ç³»](https://www.pdai.tech/md/java/io/java-io-nio-select-epoll.html)

  