# **Nmap = Network Mapper**
- هي أداة مفتوحة المصدر لتحليل الشبكات والتدقيق الأمني، مكتوبة بلغات C و C++ و Python و Lua. 
-  وظيفتها:
    
    1. **عمل فحص (Scan) للشبكات** لمعرفة الأجهزة (Hosts) اللي شغالة.
        
    2. **معرفة البورتات والخدمات اللي بتشتغل** على كل جهاز (مع الاسم والنسخة لو ممكن).
        
    3. **تحديد نظام التشغيل (OS Detection)** وإصداره.
        
    4. **تحليل إعدادات الأمان** زي الـ Firewall والـ IDS (Intrusion Detection System).

## **أهم استخدامات Nmap**

- **تدقيق أمان الشبكات (Security Audit).**
    
- **محاكاة اختبارات الاختراق (Penetration Testing).**
    
- **فحص إعدادات الـ Firewall والـ IDS.**
    
- **رسم خريطة الشبكة (Network Mapping).**
    
- **تحليل الاستجابات (Response Analysis).**
    
- **تحديد البورتات المفتوحة (Open Ports).**
    
- **تقييم الثغرات (Vulnerability Assessment).**

## **هيكل Nmap – Nmap Architecture**

تقريبًا أي فحص باستخدام Nmap بيتقسم للمراحل دي:

1. **Host Discovery** → هل الجهاز شغال ولا لا؟
    
2. **Port Scanning** → أي بورتات مفتوحة؟
    
3. **Service Enumeration** → إيه الخدمات اللي شغالة على البورتات دي؟
    
4. **OS Detection** → نظام التشغيل وإصداره.
    
5. **Nmap Scripting Engine (NSE)** → تعمل سكريبتات جاهزة للفحص المتقدم (زي البحث عن ثغرات أو كلمات سر ضعيفة).
    

---

### **الصيغة العامة للأوامر**

```bash
nmap <scan types> <options> <target>
```
- `<scan types>` = نوع الفحص (مثلاً -sS أو -sU).
    
- `<options>` = خيارات إضافية (زي -p للبورتات أو -A للفحص الكامل).
    
- `<target>` = الهدف (IP أو اسم دومين).

### **أنواع الـ Scans في Nmap**


#### **1. TCP Scans**

- `-sS` → **SYN Scan** (الأكثر استخدامًا والأسرع).
    
    - بيبعت SYN Packet.
        
    - لو السيرفر رد بـ **SYN-ACK** → البورت **Open**.
        
    - لو رد بـ **RST** → البورت **Closed**.
        
    - لو مفيش رد → البورت **Filtered** (Firewall بيمنع الرد بس ممكن يكون **open** عادي).
        
    - الميزة: الفحص سريع ومش بيكمل الـ TCP Handshake (Stealth Scan) [مش بيتقفش ف ملفات ال**log**].


- `-sT` → **Connect Scan**.
    
    - بيعمل الـ 3-way Handshake كامل (أبطأ – أسهل في الاكتشاف).
        
- `-sA` → **ACK Scan**.
    
    - بيتأكد هل البورت محمي بـ Firewall ولا لا (مش بيقولك Open أو Closed).
    

---

#### **2. UDP Scan**

- `-sU` → **UDP Scan**
    
    - أصعب من الـ TCP لأنه بدون Handshake.
        
    - لو السيرفر بيرد → البورت Open.
        
    - لو بيبعت ICMP Unreachable → البورت Closed.
        
    - لو مفيش رد → ممكن يكون Open أو Firewall بيمنع.
        

---

## **ليه بنعمل Host Discovery في الأول خالص؟**

- في اختبار الاختراق الداخلي (Internal Penetration Test) لازم نعرف **أي الأجهزة شغّالة (Online)** جوة الشبكة قبل ما نبدأ نفحص البورتات أو الخدمات.
    
- الNmap في طرق يعرفنا الأجهزة (Hosts) شغالة ولا لا (**UP or Down**).
    
- أهم طريقة وأسهلها: **ICMP Echo Requests (Ping)** → لو الجهاز بيرد يبقى شغال.
    
