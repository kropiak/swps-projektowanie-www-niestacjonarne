# Projektowanie aplikacji webowych, semestr 2024Z

## Lab 9 - Autentykacja w Django i DRF.
---

### **1. Autentykacja (uwierzytelnianie).**

**Autentykacja** to proces polegający na potwierdzeniu tożsamości (uwierzytelnianie) klienta (aplikacji), który został wcześniej zarejestrowany w danym systemie. **Autoryzacja** to z kolei proces sprawdzenia uprawnień dla zasobów, do których klient próbuje uzyskać dostęp.

W przypadku Django i oraz DRF istnieje kilka sposobów na implementację procesu uwierzytelniania. W tym labie zostaną przedstawione tylko dwie z nich: uwierzytelnianie na podstawie sesji oraz tokena.

>**Oficjalna dokumentacja przydatna przy zadaniach z bieżącego labu:**
>* tutorial: https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/
>* Authentication API Reference: https://www.django-rest-framework.org/api-guide/authentication/
>* Permissions API Reference: https://www.django-rest-framework.org/api-guide/permissions/

Do tej pory dostęp do API nie jest w żaden sposób kontrolowany i każdy znając adres URL może nasze API odpytać, również dla żądań PUT, POST czy DELETE.
Aby dla danego endpointu określić restrykcje polegającą na dostępie tylko dla zalogowanego użytkownika musimy dokonać kilku zmian w dotychczasowym projekcie.

**Krok 1 - przebudowanie endpointu dla klasy Person.**

Przyjmijmy, że żądanie GET będzie możliwe bez restrykcji, a pozostałe będą wymagały uwierzytelnionego użytkownika.
Aktualnie wszystkie metody (PUT, GET, DELETE) są zdefiniowane w jednej funkcji, co utrudnia nam możliwość rozdzielenia uprawnień dla każdego z nich.
Rozdzielimy więc oba widoki. Dla przypomnienia, poniżej oryginalny kod dla endpointów związanych z klasą Person.

**_Listing 1_**
 ```python
from django.shortcuts import render
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .models import Person, Team
from .serializers import PersonSerializer


@api_view(['GET'])
def person_list(request):
    """
    Lista wszystkich obiektów modelu Person.
    """
    if request.method == 'GET':
        persons = Person.objects.all()
        serializer = PersonSerializer(persons, many=True)
        return Response(serializer.data)


@api_view(['GET', 'PUT', 'DELETE'])
def person_detail(request, pk):

    """
    :param request: obiekt DRF Request
    :param pk: id obiektu Person
    :return: Response (with status and/or object/s data)
    """
    try:
        person = Person.objects.get(pk=pk)
    except Person.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    """
    Zwraca pojedynczy obiekt typu Person.
    """
    if request.method == 'GET':
        person = Person.objects.get(pk=pk)
        serializer = PersonSerializer(person)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = PersonSerializer(person, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        person.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
 ```

Rozdzielamy więc metodę GET od reszty i dodajemy brakujące importy oraz dekoratory dla żadań PUT i DELETE.

**_Listing 2_**

```python
from django.shortcuts import render
from rest_framework import status
from rest_framework.decorators import api_view, authentication_classes, permission_classes
from rest_framework.authentication import SessionAuthentication, BasicAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from .models import Person, Team
from .serializers import PersonSerializer


@api_view(['GET'])
def person_list(request):
    """
    Lista wszystkich obiektów modelu Person.
    """
    if request.method == 'GET':
        persons = Person.objects.all()
        serializer = PersonSerializer(persons, many=True)
        return Response(serializer.data)


@api_view(['GET'])
def person_detail(request, pk):

    """
    :param request: obiekt DRF Request
    :param pk: id obiektu Person
    :return: Response (with status and/or object/s data)
    """
    try:
        person = Person.objects.get(pk=pk)
    except Person.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    """
    Zwraca pojedynczy obiekt typu Person.
    """
    if request.method == 'GET':
        person = Person.objects.get(pk=pk)
        serializer = PersonSerializer(person)
        return Response(serializer.data)


@api_view(['PUT', 'DELETE'])
@authentication_classes([SessionAuthentication, BasicAuthentication])
@permission_classes([IsAuthenticated])
def person_update_delete(request, pk):

    """
    :param request: obiekt DRF Request
    :param pk: id obiektu Person
    :return: Response (with status and/or object/s data)
    """
    try:
        person = Person.objects.get(pk=pk)
    except Person.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'PUT':
        serializer = PersonSerializer(person, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        person.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```

