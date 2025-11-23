# Projektowanie aplikacji webowych, semestr 2025Z

## Lab 4
---
### **1. Więcej możliwości modeli w ekosystemie Django.**


Oprócz typowego mapowania obiektowo-relacyjnego klasy modeli frameworka Django oferują dużo więcej możliwości, m.in. implementację funkcji pomocniczych, określenie domyślnego porządku sortowania zwracanych krotek, określenie parametrów walidacji i inne. Poniżej zostaną zaprezentowane niektóre z możliwości wraz z przykładami użycia.

Przydatne linki do dokumentacji, które mogą się przydać w trakcie rozwiązywania zadań:
1. Lista dostępnych pól klas modeli: https://docs.djangoproject.com/pl/5.2/ref/models/fields/#model-field-types

W Django modele definiują strukturę tabel w bazie danych, a typy pól (field types) określają, jakie dane mogą być przechowywane w tych tabelach. Poniżej znajduje się lista najpopularniejszych typów pól w Django, wraz z przykładami zastosowań i sposobem ich użycia:

### 1. **CharField**
   Przechowuje tekst o ograniczonej długości.
   - **Przykład**: Imię, nazwisko, tytuł książki.
   - **Użycie**:
     ```python
     from django.db import models

     class Person(models.Model):
         first_name = models.CharField(max_length=50)
         last_name = models.CharField(max_length=50)
     ```

### 2. **TextField**
   Przechowuje długi tekst, bez limitu długości (przydatne dla treści jak opisy, artykuły).
   - **Przykład**: Opis produktu, artykuł na blogu.
   - **Użycie**:
     ```python
     class Article(models.Model):
         title = models.CharField(max_length=100)
         content = models.TextField()
     ```

### 3. **IntegerField**
   Przechowuje liczby całkowite.
   - **Przykład**: Wiek, liczba przedmiotów.
   - **Użycie**:
     ```python
     class Product(models.Model):
         name = models.CharField(max_length=100)
         stock_quantity = models.IntegerField()
     ```

### 4. **FloatField**
   Przechowuje liczby zmiennoprzecinkowe.
   - **Przykład**: Cena produktu, współrzędne geograficzne.
   - **Użycie**:
     ```python
     class Product(models.Model):
         name = models.CharField(max_length=100)
         price = models.FloatField()
     ```

### 5. **BooleanField**
   Przechowuje wartość logiczną (`True` lub `False`).
   - **Przykład**: Czy produkt jest dostępny.
   - **Użycie**:
     ```python
     class Product(models.Model):
         name = models.CharField(max_length=100)
         is_available = models.BooleanField(default=True)
     ```

### 6. **DateField**
   Przechowuje daty.
   - **Przykład**: Data urodzenia, data publikacji.
   - **Użycie**:
     ```python
     class Event(models.Model):
         event_name = models.CharField(max_length=100)
         event_date = models.DateField()
     ```

### 7. **DateTimeField**
   Przechowuje daty i godziny.
   - **Przykład**: Data i czas rejestracji użytkownika, czas utworzenia posta.
   - **Użycie**:
     ```python
     class Post(models.Model):
         title = models.CharField(max_length=100)
         created_at = models.DateTimeField(auto_now_add=True)
     ```

### 8. **EmailField**
   Specjalizacja `CharField` przechowująca adresy email.
   - **Przykład**: Email użytkownika.
   - **Użycie**:
     ```python
     class User(models.Model):
         name = models.CharField(max_length=100)
         email = models.EmailField(unique=True)
     ```

### 9. **URLField**
   Przechowuje adresy URL.
   - **Przykład**: Strona internetowa firmy.
   - **Użycie**:
     ```python
     class Company(models.Model):
         name = models.CharField(max_length=100)
         website = models.URLField()
     ```

### 10. **ForeignKey**
   Tworzy relację jeden-do-wielu (One-to-Many) z inną tabelą.
   - **Przykład**: Posty powiązane z użytkownikiem.
   - **Użycie**:
     ```python
     class User(models.Model):
         name = models.CharField(max_length=100)

     class Post(models.Model):
         title = models.CharField(max_length=100)
         content = models.TextField()
         author = models.ForeignKey(User, on_delete=models.CASCADE)
     ```