- لكن لو فيه Firewall بيمنع الـ Ping، نحتاج لطرق تانية (هنشوفها لاحقًا في Firewall & IDS Evasion).
    

---

### **ملاحظات مهمة:**

1. **لازم تخزن كل Scan** (باستخدام `-oA`) علشان تقدر ترجع له وتقارن النتائج أو تستخدمها في التقرير.
    
2. **أدوات مختلفة ممكن تطلع نتائج مختلفة**، فتوثيق النتائج مهم علشان تعرف الفرق بينهم.
    

---

### **السيناريوهات المختلفة لفحص الأجهزة (Host Discovery)**


#### **1. فحص شبكة كاملة (Network Range Scan)**

لو عايز تشوف كل الأجهزة اللي شغالة في رينج كامل (مثلاً /24):

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```
**شرح الأوامر:**

- `10.129.2.0/24` → نطاق الشبكة.
    
- `-sn` → تعطيل فحص البورتات، نكتفي بفحص الأجهزة الحية.
    
- `-oA tnet` → حفظ النتائج بكل الصيغ (nmap, xml, grepable) باسم tnet.
    
- `grep for | cut -d" " -f5` → يعرض فقط الـ IPs اللي شغالة.
    

**النتيجة:**

`10.129.2.4 10.129.2.10 10.129.2.11 10.129.2.18 10.129.2.19 10.129.2.20 10.129.2.28`

> ملاحظة: لو الـ Firewall بيمنع الـ Ping، الأجهزة ممكن تظهر كأنها Down رغم إنها شغالة.



#### **2. فحص قائمة IP جاهزة (Scan IP List)**

لو معاك ملف hosts.lst فيه الـ IPs:


**المحتوى:**

`10.129.2.4 10.129.2.10 10.129.2.11 10.129.2.18 10.129.2.19 10.129.2.20 10.129.2.28`

**تشغيل الفحص:**

```bash
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

- `-iL hosts.lst` → الفحص يتم على كل IP موجود في الملف.
    

**النتيجة:**

`10.129.2.18 10.129.2.19 10.129.2.20`

> يعني بس 3 أجهزة من 7 ردوا. الباقي ممكن يكون واصل لكن بيمنع الـ ICMP Echo.



#### **3. فحص مجموعة IP صغيرة (Scan Multiple IPs)**

لو عندك IPs متفرقة:

```bash
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20 | grep for | cut -d" " -f5
```
**لو ورا بعض (**Range**):**

```bash
sudo nmap -sn -oA tnet 10.129.2.18-20 | grep for | cut -d" " -f5
```
**النتيجة:**

`10.129.2.18 10.129.2.19 10.129.2.20`


#### **4. فحص IP واحد (Single Host Discovery)**

قبل ما تفحص بورتات وخدمات، تأكد إن الجهاز حي:

```bash
sudo nmap 10.129.2.18 -sn -oA host
```
**الناتج:**

`Nmap scan report for 10.129.2.18 Host is up (0.087s latency). MAC Address: DE:AD:00:00:BE:EF`

    

### **تفاصيل تقنية مهمة عن الـ Ping في Nmap**

- بشكل افتراضي: Nmap بيستخدم **ARP Ping أولًا** (في الشبكات المحلية LAN).
    
- بعد كده ممكن يرسل **ICMP Echo Request (-PE)** لو عايز تجبره.
    
- استخدم `--packet-trace` علشان تشوف كل الـ Packets المرسلة والمستقبَلة.
    
- استخدم `--reason` علشان تعرف **ليه Nmap صنف الجهاز إنه Up** (بناءً على ARP أو ICMP).
    
- لو مش عايز يستخدم ARP نهائيًا: استخدم `--disable-arp-ping`.
    

---