Pojawienie się dekoratorów `@authentication_classes([SessionAuthentication, BasicAuthentication])` oraz `@permission_classes([IsAuthenticated])` określa odpowiednio dopuszczalne metody autentykacji oraz listę klas, które definiują wymagane uprawnienia. Bez ich spełnienia kod metody nie zostanie wykonany, a do obiektu `Response` zostanie dołączony odpowiedni kod błędu np.:
```HTTP 403 Forbidden
Allow: OPTIONS, PUT, DELETE
Content-Type: application/json
Vary: Accept

{
    "detail": "Nie podano danych uwierzytelniających."
}
```
Dodatkowo musimy również zaktualizować definicję URL-i dla zmodyfikowanych endpointów. Postać pliku `ankiety/urls.py` po modyfikacji:


**_Listing 3_**
```python
from django.urls import path, include
from . import views

urlpatterns = [
    path('persons/', views.person_list),
    path('persons/<int:pk>/', views.person_detail),
    path('persons/update/<int:pk>/', views.person_update_delete),
    path('persons/delete/<int:pk>/', views.person_update_delete),
]
```
Docelowo niezłym pomysłem jest budowanie oddzielnego endpointu dla każdej operacji, co zmniejszy ilość refactoringu, który będzie trzeba wykonać w przyszłości jeżeli pojawi się potrzeba dodania uprawnień dla każdego z nich z osobna.

Po przetestowaniu tych dwóch endpointów powinniśmy móc wyświetlać obiekty typu Person, ale bez uwierzytelnienia nie możemy ich modyfikować ani usuwać.

**Krok 2 - dodanie możliwości zalogowania się poprzez interfejs API dostarczony przez DRF.**

Do pliku `urls.py` w projekcie dodajemy:

**_Listing 4_**
```python
urlpatterns += [
    path('api-auth/', include('rest_framework.urls')),
]
```

Po odświeżeniu widoku DRF API pojawi się możliwość zalogowania z wykorzystaniem wcześniej utworzonych kont użytkowników. Po uwierzytelnieniu otrzymamy dostęp do wszystkich endpointów, które wymagają uprawnienia `IsAuthenticated`.


### **Autentykacja poprzez token**

**Autentyfikacja przez token w Django Rest Framework (DRF)** polega na przyznaniu użytkownikowi unikalnego tokena, który jest przesyłany w nagłówkach każdej żądanej operacji. Dzięki temu użytkownik może się uwierzytelnić bez konieczności wysyłania swoich danych logowania w każdym żądaniu.

Żądanie HTTP składa się z kilku kluczowych elementów, które pozwalają klientowi komunikować się z serwerem. Oto ich struktura:

#### 1. **Linia początkowa (Request Line)**  
Zawiera podstawowe informacje o żądaniu:
- **Metoda HTTP**: Określa typ operacji (np. `GET`, `POST`, `PUT`, `DELETE`).
- **Ścieżka URI**: Wskazuje zasób na serwerze (np. `/api/resource`).
- **Wersja protokołu HTTP**: Najczęściej `HTTP/1.1` lub `HTTP/2`.

**Przykład**:  
```
GET /api/resource HTTP/1.1
```

#### 2. **Nagłówki HTTP (Headers)**  
Metadane opisujące żądanie. Kluczowe nagłówki to:
- **Host**: Adres serwera (np. `Host: www.example.com`).
- **Authorization**: Dane uwierzytelniające (np. `Authorization: Token abc123`).
- **Content-Type**: Typ danych w treści żądania (np. `application/json`).
- **Accept**: Preferowany format odpowiedzi (np. `application/json`).
- **User-Agent**: Informacja o kliencie (np. przeglądarka lub aplikacja).

**Przykład nagłówków**:
```
Host: www.example.com
Authorization: Token abc123
Content-Type: application/json
User-Agent: curl/7.64.1
```

#### 3. **Treść żądania (Body)**  
Obecna tylko w żądaniach takich jak `POST` lub `PUT`. Zawiera dane przesyłane do serwera, np. formularze, JSON, pliki.  
**Przykład treści w formacie JSON**:
```json
{
    "name": "John",
    "email": "john@example.com"
}
```

#### 4. **Pusta linia końcowa**  
Oddziela nagłówki od treści żądania. W żądaniach bez treści jest to po prostu końcowy znak nowej linii.

#### Przykładowe pełne żądanie HTTP:
```
POST /api/resource HTTP/1.1
Host: www.example.com
Authorization: Token abc123
Content-Type: application/json
User-Agent: curl/7.64.1

{
    "name": "John",
    "email": "john@example.com"
}
```

### Jak to działa token DRF:

