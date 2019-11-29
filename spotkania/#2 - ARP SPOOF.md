![foto](https://www.incapsula.com/images/illustrations/web-app-security-mini-site/man-in-the-middle-mitm.jpg)
### ARP
**ARP - Address Resolution Protocol (ARP)** – protokół sieciowy umożliwiający mapowanie logicznych adresów warstwy sieciowej (warstwa 3) na fizyczne adresy warstwy łącza danych (2) - innymi słowy, kojarzenie adresów IP z adresami MAC w tablicy ARP.

Gdy użytkownik chce przesłać dane do komputera o adresie IP 192.168.1.34, to:

1. Karta sieciowa w komputerze źródłowym wysyła zapytanie ARP: kto ma adres IP 192.168.1.34?

2. Karta sieciowa w komputerze docelowym o adresie MAC 00:07:95:03:1A:7E wysyła odpowiedź: Hej to ja!

3. W komputerze nadawcy zostaje do dynamicznej tablicy ARP dodany wpis: IP 192.168.1.34 -> MAC 00:07:95:03:1A:7E.

4. Wszystkie pakiety z docelowym adresem IP 192.168.1.34 są przez warstwę łącza danych tłumaczone na pakiety z docelowym adresem MAC 00:07:95:03:1A:7E.

### ARP SPOOFING
Rozsyłanie w sieci LAN spreparowanych pakietów ARP zawierających fałszywe adresy MAC. 

W sytuacji, gdy atakowana maszyna posiada już poprawny adres warstwy łącza danych dla celu, atakujący może podmienić go, wysyłając odpowiednio spreparowaną „odpowiedź”, mimo iż atakowana maszyna nie wysłała zapytania. Spreparowana odpowiedź zostanie zaakceptowana, a wpis w tablicy ARP atakowanej maszyny – zmieniony.

1. Karta sieciowa w komputerze źródłowym wysyła zapytanie ARP: Kto ma adres IP 192.168.1.34?

2. Karta sieciowa w komputerze crackera o adresie MAC 00:C0:DF:01:AE:43 wysyła odpowiedź: Hej to ja!

3. W komputerze nadawcy do dynamicznej tablicy ARP trafia wpis: IP 192.168.1.34 -> MAC 00:C0:DF:01:AE:43.

4. Wszystkie pakiety z docelowym adresem IP 192.168.1.34 są przez warstwę łącza danych tłumaczone na pakiety z docelowym adresem MAC 00:C0:DF:01:AE:43 i trafiają do komputera crackera.

5. Cracker dodaje odpowiedni wpis do swojej tablicy ARP wiążący prawdziwy adres IP komputera docelowego 192.168.1.34 z jego fizycznym adresem MAC 00:07:95:03:1A:7E

6. Haker uruchamia na swoim komputerze usługę IP Forwarding, pozwalającą przekierowywać komunikaty sieciowe protokołu IP.

7. Wszystkie pakiety z docelowym adresem IP 192.168.1.34 cracker wysyła do ich prawdziwego odbiorcy z adresem MAC 00:07:95:03:1A:7E.

**robota**

brak arpspoofa

```
apt install dsniff
```

packet forwarding

```
echo > 1 /proc/sys/net/ipv4/ip_forward
```

arpspoof

```
arpspoof -i eth0 -t target -r router
```

stronka do testowania
```
http://testing-ground.scraping.pro/login
```