#### **مثال: فرض استخدام ICMP Echo بدل ARP**

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```
**الناتج:**

`SENT ARP who-has 10.129.2.18 RCVD ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF Host is up (0.023s latency). MAC Address: DE:AD:00:00:BE:EF`

- هنا Nmap اعتبر الجهاز Up من خلال رد الـ ARP.
    


#### **تعطيل الـ ARP Ping وإرسال ICMP فقط**

`sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping`

**الناتج:**

`SENT ICMP Echo request RCVD ICMP Echo reply Host is up (0.086s latency).`

- هنا أجبرنا Nmap يستخدم ICMP فقط بدون ARP.
    

---

### **الخلاصة**

1. **ابدأ دائمًا بـ Host Discovery قبل أي Port Scan.**
    
2. **استخدم -sn لتعطيل فحص البورتات واكتشاف الأجهزة فقط.**
    
3. **احفظ كل النتائج (-oA) للمقارنة والتوثيق.**

4. **ARP Ping أسرع في LAN، وICMP مفيد لو عايز فحص كلاسيكي.**
    
5. **--reason و --packet-trace أدوات مهمة لفهم النتائج.**

6. **الNmap في الشبكات الداخلية بيستخدم افتراضيا ال**ARP Ping** لأ عادة الشركات بتمنع ال**ICMP Ping** العادي بينها وبين بعض ,عشان كده وانت بتفحص شبكة انت جواها اللي هيشتغل ال**ARP Ping** ,على عكس لو بتفحص شبكة خارجية عنك او انت براها ال**ARP Ping** مش بيشتغل أصلا لأن هو بيشتغل في **Layer 2** وكمان بيحتاج يعمل **Broadcast** و دي حاجة الراوترات بتمنعها ,فهتستخدم ال**ICMP Ping**.**

---

## فحص المضيفين والمنافذ (**Port and Host Scanning**)
### 1. **الهدف من الPort Scanning:**
- معرفة ال**Ports** والخدمات التي تعمل عليها.
- معرفة اصدار الخدمات (**Service Versions**).
- معلومات عن الخدمات.
- معرفة نظام التشغيل.

### 2. حالات المنفاذ ال6 (**States of Ports**):

| الحالة            | الشرح                                                                                                                                                           |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **open** | .المنفذ مفتوح ويستقبل اتصالات                                                                                                                                    |
| **closed** | `RST` تحتوي TCP المنفذ مغلق والجهاز يرد بحزم                                                                                                                     |
| **Filtered** | (يمنع تحديد الحالة Firewallال) لا رد أو رد غامض                                 |
| **Unfiltered** |(**open or close**) المنفذ متاح الوصول ليه بس معرفش (TCP-ACK scanبيحصل في ال)             | 
| open \| filtered | \|If we do not get a response for a specific port, `Nmap` will set it to that state. This indicates that a firewall or packet filter may protect the port.\|        |
| closed \| filterd | This state only occurs in the **IP ID idle** scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.\|         |

### 3. مسح منافذ ال (SYN Scan vs Connect Scan)TCP
#### **أولا:** (-sS) SYN Scan 
- يتم بشكل افتراضي عند تشغيل Nmap ك root.
- يسمى Half-Open Scan لأنه يرسل SYN,وينتظر SYN-ACK,ثم يغبلق الاتصال قبل اكتماله.
- سريع وأكثر تخفيا (لأنه لا يكمل TCP handshake).
 

#### **ثانيا:** (-sT) Connect Scan
- يتم افتراضيا بدون صلاحية الroot.
- بيعمل (SYN=>SYN-ACK=>ACK) TCP handshake كاملة.
- أكثر دقة وأقل تخفيا (بيتم تسجيله في ملفات الlogs).
- جيد إذا أردت التأكد بدقة أو إذا كان جدار الحماية يسمح بالاتصالات الصادرة فقط.


### 4. أمثلة على الNmap Scans:
#### فحص أشهر 10 منافذ TCP:
```bash
sudo nmap 10.129.2.28 --top-ports=10 
```
- يظهر المنفذ وحالته (open,closed,filtered) وخدمته.
- **Top 10** => `21|22|23|25|80|110|139|443|445|3389`  
#### استخدام packet-trace-- لمعرفة ما يحدث كاملا خلف الكواليس
```bash
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping
```
- SENT => ألحزمة المرسلة
- RCVD => الحزمة المستقبلة
- `-Pn` => من غير ما يتأكد UP لأنه بيفترض يعني أن الجهاز pingبيعطل خدمة ال
- `-n` => DNSتعطيل فحص ال

#### Connect Scan on HTTPS (443):
```bash
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n -sT --reason
```
`SYN-ACK` => المنفذ مفتوح
`RST` => المنفذ مغلق
The `Connect` scan (also known as a full TCP connect scan).

### 5. حالات الFiltered Ports:
- إذا كان ال**Firewall** يسقط الحزمة => لا رد ,Nmap يعيد المحاولة 10 مرات (بطئ).
- إذا كان ال**Firewall** يرفض الحزمة => يوجد رد ,يرد ب Port unreachable (type=3/code=3)

### 6. فحص منافذ UDP (-sU) :
- أبطأ من TCP لأنه **stateless** بلا **handshake**.
- غالبا لا يوجد رد إذا كان المنفذ مفتوح.
- إذا المنفذ مغلق → الجهاز يرسل ICMP port unreachable.
- إذا الرد غامض → الحالة open|filtered.

## 7. تحديد إصدار الخدمة (-sV):
عشان تعرف نوع وإصدار السيرفر والخدمة:
```bash
sudo nmap 10.129.2.28 -p 445 -sV --reason -Pn -n --disable-arp-ping --packet-trace
```


- مفيد جداً لاختيار الـ exploit المناسب.
- يظهر تفاصيل مثل: نوع SMB أو Apache أو SSH وإصداره.

#### الخلاصة:

- SYN Scan (-sS) سريع وتخفي.
- Connect Scan (-sT) أدق لكن مكشوف.
- UDP Scan (-sU) أبطأ وأصعب في التفسير.
- Filtered ports تعني أن firewall موجود.
- استخدم -sV لمعرفة الإصدارات لتخطيط الهجمات.
- استخددم --packet-trace لفهم حركة الحزم أثناء الفحص.

`Note : To Know The Hostname Of the IP`
```bash
sudo nmap -p 139,445 --script smb-os-discovery 192.168.x.x
```
---
### ليه لازم نحفظ النتائج ؟
لما بتعمل Scans كتير بطرق مختلفة (مثلاً: TCP scan، UDP scan، أو Scans بسكريبتات معينة)، لازم تحفظ النتايج عشان:
- تقارن النتايج ببعض.
- تراجعها من غير ما تعيد الـ Scan (خصوصًا لو السيرفر بعيد أو النت بطيء).
- تستخدمها في التقارير أو لما تسلم Proof of Work.

### صيغ حفظ الملفات ب Nmap:
1. normal output (-oN) => ملف بامتداد `.nmap`
- نص عادي مقروء للبشر.
```bash
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http
```
2. Grepable output (-oG) => ملف امتداده `.gnmap`
- بيكتب كله ف سطر واحد أو سطور بسيطة.
- بيتفهم بسهولة بأوامر زي `grep` أو `awk` عشان الفتلرة.
```bash
Host: 10.129.2.28 ()  Status: Up
Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, 80/open/tcp//http///
```

3. XML output (-oX) =>ملف بصيغة `.xml`
- منظم بشكل xml.
- ممكن تحوله لأي شكل تاني (HTML,JSON) باستخدام `xsltproc`.
```bash
<port protocol="tcp" portid="22">
  <state state="open"/>
  <service name="ssh"/>