1. **Tworzenie tokena**: Po pomyślnym zalogowaniu użytkownik otrzymuje token, który jest generowany przez serwer.
2. **Przesyłanie tokena**: Klient przesyła token w nagłówku `Authorization: Token <token>` w każdej kolejnej żądanej operacji.
3. **Weryfikacja tokena**: Serwer sprawdza poprawność tokena i identyfikuje użytkownika.

---

### Instrukcja krok po kroku

#### 1. **Instalacja i konfiguracja Django Rest Framework**
Upewnij się, że masz zainstalowane Django i DRF:
```bash
pip install django djangorestframework
```

Jeżeli nadal nie masz to dodaj `rest_framework` do zmiennej `INSTALLED_APPS` w `settings.py`:
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
]
```

Uruchom migracje, aby utworzyć odpowiednie tabele w bazie danych:
```bash
python manage.py migrate
```


#### 2. **Aktywacja autentyfikacji przez token**
W `views.py` zmień `@authentication_classes([SessionAuthentication, BasicAuthentication])` na `@authentication_classes([SessionAuthentication, TokenAuthentication])`:
```python
@api_view(['GET'])
@authentication_classes([SessionAuthentication, TokenAuthentication])
@permission_classes([IsAuthenticated])
def person_detail(request, pk):
```

> !!! Uwaga !!! </br>
> Jeżeli chcesz ustawić domyślną autentyfikację dla całego projektu, wtedy dodaj taki kod do pliku `settings.py` zmieniając w miejscu `DEFAULT_AUTHENTICATION_CLASSES` sposób autentyfikacji (Username & Password: `BasicAuthentication`, Token: `TokenAuthentication`), a dekoratora `@authentication_classes` używaj jedynie do widoków, które mają mieć inną autentyfikację:
>```python
>REST_FRAMEWORK = {
>    'DEFAULT_AUTHENTICATION_CLASSES': [
>        'rest_framework.authentication.TokenAuthentication',
>    ],
>    'DEFAULT_PERMISSION_CLASSES': [
>        'rest_framework.permissions.IsAuthenticated',
>    ],
>}
>```
> Dodatkowo warto wiedzieć, że przy globalnej domyślnej autentyfikacji, aby wyłączyć ją dla danego endpointa (czyli, aby również niezalogowani mogli się do niego dostać) nalezy ustawić wartość dla `permission_classes` jako `AllowAny` na przykład korzystając z dekoratora:
> ```python
> @permission_classes([AllowAny])
> def person_list(request):
> ```
> lub ze zmiennej w środku klasy jeżeli używamy widoku klasowego zamiast funkcyjnego, na przykład:
> ```python
> class PublicEndpointView(APIView):
>    permission_classes = [AllowAny]
>
>    def get(self, request):
>        return Response({"message": "This is a public endpoint, no authentication required!"})
> ```


#### 3. **Generowanie tokenów dla użytkowników**
W pliku `urls.py` dodaj widok generowania tokenów:
```python
from django.urls import path
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    ...
    path('api-token-auth/', obtain_auth_token, name='api_token_auth'),
]
```

Po wdrożeniu możesz uzyskać token dla użytkownika, wysyłając żądanie `POST` na `/api-token-auth/` z danymi logowania na przykład używając polecenia curl:
```bash
curl -X POST -d "username=admin&password=admin" http://127.0.0.1:8000/api-token-auth/
```

bądź logując się w przeglądarce używając stworzonego endpointa z **metodą POST**.

#### 4. **Dodanie tokenów do nagłówków**
Przy każdym żądaniu API, prześlij token w nagłówku `Authorization`:
```bash
curl -H "Authorization: Token <twój_token>" http://127.0.0.1:8000/api/some-endpoint/
```

Niestety nie ma prostego spsobu, żeby dodać ręcznie taki token do pamięci przeglądarki. Można to zrobić w Postmanie, co pokażemy później.

#### 5. **Przykład API z ochroną przez token**
W `views.py` utwórz widok chroniony przez autentyfikację:
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

class ProtectedView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"message": "Hello, authenticated user!"})
```

Dodaj trasę w `urls.py`:
```python
from django.urls import path
from .views import ProtectedView

urlpatterns = [
    ...
    path('protected/', ProtectedView.as_view(), name='protected'),
]
```

#### 6. **Testowanie**
1. Zaloguj się, aby uzyskać token.
2. Przetestuj chronione API, wysyłając żądanie z tokenem.


Autentykacja poprzez token wymaga dodania odpowiedniego atrybutu w nagłówku żądania. W zależności od klienta do odpytywania API, z którego korzystamy sam sposób przekazania może się różnić. 

