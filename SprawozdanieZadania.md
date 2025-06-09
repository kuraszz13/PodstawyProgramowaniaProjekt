
# Sprawozdanie – Zadania 3–10 (MSYS2)

## Wprowadzenie

Celem zestawu zadań było zastosowanie narzędzi systemu MSYS2 (środowisko bash i zestaw narzędzi Unixowych) do przetwarzania danych tekstowych i graficznych. Wykorzystano m.in. `cat`, `grep`, `awk`, `sed`, `diff`, `patch`, `md5sum`, `magick`, `tr`, `dos2unix`, `unix2dos`.

---

## Zadanie 3 - Niesforne dane (cat + paste + echo)

### Cel

- Podzielić plik na trzy kolumny, tak aby był gotowy do importu do pliku programu Excel
- Dodać nagłowki ("X","Y","Z")

### Wykonanie
```bash
cat dane.txt
paste - - - < dane.txt
echo "x y z" >> dane.txt
```
### Uwagi 

- `cat` wyświetla podany plik dane.txt
- `paste` dzieli tekst na trzy kolumny
- `echo 'x y z"` dodaje nagłowki do kolumn

## Zadanie 4 – Dodawanie poprawek (diff + patch)

### Cel

- Porównać dwa pliki tekstowe `lista.txt` i `lista-pop.txt`.
- Wygenerować różnice jako plik `.patch`.
- Zaaplikować zmiany na pliku `lista.txt`, aby uzyskać `lista-pop.txt`.

### Wykonanie

```bash
diff -u lista.txt lista-pop.txt > lista.patch
patch lista.txt lista.patch
```

### Uwagi

- `diff -u` tworzy różnice w formacie unifikowanym, wymaganym przez `patch`.
- Jeśli pliki są puste, może to oznaczać błędne kodowanie – można użyć `dos2unix`:

```bash
dos2unix lista.txt lista-pop.txt
```

---

## Zadanie 5 – Z CSV do SQL i z powrotem

### Cel

- Wyświetlić unikalne nazwiska z pliku `dane.csv`.
- Posortować je alfabetycznie.

### Rozwiązanie z użyciem `awk`, `sort`, `uniq`, `tail`

```bash
awk -F ';' '{ print $2 }' dane.csv | sort | uniq
dos2unix steps-2sql.csv
tail -n +2 steps-2sql.csv | awk -F';' '{print "INSERT INTo stepsData (time, intensity, steps) VALUES (" $1 ", " $2 ", " $3 ");"}' > stepsData.sql
cat steps-2sql
cat steps-2sql.csv
```


### Objaśnienie

- `-F ';'` ustawia separator pól na średnik.
- `$2` to drugie pole – nazwisko.
- `sort | uniq` usuwa duplikaty i porządkuje wyniki alfabetycznie.

---

## Zadanie 6 – Marudny tłumacz (md5sum)

### Cel

- Sprawdzić, czy pliki `plik1.txt`, `plik2.txt`, itd. nie zostały zmodyfikowane.
- Porównać ich sumy MD5 z oryginalnymi.

### Wykonanie

#### 1. Obliczenie sum kontrolnych:

```bash
md5sum plik1.txt plik2.txt plik3.txt > suma.md5
```

#### 2. Sprawdzenie później:

```bash
md5sum -c suma.md5
```

### Wynik

- `OK` – plik niezmieniony,
- `FAILED` – zawartość pliku się zmieniła.

---

## Zadanie 7 – Fotografik gamoń (konwersja obrazów)

### Cel

- Rozpakować dwa archiwa ze zdjęciami (`kopie-1.zip`, `kopie-2.zip`).
- Skonwertować obrazy do `.jpg`.
- Zmniejszyć ich wysokość do 720 px.
- Ustawić rozdzielczość 96x96 DPI.
- Spakować je do jednego archiwum ZIP.

### Wykorzystane narzędzia

- `unzip`, `cp`, `magick`, `zip`, `bash`.

### Skrypt:

```bash
unzip kopie-1.zip -d kopie1
unzip kopie-2.zip -d kopie2

mkdir all
cp kopie1/* all/
cp kopie2/* all/

mkdir -p output

for file in all/*; do
  if [ -f "$file" ]; then
    name=$(basename "$file")
    name_noext="${name%.*}"
    magick "$file" -resize x720 -density 96x96 "output/${name_noext}.jpg"
  fi
done

zip -j gotowe-fotki.zip output/*
```

### Napotkane problemy

- **"no decode delegate for this image"** – brak obsługi formatu (np. TIFF, HEIC):
  - Rozwiązanie: zainstalowanie pełnej wersji ImageMagick w `msys64`.
- **"[: missing ']'"** – błąd składni `if`:
  - Rozwiązanie: poprawne użycie `if [ -f "$file" ]; then`.

---
