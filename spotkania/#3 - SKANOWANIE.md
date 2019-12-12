## Skanowanie

### Metasploit i osobny workspace

1. Uruchomienie uslugi bazy danych PostgreSQL

```
sudo service postgresql start
```

2. Uruchomenie Metasploit i nowy workspace

```
db_status
workspace --add new_workspace
workspace new_workspace
```

### Skanowanie do bazy danych

1. Skan otwartych portw

```
db_nmap -F -sS -n --reason --open IP
```
-sS TCP SYN

--reason - wyswietl powod dla ktorego port jest w konkretnym stanie

--open pokazuj tylko otwarte porty


2. Skan wersji oprogramowania usug

```
db_nmap -sS -sV -sC -n -p- IP
```
-sC = --script=default

3. Tabela z informacjami o serwisach

```
services
```

4. Skan podatnosci za pomoca skryptu Nikto

```
nikto -host IP
```

5. Odkrywanie naglowkow
```
curl --head IP
```
