![foto](https://lh6.googleusercontent.com/F7jFB9zaGo1abo5wq80tbzhxCEDLiw_dd9m09p-_rc5v-HW0B3zgci8yCJ7a6EVsCrW0n8bxj8CLmvvg9kZ-1Z0mJJbPxggAbi1t-g4tfFYCZY1cV9Eeh2JInJgaZ6DGl6OZ46QOGIU_pNyvJw)
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

### Wykonanie

Celem naszego ataku będzie możliwość podglądania pakietów wychodzących i przychodzących do maszyny ofiary w jej komunikacji z routerem. Na początek używając programu wireshark, możemy zobaczyć, że póki co na maszynie atakującej widzimy jedynie ruch sieciowy przychodzący i wychodzący z naszej maszyny, oraz ramki BROADCAST i MULTICAST, które są widoczne dla wszystkich użytkowników sieci. Jeżeli jako maszyna ofirara zaczniemy wchodzić na strony internetowe, pakiety te są niewidoczne dla innych w sieci.

Żeby użyć komendy arpspoof będzie nam potrzebny pakiet dsniff:

```
apt install dsniff
```

Kolejnym krokiem który musimy wykonać, jest włączenie opcji packet forwarding na naszej maszynie atakującej.

```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Teraz pozostaje nam już sam arpspoof, w miejsce target wpisujemy adres IP maszyny którą chcemy zaatakować, a w miejsce router wpisujemy ip naszego routera w sieci(POWTÓRKA Z POPRZEDNICH ZAJĘĆ - można to szybko sprawdzić wpisując komendę "ip r", a następnie spojrzeć na adres przypisany w linii default)

```
arpspoof -i eth0 -t target -r router
```

Po wpisaniu komendy arpspoof powinniśmy mieć możliwość, poprzez program Wireshark, podglądać ramki którymi maszyna ofiara komunikuje się z Internetem - czyli widzimy teraz wszystkie ramki, a nie tylko BROADCASTy i MULTICASTy. Obecnie większość stron używa protokołu HTTPS, więc zawartość ramek jest szyfrowana. Poniżej dla przykładu umieściłem stronę która używa HTTP do przesyłania danych. Przez to, że protokół HTTP nie szyfruje ramek, jesteśmy w stanie przechwycić ich zawartość.

stronka do testowania
```
http://testing-ground.scraping.pro/login
```
![foto](https://i.imgur.com/UsYDbdD.png)

### Ukrywanie naszej aktywności w sieci

Jednym ze sposobów ukrywania naszej aktywności w sieci przed innymi jest wykorzystanie prostego skryptu napisanego w pythonie - noisy.py . 

> A simple python script that generates random HTTP/DNS traffic noise in the background while you go about your regular web browsing, to make your web traffic data less valuable for selling and for extra obscurity.


[Repozytorium na githubie zawierające noisy.py](https://github.com/1tayH/noisy)

[Tutorial na YT przedstawiający korzystanie z noisy.py](https://youtu.be/iCKj0La4Grg)