### 11. **ManyToManyField**
   Tworzy relację wiele-do-wielu (Many-to-Many) z inną tabelą.
   - **Przykład**: Grupy użytkowników w systemie.
   - **Użycie**:
     ```python
     class Group(models.Model):
         name = models.CharField(max_length=100)

     class User(models.Model):
         name = models.CharField(max_length=100)
         groups = models.ManyToManyField(Group)
     ```

### 12. **OneToOneField**
   Tworzy relację jeden-do-jednego (One-to-One) z inną tabelą.
   - **Przykład**: Profil użytkownika.
   - **Użycie**:
     ```python
     class User(models.Model):
         name = models.CharField(max_length=100)

     class Profile(models.Model):
         user = models.OneToOneField(User, on_delete=models.CASCADE)
         bio = models.TextField()
     ```

### 13. **DecimalField**
   Przechowuje liczby dziesiętne z określoną precyzją (przydatne dla walut, wartości finansowych).
   - **Przykład**: Cena produktu.
   - **Użycie**:
     ```python
     class Product(models.Model):
         name = models.CharField(max_length=100)
         price = models.DecimalField(max_digits=10, decimal_places=2)
     ```

### 14. **SlugField**
   Przechowuje krótkie etykiety (slug), często używane w adresach URL.
   - **Przykład**: Etykieta dla artykułu.
   - **Użycie**:
     ```python
     class Article(models.Model):
         title = models.CharField(max_length=100)
         slug = models.SlugField(unique=True)
     ```

### 15. **FileField**
   Przechowuje ścieżkę do pliku (np. dokumentu lub obrazu).
   - **Przykład**: Załączniki do wiadomości, zdjęcia użytkowników.
   - **Użycie**:
     ```python
     class Document(models.Model):
         title = models.CharField(max_length=100)
         file = models.FileField(upload_to='documents/')
     ```

### 16. **ImageField**
   Specjalizacja `FileField` dla plików graficznych.
   - **Przykład**: Awatar użytkownika.
   - **Użycie**:
     ```python
     class UserProfile(models.Model):
         user = models.OneToOneField(User, on_delete=models.CASCADE)
         avatar = models.ImageField(upload_to='avatars/')
     ```

### 17. **TimeField**
   Przechowuje czas (bez daty).
   - **Przykład**: Czas rozpoczęcia spotkania.
   - **Użycie**:
     ```python
     class Meeting(models.Model):
         title = models.CharField(max_length=100)
         start_time = models.TimeField()
     ```

### 18. **DurationField**
   Przechowuje czas trwania.
   - **Przykład**: Czas trwania filmu, sesji.
   - **Użycie**:
     ```python
     class Movie(models.Model):
         title = models.CharField(max_length=100)
         duration = models.DurationField()
     ```

Każde pole w modelu jest przetwarzane i walidowane automatycznie przez Django ORM, co znacznie upraszcza pracę z danymi. Można dodatkowo używać atrybutów takich jak `null=True`, `blank=True`, `default=...`, aby dostosować sposób przechowywania danych w bazie.

2. Atrybuty modeli: https://docs.djangoproject.com/pl/5.2/topics/db/models/#field-options

Atrybuty modeli w Django definiują różne aspekty zachowania pól i modeli w interakcji z bazą danych oraz walidacją. Oto najczęściej używane atrybuty modeli Django, ich przykłady i zastosowanie:

### 1. **`null`**
   - Określa, czy pole może przechowywać wartość `NULL` w bazie danych.
   - **Typ pól**: Wszystkie pola (oprócz `ManyToManyField`, `OneToOneField` i `ForeignKey`).
   - **Domyślna wartość**: `False` (pole musi mieć wartość).
   - **Przykład**:
     ```python
     class Person(models.Model):
         name = models.CharField(max_length=100, null=True)
     ```

### 2. **`blank`**
   - Określa, czy pole może być puste podczas walidacji formularzy (dotyczy walidacji na poziomie formularzy, a nie bazy danych).
   - **Typ pól**: Wszystkie pola.
   - **Domyślna wartość**: `False` (pole nie może być puste w formularzach).
   - **Przykład**:
     ```python
     class Person(models.Model):
         email = models.EmailField(blank=True)
     ```

#### Jaka jest różnica?

![Tu powinien być obrazek :)](./blank_vs_null.png)


### 3. **`default`**
   - Ustala domyślną wartość dla pola, jeśli użytkownik jej nie poda.
   - **Typ pól**: Wszystkie pola.
   - **Przykład**:
     ```python
     class Product(models.Model):
         price = models.DecimalField(max_digits=10, decimal_places=2, default=0.00)
     ```