</port>
```

### لو عايز تحفظ التلاتة مرة واحد مع بعض:
بدل ما تعمل كل ملف واحد بواحد بتستخدم `-oA`:
```bash
sudo nmap 10.129.2.28 -p- -oA target
```
- `-p-` => لكل البورتات scan
- `-oA target` => حفظ النتيجة في 3 ملفات بنفس الاسم بصيغ مختلفة:
    - `target.nmap`
    - `target.gnmap`
    - `target.xml`
#### Note:
- لو مكتبتش الpath كامل هيتحفظو في المكان اللي انت واقف فيه.

### تحويل ملف الxml لصفحة HTML مفهومة :
عشان تعرض النتائج بشكل مرتب :
```bash
xsltproc target.xml -o target.html
```
- بعدها افتح `target.html` في المتصفح هتشوف جدول مرتب بكل البيانات.
---
## Service Enumeration
لما تعرف اسم الخدمة وإصدارها زي (`Apache 2.4.29`أو `OpenSSH 7.6`) هتقدر:
- تدور على ثغرات معروفة لنفس الإصدار (Exploit-DB/CVE).
- تشوف الsource code لو الخدمة open source.
- نحدد Exploit مناسب بدل ما نجرب أي حاجة وخلاص.

### أوامر Nmap المهمة:
#### 1. مسح كامل للبورتات مع تحديد الخدمة والإصدار
```bash
sudo nmap 10.129.2.28 -p- -sV
```
- `-p-` => لكل البورتات scan
- `-sV` => يحدد الخدمة وإصدارها
#### 2. متابعة حالة الscan أثناء التقدم
- دوس <space> أثناء الفحص => هيظهرلك نسبة التقدم.
- أو اكتب
```bash
--stats-every=5s
```
- هيظهرلك نسبة التقدم كل 5 ثوان.
#### 3. زيادة التفاصيل (Verbosity)
- اكتب 
```bash
v
```
- أو 
```bash
vv
```
- هيظهرلك البورتات المفتوحة بمجرد العثور عليها مش هيستنى يخلص العملية كاملة.

### Banner Grabing
- الBanner هو اللي الخدمة بتبعته أول ما عملية الاتصال تتم والNmap بيحاول يقرأه.
    - لو نجح => بيحدد الإصدار تلقائي.
    - لو فشل أو البانر ناقص => ممكن تستخدم أدوات يدوية.
#### اتصال يدوي ب nc (netcat)
```bash
nc -nv 10.129.2.28 25
```
- هتلاقي سيرفر الSMTP مثلا رد ب:
```bash
220 inlane ESMTP Postfix (Ubuntu)
```
- و دي معلومة ممكن Nmap ميطبعهاش كاملة أو ميطبعهاش خالص.

### متابعة الاتصال ب tcpdump
- عشان تشوف الpackets اللي بتتحرك أثناء الاتصال:
```bash
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
```
- هيظهرلك الTCP Handshake
    - SYN
    - SYN-ACK
    - ACK

    ### الخلاصة
1. امسح كل البورتات وحدد الخدمة والإصدار:
```bash
sudo nmap 10.129.2.28 -p- -sV -v
```
2. لو النتايج ناقصة → استخدم:
```bash
sudo nmap 10.129.2.28 -p- -sV -Pn -n --disable-arp-ping --packet-trace
```
3. لو لقيت خدمة SMTP أو HTTP أو أي خدمة بتعرض بانر → جرّب تتصل بيها يدوي:
```bash
nc -nv 10.129.2.28 25
```
4. لو عايز تشوف الـ Packets بنفسك:
```bash
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
```
 ---
## Nmap Scripting Engine (NSE)
هي ميزة في Nmap بتخليك تشغل سكريبتات مكتوبة ب لغة Lua عشان :
- تعرف معلومات زيادة عن الخدمة.
- تعمل فحص ثغرات أو Brute Force أو Exploit.
- أو ممكن تكتب سكريبتات بنفسك عادي.
والسكريبتات دي متقسمة ل **14 تصنيف** :
1. **auth** => (اسم المستخدم والباسوورد) credentialsبتجرب تكتشف ال
2. **broadcast** =>  Broadcast بيعمل اكتشاف لأجهزة الشبكة عن طريق
3. **brute** => على الخدمات Brute Force  بيعمل 
4. **default** => `-sC` السكريبتات الافتراضية اللي مع 
5. **discovery** => بيجيب معلومات إضافية عن الخدمات المتاحة
6. **dos** => (مش بنستخدمها كتير عشان بتضر الخدمة) ولا لا  DoS  بنستخدمها لو عايز نعرف الخدمة ممكن يتعملها 
7. **exploit** => بتحاول تستغل ثغرة موجودة بالفعل 
8. **external** => سكريبتات بتستخدم خدمات خارجية للتحليل
9. **fuzzer** => Bugs غريبة ومختلفة عشان يكشف لو فيه ثغرات أو Requests  بتبعت 
10. **intrusive** => فحوصات ممكن تأثر على النظام المستهدف أو تعطله
11. **malware** => Malware بيدور لو النظام المستهدف مصاب بأي 
12. **safe** => فحوصات امنة مش هتبوظ حاجة
13. **version** =>  بتحسن كشف إصدارات الخدمات
14. **vuln** => CVE بتكشف الثغرات المعروفة 

### ازاي تكتب سكريبتات الNSE ؟
#### 1. تشغيل الاسكتريبتات الافتراضية:
```bash
sudo nmap <target> -sC
```
هنا بيشغل الكسريبتات الDefault
#### 2. تشغيل تصنيف كامل من السكريبتات:
```bash
sudo nmap <target> --script <category>
```
زي
```bash
sudo nmap 10.13.191.50 --script vuln
```
هنا بيشغل كل السكريبتات اللي بتدور على ثغرات.
#### 3. تشغيل سكريبتات معينة بالاسم:
```bash
sudo nmap <target> --script <script-name>,<script-name>
```
زي 
```bash
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```
- `banner` => بيطبع بانر الخدمة
- `smtp-commands` => المتاحة SMTPبيكشف أوامر ال

### خيار (-A) Aggressive Scan
لو استخدمت:
```bash
sudo nmap <target> -A
```
بيعمل:
- Service Detection (-sV)
- OS Detection (-O)
- Traceroute (--traceroute)
- Default NSE Scripts (-sC)
مثال:
```bash
sudo nmap 10.129.2.28 -p 80 -A
```
- كشف إن السيرفر Apache 2.4.29.
- كشف إنه بيستخدم WordPress 5.3.4.
- طلع اسم الموقع من الـ `Title: `blog.inlanefreight.com
- حاول يتعرف على نظام التشغيل (Linux بنسبة 96%).
### استخدام vuln لفحص الثغرات
```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```
- بيركز على الخدمات وبيبحث في قواعد البيانات الموجودة الCVE على أي ثغرات.
- مثال:
    - WordPress Login Page → `/wp-login.php`
    - اسم مستخدم `WordPress: `admin
    - ثغرات Apache (CVE-2019-0211, CVE-2018-1312...)
ده بيوضحلك اذا السيرفر معرض لثغرات من غير ما تبحث في جوجل.

**Note:** http-enum => بيدور في ملفات الويب سيرفر
---
### ايه الفرق بين الFirewall والIDS/IPS ؟
- **Firewall** => **Drop** أو يرميها **Reject** ممكن يرفض الحزمة.(port,ip,protocol) وبيقرر السماح أو الرفض بناء على packetsبيراقب ال
> Against unauthorized connection attempts from external networks
- **IDS** => بيراقب الشبكة ويكشف الهجمات المحتملة ويبلغ المسؤول
>  scans the network for potential attacks, analyzes them, and reports any detected attacks
- **IPS** => (ip إغلاق اتصال,حظر) لكن بينفذ اجراءات تلقائية مثل `IDS`شبه ال
>complements `IDS` by taking specific defensive measures if a potential attack should have been detected

### كيف تعرف بوجود الFirewall ؟
- عندما يظهر المنفذ **filtered** وليس **open/closed** => يعني الحزم اللي تم إسقاطها بدون رد (Drop) أو رفضها (Reject) برسالة ICMP/RST.
- الأخطاء المحتملة:
    - Net Unreachable / Prohibited
    - Host Unreachable / Prohibited
    - Port Unreachable
    - Proto Unreachable

### طرق تجاوز الFirewall
#### SYN Scan (-sS)
- بيرسل حزمة SYN
    - لو رجع SYN/ACK => البورت مفتوح.
    - لو مفيش رد يبقى الFirewall بيعمل بلوك للSYN.
- المشكلة هنا ان الFirewall بيبقى شايف انك عايز تبدأ اتصال كامل من جديد ف بيبقى شايفك ويقدر يمنعك بسهولة.
#### ACK Scan (-sA)
- بيرسل حزمة ACK
    - لو رجع RST => يبقى البورت **unfiltered** (مفيش حاجة قدامه بتمنعه-سواء مفتوح أو مقفول)
    - لو مفيش رد => البورت **filtered** (فيه **firewall** قدامه)
- الميزة المهمة : معظم الجدران النارية بتسمح بمرور ACK لأنها مش بتقدر تحدد هل الاتصال ده بدأ من جوه الشبكة (داخليًا) ولا من برة (خارجيًا) - لأن الـ ACK نفسه مالوش سياق واضح لوحده، فبيفترض الـ Firewall غالبًا إنه رد على حاجة شرعية ,يعني من الاخر بيفكر أن الACK اللي جاية دي حاجة تبع اتصال قديم شغال ف مش بيفحصها بالقدر اللي بيفحص بيه الSYN لأنه بيعتبرها اتصال جديد وهو كذلك بالفعل.
مثال:
```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n
```
- لو منفذ 22 رد بRST => يبقى ليس محمي (unfiltered).
- لو 21,25 مردوش => عليهم حماية (filtered).

### الكشف عن IPS/IDS
- لو عملت Aggressive Scan ممكن الIPD/IDS يحظروا الip بتاعك.
- لو انقطع وصولك للهدف بعد الفحص => غالبا فيه IPS.
- الحل: استخدام عدة `VPS` او كذا `IP`.

### تقنيات التخفي وتجاوز الحماية
#### 1. استخدام (-D) Decoys
- يضيف Nmap عدة IP وهمية لإخاء عنوانك الحقيقي.
- مثال:
```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n -D RND:5
```
يعني استخدام 5 عناوين عشوائية كDecoys.
- **Note:** **SYN flood** حقيقية (حية) وإلا قد يسبب Decoysيجيب أن تكون ال
**SYN Flood**: هجوم DoS بيستغل الـ TCP handshake عن طريق إرسال آلاف SYN packets من غير إكمال الـ handshake (مفيش ACK نهائي)، فالسيرفر بيفضل يحجز موارد لاتصالات "نص مفتوحة" لحد ما الـ connection queue تمتلي وميقدرش يستقبل اتصالات حقيقية جديدة.
> Exploits half-open TCP connections to exhaust server resources and cause denial of service

#### 2. تغيير عنوان المصدر(-S)
- تختبر هل جدار الحماية بيمنع شبكات محددة فقط.
```bash
sudo nmap 10.129.2.28 -p 445 -O -S 10.129.2.200 -e tun0
```
- لو لقيتالمنفذ مفتوح => يبقى الحظر معمول على جهة الاتصال.

#### 3. استخدام منفذ موثوق (--source-port53)
- الFirewall غالبا بتسمح ب (port 53) DNS.
- إذا استخدمت بورت 53 كمصدر للفحص ,قد يمر بدون حظر:
```bash
sudo nmap 10.129.2.28 -p 50000 -sS --source-port 53
```
- لو البورت بقا open بدل filtered ,يبقى الحماية عليه ضعيفة.
- يمكنك حتى الاتصال مباشرة باستخدام netcat:
```bash
ncat -nv --source-port 53 10.129.2.28 50000
```

### ملخص الأدوات والخيارات :
- `-sA` => تحديد قواعد SYN-ACK
- `-D RND:5` => لتخفي المصدر باستخدام الDecoys
- `-S <IP> -e iface` => تغيير عنوان المصدر وواجهة الإرسال
- `--source-port` => تمرير الفحص من منفذ موثوق
- `dns-server <IP>` => استخدام DNS داخلي موثوق

### تلخيص:
- **ACK Scan** => يمكن اكتشاف قواعد الFirewall
- **IPS/IDS** => بتكتشفه لو الIP اتحظر بعد الفحص
- **Decoys, Source Port Spoofing, Changing IP** => تساعد في التخفي
- **DNS port(53) => غالبا ثغرة لتجاوز الحماية
---

