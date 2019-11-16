## Wirtualizacja
### Konfiguracja SSH i firewalla
Serwer - połączenie SSH
```
service sshd start
```
Serwer - uruchomienie serwera Apache
```
sudo yum -y install httpd

sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp

sudo systemctl start httpd
```

Klient
```
ssh root@ip
```

## OSINT
#### 1. hunter.io
#### 2. 1.4 billion passwords
https://gist.github.com/scottlinux/9a3b11257ac575e4f71de811322ce6b3
#### 3. theHarvester
Automatyczne zbieranie informacji dostępnych w sieci przez wyszukiwarkę
```
theHarvester -d tesla.com -l 500 -b google
```
#### 4. Bluto
Bardzo dużo informacji o domenie, brute forcing subdomains wymusza odpowiedzi subdomen(404 - brak odpowiedzi)
#### 5. crt.sh
Wykrywanie subdomen
```
%.pg.edu.pl
```
#### 6. builtwith.com / wappalyzer
Wykrywanie technologii wykonania stron internetowych - możliwość szukania ich wrażliwych punktów.
#### 7. haveibeenpwned.com/

## Scanning
#### network sweep
```
nmap -sn 192.168.1.0/24
```
#### staged scanning
Krok 1 - sprawdzenie aktywnych portów:
```
nmap -T4 -p- 192.168.1.103
```
Krok 2 - dokładna analiza tylko aktywnych portów(oszczędność czasu):
```
nmap -T4 -A -p80,20 192.168.1.103
```
#### UDP scan
- unikać skanowania wszystkich portów, bo będzie trwało godzinami
- może wystąpić wiele rezultatów false-positive
- nie do końca można ufać tym skanom, ale nie powinno się ich lekceważyć
```
sudo nmap -T4 -sU -p8000 192.168.1.103
```
#### nmap scripts
Katalog z dostępnymi skryptami:
```
ls /usr/share/nmap/scripts/
```
Przykładowy skrypt:
```
nmap -p 443 --script=ssl-enum-ciphers tesla.com
```
