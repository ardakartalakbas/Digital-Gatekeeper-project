# Digital-Gatekeeper-project
Digital gatekeeper projesi 14. hafta

Merhaba arkadaşlar bu repoda size Digital gatekeeper projesi 14. hafta sunumu için yaptıklarımı sıralayacağım.

İlk olarak daha önceki haftalarda ağı dinlemek için kullandığımız snorta yeni bir kural yazarak nmap scanlerini yakalamasını sağlıyoruz:

"/etc/snort/rules/local.rules"
"
# SYN taraması
alert tcp any any -> any any (msg:"Port Scan Detected - SYN"; flags:S; detection_filter:track by_src, count 10, seconds 5; sid:1000010; rev:1;)

# NULL taraması
alert tcp any any -> any any (msg:"Port Scan Detected - NULL"; flags:0; detection_filter:track by_src, count 5, seconds 5; sid:1000011; rev:1;)

# XMAS taraması
alert tcp any any -> any any (msg:"Port Scan Detected - XMAS"; flags:FPU; detection_filter:track by_src, count 5, seconds 5; sid:1000012; rev:1;) "

Ve aşağıdaki komumtu yazarak snortun dinlemesini başlatıyoruz:

"sudo snort -c /etc/snort/snort.lua -i eth1 -l /var/log/snort/ -A alert_fast &"

Bu komut snortun ağı dinlemesini ve yakaladığı nmap scanlerini belirtilen yolda loglamasını sağlıyor.

Ardından windows makinamıza "https://nmap.org/download.html" linkinden indirdiğimiz nmap toolunu cmd'de yönetici olarak çalıştırıyoruz ve kali makinamıza port taraması gerçekleştiriyoruz: "nmap -sS -p 1-1000 192.168.10.10"
<img width="547" height="122" alt="image" src="https://github.com/user-attachments/assets/f22463a0-6376-42cb-a7a9-f913e12dec16" />

Ardından kalide snorta eklediğimiz kuralın çalıştığını ve nmap port taramalarını yakaladığını ve bunları logladığını görüyoruz:

<img width="907" height="506" alt="image" src="https://github.com/user-attachments/assets/ccd6355c-4853-4509-9f84-d05a6196855d" />

Böylelikle projenin 14.1 gereksinimlerini tamamlamış olup 14.2 ye geçiş yapıyoruz.

İlk olarak Fail2ban'ı makinamıza kurmamız gerekiyor: 

"sudo apt update && sudo apt install fail2ban -y"  komutu ile toolu kuruyoruz.

Ardından snort için özel bir filtre oluşturuyoruz:

"sudo nano /etc/fail2ban/filter.d/snort.conf"  komutu ile belirttiğimiz yola bir dosya oluşturuyoruz ve içine:

"[Definition]
failregex = .*Port Scan Detected.* \{<HOST>\} ->
ignoreregex ="

kuralıyla bir filtre oluşturuyoruz.

"sudo nano /etc/fail2ban/jail.local" Komutu ile bir jail tanımlıyoruz ve içine:

"[snort-portscan]
enabled  = true
filter   = snort
backend  = polling
action   = iptables-allports[name=snort, protocol=all]
logpath  = /var/log/snort/alert_fast.txt
maxretry = 1
findtime = 120
bantime  = 600"

kuralını ekliyoruz.

"sudo systemctl start fail2ban
sudo fail2ban-client status
sudo fail2ban-client status snort-portscans"   Komutlarını sırasıyla yazarak fail2ban'ı test için başlatıyoruz.


Ardından aşağıdaki komutu çalıştırıp fail2ban filterının snort logları ile eşleşmesini kontrol ediyoruz:

"sudo fail2ban-regex /var/log/snort/alert_fast.txt /etc/fail2ban/filter.d/snort.conf"

Eğer eşleşmede sorun yoksa fail2ban'ı yeniden başlatıyoruz.

Ardından snort ile dinlemeyi başlatıp windows makinamızdan nmap scan yaptığımızda fail2banın logları kullanarak windows makinadan gelen trafiği engellediğini görüyoruz.

<img width="901" height="254" alt="image" src="https://github.com/user-attachments/assets/0602fb7d-9e5c-4e07-a99e-c2f3de1317c3" />

Already banned ifadesi karşıdaki ipnin halihazırda banlandığını ve trafiğinin reddedildiğini gösteriyor yani böylelikle 14.2 koşulunu da sağlamış oluyoruz.