### 4. **`unique`**
   - Sprawia, że wartość w danym polu musi być unikalna w całej tabeli.
   - **Typ pól**: Wszystkie pola (oprócz pól relacyjnych jak `ManyToManyField`).
   - **Przykład**:
     ```python
     class User(models.Model):
         username = models.CharField(max_length=50, unique=True)
     ```

### 5. **`choices`**
   - Definiuje listę dostępnych wartości dla pola (jako pary: wyświetlana wartość i wewnętrzna wartość).
   - **Typ pól**: Najczęściej `CharField`, `IntegerField`, itp.
   - **Przykład**:
     ```python
     class Person(models.Model):
         STATUS_CHOICES = [
             ('A', 'Active'),
             ('I', 'Inactive'),
         ]
         status = models.CharField(max_length=1, choices=STATUS_CHOICES, default='A')
     ```

### 6. **`primary_key`**
   - Określa, czy pole jest kluczem głównym (zastępuje automatyczne tworzenie pola `id`).
   - **Typ pól**: Zazwyczaj `IntegerField` lub `CharField`.
   - **Przykład**:
     ```python
     class Person(models.Model):
         ssn = models.CharField(max_length=9, primary_key=True)
     ```

### 7. **`db_index`**
   - Tworzy indeks na kolumnie, co przyspiesza operacje wyszukiwania.
   - **Typ pól**: Wszystkie pola.
   - **Przykład**:
     ```python
     class Product(models.Model):
         sku = models.CharField(max_length=30, db_index=True)
     ```

### 8. **`verbose_name`**
   - Określa bardziej opisową nazwę pola, wyświetlaną w panelu administracyjnym Django.
   - **Typ pól**: Wszystkie pola.
   - **Przykład**:
     ```python
     class Person(models.Model):
         first_name = models.CharField(max_length=50, verbose_name="First Name")
     ```

### 9. **`verbose_name_plural`**
   - Definiuje liczbę mnogą nazwy modelu, wyświetlaną w panelu administracyjnym.
   - **Typ modeli**: Wszystkie modele.
   - **Przykład**:
     ```python
     class Person(models.Model):
         name = models.CharField(max_length=100)

         class Meta:
             verbose_name_plural = "People"
     ```

### 10. **`help_text`**
   - Dodaje pomocniczy tekst wyświetlany w formularzach, opisujący pole.
   - **Typ pól**: Wszystkie pola.
   - **Przykład**:
     ```python
     class Product(models.Model):
         price = models.DecimalField(max_digits=10, decimal_places=2, help_text="Enter price in USD")
     ```

### 11. **`auto_now`**
   - Automatycznie ustawia pole na bieżący czas i datę przy każdej aktualizacji obiektu.
   - **Typ pól**: `DateField`, `DateTimeField`.
   - **Przykład**:
     ```python
     class Post(models.Model):
         updated_at = models.DateTimeField(auto_now=True)
     ```

### 12. **`auto_now_add`**
   - Automatycznie ustawia pole na bieżący czas i datę przy tworzeniu obiektu.
   - **Typ pól**: `DateField`, `DateTimeField`.
   - **Przykład**:
     ```python
     class Post(models.Model):
         created_at = models.DateTimeField(auto_now_add=True)
     ```

### 13. **`on_delete`**
   - Określa zachowanie relacyjnych pól (np. `ForeignKey`) przy usunięciu obiektu, do którego odnosimy się.
   - **Typ pól**: `ForeignKey`, `OneToOneField`.
   - **Przykład**:
     ```python
     class Post(models.Model):
         author = models.ForeignKey(User, on_delete=models.CASCADE)
     ```

   - Najczęściej używane wartości dla `on_delete`:
     - `CASCADE`: Usuwa powiązane obiekty.
     - `SET_NULL`: Ustawia `NULL` (pole musi mieć `null=True`).
     - `PROTECT`: Blokuje usunięcie powiązanego obiektu.
     - `SET_DEFAULT`: Ustawia wartość domyślną (pole musi mieć `default`).
     - `DO_NOTHING`: Nie wykonuje żadnej akcji.

