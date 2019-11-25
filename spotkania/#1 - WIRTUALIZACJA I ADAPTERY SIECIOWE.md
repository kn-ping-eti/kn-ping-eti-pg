## VirtualBox i adaptery sieciowe

Korzystając z maszyny wirtualnej należy dostosować jej tryb pracy sieciowej w zależności od potrzeb. Na przykład ucząc się exploitowania systemu z celowo wbudowaną dużą ilością podatności na ataki, najlepiej umieścić taką dystrybucję w sieci prywatnej razem z Kalim/Parrotem, aby uniknąć dostępu z zewnątrz do tak przygotowanego środowiska.

Klikając na jedną z dodanych na naszym komputerze maszyn wirtualnych, a następnie przechodząc do zakładki Settings -> Network mamy do wyboru kilka adapterów sieciowych, które decydują o trybie pracy sieciowej maszyny wirtualnej.

![foto](https://i.imgur.com/UCawDpj.png)

Poniżej znajduje się tabela wyjaśniająca zależności dostępu pomiędzy maszyną wirtualną, komputerem który jest jej hostem, a także siecią zewnętrzną - Internetem.

![foto](https://i.imgur.com/7EeSsZk.png)

**Objaśnienia:** 

**Guest** - maszyna wirtualna, 

**Host** - fizyczny komputer na którym uruchamiamy maszynę wirtualną, 

**External network** - sieć zewnętrzna, Internet

Krótki artykuł dokładniej opisujący rodzaje adapterów sieciowych - https://www.thomas-krenn.com/pl/wiki/Konfiguracja_sieciowa_w_VirtualBox

### Tworzymy własną, wirtualną sieć komputerową (Work In Progress)
![foto](https://i.imgur.com/kv6q2wD.png)

Na potrzeby tego materiału będę korzystał z dwóch maszyn wirtualnych, jedną z Kali Linuxem, a drugą z systemem Metasploitable(możecie korzystać z dowolnych dystrybucji bazujących na Debianie/Ubuntu, np. 2 maszyny z Kalim, komendy będą te same).

#### 1. Konfiguracja maszyn wirtualnych

Dla maszyny, przez którą będziemy przekierowywali ruch ustalamy następujące adaptery sieciowe - pierwszy jako Bridged, a drugi jako Internal network - dzięki temu maszyna będzie miała dostęp do internetu, a także będzie widoczna w sieci wewnętrznej dla drugiej maszyny.

![foto](https://i.imgur.com/h7Zw9Od.png)
![foto](https://i.imgur.com/7glh0R0.png)

Następnie dla maszyny, do której nie chcemy aby możliwy był dostęp z zewnątrz, konfigurujemy tylko jeden adapter sieciowy w tryb Internal Network(ważne aby miała tę samą nazwę, jak ta ustalona dla pierwszej wirtualki)

![foto](https://i.imgur.com/qiLaHVV.png)

#### 2. Nadawanie adresów IP

Kolejnym krokiem jest przypisanie maszynom wirtualnym adresom IP w sieci wewnętrznej. Aby sprawdzić jakie adresy IP mamy przypisane dla naszych interfejsów wpisujemy komendę
```
ifconfig
```
![foto](https://i.imgur.com/5Sa0yZ1.png)

W systemie Metasploitable interesuje nas tylko interfejs eth0, gdyż reprezentuje on nasze połączenie z siecią lokalną. Można zauważyć, że brakuje w nim adresu IPv4 oznaczonego jako **inet addr**. Warto zauważyć, że w systemie Kali, przez który będziemy przekierowywali ruch z Metasploitable występuje interfejs eth0 oraz eth1 - eth0 jest przypisany do Bridged Adapter, który umożliwia połączenie z Internetem, natomiast eth1 odpowiada za sieć wewnętrzną z Metasploitable.

Jako przykład, naszym zadaniem jest przypisanie adresu 192.168.3.1/24 dla maszyny z Metasploitable:
```
sudo ip addr add 192.168.3.2/24 dev eth0
```
![foto](https://i.imgur.com/aenEbR0.png)

To samo robimy dla maszyny z Kalim, zmieniając adres który chcemy nadać na 192.168.3.1/24 oraz na interfejs eth1 i po wpisaniu komendy ifconfig powinniśmy otrzymać taki rezultat (ponownie, szukamy wiersza z **inet addr**)

![foto](https://i.imgur.com/p45Jpqr.png)

Po nadaniu adresów IP maszyny powinny widzieć się w obrębie naszej sieci wewnętrznej, co możemy sprawdzić pingując adres IP przeciwnej:

![foto](https://i.imgur.com/Mb7RoAL.png)


#### 3. Przekierowanie ruchu przez maszynę posiadającą dostęp do Internetu

Adapter sieci wewnętrznej ogranicza maszynę z systemem Metasploitable jedynie do komunikacji z innymi wirtualkami połączonymi z tą siecią.

Teraz naszym zadaniem jest, aby system Metasploitable również mógł uzyskać dostęp do Internetu, bez zmiany jego adaptera sieciowego. Jak to zrobić? Przede wszystkim musimy przekierować nasze żądania do maszyny wirtualnej z Kali Linuxem, która posiada dostęp do Internetu. Aby tego dokonać, musimy dodać w tablicy routingu adres **default**, na który będą domyślnie przekierowywane pakiety wychodzące. Aby podejrzeć tablicę routingu wpisujemy komendę:
```
ip r
```
![foto](https://i.imgur.com/Dpum1EN.png)

Dla porównania, tablica routingu dla maszyny z Kalim, która ma połączenie z Internetem i która jako adres default ma ustawiony adres mojego routera w sieci lokalnej(ustawiany automatycznie po połączeniu się z siecią WiFi).

![foto](https://i.imgur.com/44O27lX.png)

Aby ręcznie dodać adres **default** w tablicy routingu dla maszyny z systemem Metasploitable, wpisujemy komendę:

```
ip r a default via 192.168.3.1
```

Gdy spróbujemy spingować 8.8.8.8(google), coś dalej jest nie tak, jednak nasz ruch jest już przekierowywany.

![foto](https://i.imgur.com/wAvtA7S.png)

Ab zobaczyć, co dzieje się z pakietami które zostają przekierowane do maszyny z Kali Linuxem, uruchamiamy tam program Wireshark i wybieramy interfejs eth1 do monitorowania(przy czym cały czas pingujemy 8.8.8.8 z maszyny Metasploitable):

![foto](https://i.imgur.com/rOEKsGQ.png)

Widać, że pakiety z Metasploitable trafiają do Kaliego, lecz nie wychodzą one dalej. Kolejnym krokiem który musimy wykonać, jest przekierowywanie pakietów poza Kaliego oraz translacja adresów przychodzących z sieci wewnętrznej:
```
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

```
sudo iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -o eth0 -j MASQUERADE
```

#### 4. Konfiguracja DNS

Ostatnim krokiem który wykonamy, będzie dodanie możliwości pingowania adresów www zamiast IP z maszyny Metasploitable. W tym celu odczytujemy adres zapisany w pliku resolv.conf, zapisany w maszynie z systemem Kali, który zawiera adres IP serwera nazw.

![foto](https://i.imgur.com/FG1cG3i.png)

Następnie ten sam adres przepisujemy do pliku resolv.conf w maszynie z Metasploitable, który najprawdopodobniej póki co będzie pusty.

```
sudo nano /etc/resolv.conf
```

Nasz plik powinien wyglądać tak(używam vima zamiast nano), **nie zapomnijmy przepisać go przed wyjściem**:

![foto](https://i.imgur.com/yeGEryz.png)

Po wykonaniu tych operacji możemy komunikować się z publicznymi adresami IP oraz stronami www(gdybyśmy zamiast Metasploitable korzystali np. z drugiego Kali Linuxa moglibyśmy korzystać z przeglądarki Internetowej i otwierać strony)

![foto](https://i.imgur.com/j2zEP41.png)

Szybki rzut okiem na to, co można zauważyć w Wiresharku w maszynie z Kalim, monitorując interfejs eth1 w czasie pingowania z maszyny Metasploitable:

![foto](https://i.imgur.com/hPmrpdy.png)

Gotowe! Wykonaliśmy nasze zadanie. Dodatkowo, warto skorzystać z komendy:
```
iptables -t nat -nvL
```
aby zobaczyć nasz wpis w opcji POSTROUTING, a także zobaczyć, jakie jeszcze są możliwe.

![foto](https://i.imgur.com/R1S5XBt.png)