W przypadku narzędzia `http` wiersza poleceń (Mac) może to wyglądać tak:
```bash
http http://127.0.0.1:8000/hello/ 'Authorization: Token 9054f7aa9305e012b3c2300408c3dfdf390fcddf'

# lub
http http://127.0.0.1:8000/hello/ 'Authorization: Bearer 9054f7aa9305e012b3c2300408c3dfdf390fcddf'

```

W przypadku polecenia `curl`:
```bash
curl -H "Authorization: Token 9054f7aa9305e012b3c2300408c3dfdf390fcddf" http://127.0.0.1:8000/hello/ 

# lub

curl -H "Authorization: Bearer 9054f7aa9305e012b3c2300408c3dfdf390fcddf" http://127.0.0.1:8000/hello/ 
```
 Z poziomu czystego Pythona:
 ```python
 import requests

url = 'http://127.0.0.1:8000/hello/'
# poniżej też może nastąpić konieczność zamiany słowa Token na Bearer
headers = {'Authorization': 'Token 9054f7aa9305e012b3c2300408c3dfdf390fcddf'}
r = requests.get(url, headers=headers)
```

A w programie Postman należy przejść do zakładki `Authorization`, wybrać `Bearer Token` i podać jego wartość jak na poniższym zrzucie ekranu.

![](postman_auth.png)

Postman narzuca też własną nazwę tokena co powoduje, że nagłówek żądania wygląda tak:
```python
headers = {'Authorization': 'Bearer 9054f7aa9305e012b3c2300408c3dfdf390fcddf'}
```

Jeżeli spowoduje to błąd, można to obejść na dwa sposoby:
1. Poprzez prostą własną implementację klasy `TokenAuthentication`:

**_Listing_ 3**
```python
class BearerTokenAuthentication(TokenAuthentication):
    keyword = u"Bearer"
```

A następnie importujemy ją w widokach i zamieniamy nazwę `TokenAuthentication` na `BearerTokenAuthentication`. Jednak to będzie wymuszało podawanie teraz nagłówka jako `'Authorization': 'Bearer 9054f7aa9305e012b3c2300408c3dfdf390fcddf'` a nie `'Authorization': 'Token 9054f7aa9305e012b3c2300408c3dfdf390fcddf'`.

2. Poprzez wyłączenie w Postmanie autoryzacji (*Type: No Auth*):

![](postman_token_auth_1.PNG)

 a następnie ręczne dodanie nagłówka `Authorization` z wartością `Token 9054f7aa9305e012b3c2300408c3dfdf390fcddf`:

![](postman_token_auth_2.PNG)

Dodatkowo, dla widoków (endpointów), które będą uwierzytelniane poprzez token nie możemy używać jednocześnie kontroli uprawnienia `IsAuthenticated`.

**Zadania**

1. W aplikacji rozwijanej w trakcie zajęć odtwórz opisany powyżej sposób uwierzytelniania (listing 4 i uwierzytelnianie przez wcześniej utworzone konto użytkownika Django np. superusera) i go przetestuj.
2. Do klasy `Osoba` dodaj pole `wlasciciel`, które będzie odwoływało się do modelu `User` wbudowanego w Django Auth. Ogranicz możliwość wyświetlania rekordów tylko ich właścicielom (na widoku wyświetlającym wszystkie obiekty typu `Osoba`). Przetestuj z użyciem dwóch różnych kont użytkowników. W tutorialu (https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/#tutorial-4-authentication-permissions) przedstawiony jest taki scenariusz z przykładami. Nie wystawiaj jednak API dla modelu `User` jeżeli nie jest to konieczne.

> Wskazówka: Jeżeli dostęp do endpointu, który zapisuje nowe obiekty `Osoba` został zadeklarowany przez widok funkcyjny (a nie poprzez dziedziczenie z klasy `APIView`) to dodanie obiektu aktualnie zalogowanego usera do danych z żądania POST można zrealizować tak:
```python
...
    if request.method == 'POST':
        serializer = PersonSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(wlasciciel=request.user)
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```
3. Korzystając z dokumentacji zawartej w https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication dodaj obsługę tokenów w aplikacji. Dodaj również tokeny dla istniejących userów.
4. Rozbij endpoint dla person na oddzielne punkty dla żądania typu PUT oraz DELETE i dla tego drugiego ustaw autentykację poprzez token. Przetestuj działanie za pomocą narzędzia curl lub Postman.
5. Dodaj lub zmodyfikuj istniejący endpoint tak aby możliwe było wyświetlenie wszystkich obiektów typu `Osoba` przypisanych do danego stanowiska. Niech url dla niego będzie postaci `.\stanowisko\<id>\members\`. Niech dostęp będzie tylko do odczytu i tylko poprzez autoryzację tokenem.