### 14. **`related_name`**
   - Określa nazwę odwrotnej relacji dla pól relacyjnych (`ForeignKey`, `ManyToManyField`, `OneToOneField`).
   - **Typ pól**: `ForeignKey`, `OneToOneField`, `ManyToManyField`.
   - **Przykład**:
     ```python
     class Book(models.Model):
         title = models.CharField(max_length=100)

     class Author(models.Model):
         name = models.CharField(max_length=100)
         books = models.ManyToManyField(Book, related_name='authors')
     ```

### 15. **`unique_together`**
   - Definiuje zestaw pól, które muszą być unikalne razem (np. kombinacja pól).
   - **Typ modeli**: Meta klasy modeli.
   - **Przykład**:
     ```python
     class Order(models.Model):
         customer = models.ForeignKey(User, on_delete=models.CASCADE)
         order_date = models.DateField()

         class Meta:
             unique_together = ('customer', 'order_date')
     ```

### 16. **`ordering`**
   - Określa domyślne sortowanie rekordów w zapytaniach do bazy danych.
   - **Typ modeli**: Meta klasy modeli.
   - **Przykład**:
     ```python
     class Product(models.Model):
         name = models.CharField(max_length=100)
         price = models.DecimalField(max_digits=10, decimal_places=2)

         class Meta:
             ordering = ['price']
     ```

### 17. **`editable`**
   - Określa, czy pole może być edytowane w formularzach Django (np. panelu administracyjnym).
   - **Typ pól**: Wszystkie pola.
   - **Przykład**:
     ```python
     class Product(models.Model):
         sku = models.CharField(max_length=30, editable=False)
     ```

### 18. **`limit_choices_to`**
   - Ogranicza wybory dla pól relacyjnych, np. `ForeignKey` lub `ManyToManyField`, na podstawie filtrów.
   - **Typ pól**: `ForeignKey`, `ManyToManyField`.
   - **Przykład**:
     ```python
     class ActiveManager(models.Manager):
         def get_queryset(self):
             return super().get_queryset().filter(is_active=True)

     class Product(models.Model):
         name = models.CharField(max_length=100)
         manager = models.ForeignKey('self', limit_choices_to={'is_active': True}, on_delete=models.CASCADE)
     ```

Każdy atrybut pomaga dostosować sposób, w jaki pola modeli są przechowywane, walidowane i prezentowane w formularzach lub administracji Django. Kombinacja tych atrybutów pozwala na szczegółowe definiowanie zachowań aplikacji.

#### **1.1 Rozbudowa naszej aplikacji** 

Oprócz typowego mapowania obiektowo-relacyjnego klasy modeli frameworka Django oferują dużo więcej możliwości, m.in. implementację funkcji pomocniczych, określenie domyślnego porządku sortowania zwracanych krotek, określenie parametrów walidacji i inne. Poniżej zostaną zaprezentowane niektóre z możliwości wraz z przykładami użycia.

Przydatne linki do dokumentacji, które mogą się przydać w trakcie rozwiązywania zadań:
1. Lista dostępnych pól klas modeli: https://docs.djangoproject.com/pl/5.2/ref/models/fields/#model-field-types
2. Atrybuty modeli: https://docs.djangoproject.com/pl/5.2/topics/db/models/#field-options
3. Klasa QuerySet i dostępne metody: https://docs.djangoproject.com/pl/5.2/ref/models/querysets/

### Zadania

**Zadanie 0**  
Rozpocznij pracę na nowym branchu swojego repozytorium.

**Zadanie 1**  
Dodaj do aplikacji `posts` nowy model `Post` zgodnie z poniższą definicją:
* pole `title` typu krótki tekst, max. 150 znaków,
* pole `text` z długim tekstem,
* pole `topic`, klucz obcy do modelu `Topic`, usuwanie kaskadowe,
* pole `slug` typu `SlugField`,
* pole `created_at` typu data i czas, wartość wstawiana automatycznie w momencie dodania rekordu,
* pole `updated_at` typu data i czas, wartość aktualizowana po każdej aktualizacji instancji obiektu,
* pole `created_by`, które jest kluczem obcym do wbudowanej klasy `django.contrib.auth.models.User`.

**Zadanie 2**  
Zarejestruj model w panelu administracyjnym Django. Znajdź w dokumentacji modułu admin ([link](https://docs.djangoproject.com/pl/5.2/ref/contrib/admin/)) jak deklaruje się pola modelu tylko do odczytu (własność `readonly_fields`) i ustaw taką własność dla pola `created_at` modelu `Post`. Sprawdź działanie w panelu administracyjnym.

---

W panelu administracyjnym dla dodanych modeli nie są widoczne wszystkie pola na liście wszystkich obiektów lecz tylko jedno z nich (domyślnie id lub to co w przesłoniętej funkcji `__str__()`). Aby to zrobić należy najpierw zdefiniować w pliku `admin.py` klasę admin dla danego modelu (ModelAdmin), a następnie zdefiniować listę pól do wyświetlenia w zmiennej `list_display`. Przykład poniżej:

**_Listing 1_**
Plik `admin.py`.
```python
# dla przykładowej klasy
class PersonAdmin(admin.ModelAdmin):
    # zmienna list_display przechowuje listę pól, które mają się wyświetlać w widoku listy danego modelu w panelu administracyjnym
    list_display = ['full_name', 'shirt_size']

    # poprzez definicję metody w klasie admin można dodać dodatkowe pola do wyświetlenia
    # zakładamy, że model Person ma pola firstname oraz lastname
    @admin.display(description='Full Name')
    def full_name(self, obj):
        return f"{obj.firstname} {obj.lastname}"

# ten obiekt też trzeba zarejestrować w module admin
admin.site.register(Person, PersonAdmin)
```
---

**Zadanie 3**  
Przesłoń metodę `__str__()` zdefiniowanych modeli `Category`, `Topic` oraz `Post` według poniższej instrukcji:
* `Category` - nazwa kategorii,
* `Topic` - nazwa tematu,
* `Post` - pierwsze 5 wyrazów tekstu posta + '...' jeżeli dłuższy.*

\* Jeżeli dodamy klasę PostAdmin w `admin.py` oraz określimy listę kolumn do wyświetlenia, należy to obsłużyć jako dodatkowe pole w klasie `PostAdmin` (definiowane przez osadzenie metody w tej klasie, patrz **_Listing 1_** powyżej).

**Zadanie 4**  
Bazując na przykładzie z dokumentacji https://docs.djangoproject.com/pl/5.2/topics/db/models/#meta-options dodaj właściwość `META` sortując domyślnie model `Category` po nazwie alfabetycznie, `Topic` podobnie, a `Post` po dacie dodania od najnowszych.


**Zadanie 5**  
Dodaj klasy `ModelAdmin` dla wszystkich zdefiniowanych modeli i zmień listę wyświetlanych kolumn w panelu administracyjnym.

**Zadanie 6**  
Zmodyfikuj kod aplikacji tak, żeby na liście modeli `Post` w panelu administracyjnym wyświetlana została również kolumna `Topic` o postaci 'nazwa topiku (nazwa kategorii)' np. `Batman (Filmy)`.  
Podpowiedź: sprawdź w dokumentacji modułu admin opis działania adnotacji `@admin.display` ([link](https://docs.djangoproject.com/pl/5.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display)) lub zajrzyj do przykładu w **_Listing 1_** powyżej.

---
W panelu administracyjnym możliwe jest również dodanie filtrów do widoków. Cały proces polega na dodaniu pola `filter_list` w klasie admin danego modułu:

**_Listing 2_**
```python
# przykład i wizualizacja dla przykładowego modeli Person oraz Team.

class Admin(admin.ModelAdmin):
    list_filter = ['team']
```
![filtry](filters.png)


W zależności od typu pola zostanie wyświetlony odpowiedni filtr. W przypadku niektórych rodzajów i dużej liczby unikalnych wartości używanie filtrów może być niepraktyczne ze względów wydajnościowych i wizualnych.

---

**Zadanie 7**  
Dodaj filtr dla tematów (topic), kategorii oraz użytkowników dla klasy `Post` w panelu administracyjnym.  Dodaj filtry nazw dla tematów oraz kategorii. Przetestuj działanie filtrów.

**Zadanie 8**  
Przeczytaj informację o możliwości automatycznego generowania wartości pola w dokumentacji(https://docs.djangoproject.com/pl/5.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.prepopulated_fields) i zastosuj to dla pola `SlugField` modelu `Post`, które będzie generowane z pola `title`.

**Zadanie 9**  
Jeżeli nie robiłeś/-aś tego wcześniej to zatwierdź wszystkie zmiany w danym branchu i spróbuj wykonać merge z główną gałęzią. Proponuję wykonać tę operację przy pomocy interfejsu IDE PyCharm, aby przetestować wbudowane narzędzie do wspomagania procesu merge (o ile wystąpią konflikty).