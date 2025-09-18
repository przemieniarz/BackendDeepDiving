Mamy kod który analizujemy
```csharp
var client = new HttpClient();
var task = client.GetStringAsync("http://google.com"); // Krok 1

for (int i = 0; i < 1_000_000_000; i++)
{ /* długa pętla */ } // Krok 2

string page = await task; // Krok 3
Console.WriteLine("Hello world"); // Krok 4
```
Oto co dzieje się "pod spodem":

1. Transformacja w Maszynę Stanów
	Gdy kompilator napotyka metodę oznaczoną jako `async`, generuje dla niej specjalną klasę (lub strukturę), która implementuje maszynę stanów (`state machine`).

	• **Zamiast funkcji mamy obiekt:** Cała metoda `Main` przestaje być zwykłą funkcją, a staje się obiektem przechowywanym w pamięci.

	• **Zmienne lokalne stają się polami:** Wszystkie zmienne lokalne z metody (takie jak `client`, `task`, `page`) stają się polami w tym obiekcie maszyny stanów. Dzięki temu ich wartości są zachowywane pomiędzy kolejnymi wznowieniami działania metody.

	• **Logika trafia do metody** **MoveNext()****:** Cała logika naszej metody jest przenoszona do jednej, centralnej metody w maszynie stanów, zazwyczaj nazywanej `MoveNext()`.
	
2. Podział kodu na części

	Słowo kluczowe `await` dzieli metodę `MoveNext()` na logiczne części, które odpowiadają kolejnym stanom maszyny.

	• **Część 1 (przed** **await****):**

	   ◦ Utworzenie `HttpClient`.
	    ◦ Wywołanie `GetStringAsync("http://google.com")`. Ta operacja jest przekazywana do systemu operacyjnego, który zarządza nią za pomocą sterowników karty sieciowej. Wywołanie to jest nieblokujące – wątek, który je zainicjował, jest natychmiast zwalniany, by mógł kontynuować pracę.
	    ◦ Rozpoczęcie i wykonanie długiej pętli `for`. Wszystko to dzieje się synchronicznie na głównym wątku.

	• **Część 2 (po** **await****):**

	    ◦ Pobranie wyniku z zakończonego zadania `task`.
	    ◦ Wywołanie `Console.WriteLine("Hello world")`.



A teraz o magii:
1. **Rozpoczęcie operacji (Część 1):** Główny wątek aplikacji (nazwijmy go Wątkiem 1) wykonuje pierwszą część kodu. Kiedy wywołuje `GetStringAsync`, przekazuje żądanie do systemu operacyjnego, który z kolei komunikuje się ze sterownikiem sieciowym. Ważne jest to, że operacja I/O (wejścia/wyjścia) jest obsługiwana poza pulą wątków .NET, na poziomie systemu. Wątek 1 nie czeka, tylko od razu przechodzi do wykonania pętli.

2. **Dotarcie do** **await** **(punkt kontrolny):** Po zakończeniu pętli, Wątek 1 dociera do instrukcji `await task`. W tym momencie dzieje się kluczowa rzecz:

    ◦ Sprawdzany jest status obiektu `task`.
    
    ◦ **Scenariusz A (Task nieukończony):** Jeśli operacja sieciowa w tle jeszcze się nie zakończyła (co ma miejsce w przypadku krótkiej pętli), `await` powoduje natychmiastowe zwrócenie sterowania z metody. **Wątek 1 jest zwalniany i wraca do puli wątków (****ThreadPool****), gdzie może być użyty do wykonania innej pracy**. Maszyna stanów "zapamiętuje", w którym miejscu przerwała działanie.
    
    ◦ **Scenariusz B (Task już ukończony):** Jeśli operacja sieciowa zakończyła się, zanim Wątek 1 skończył pętlę (co jest prawdopodobne przy bardzo długiej pętli), `await` stwierdza, że wynik jest już dostępny. **W takim przypadku nie ma potrzeby zawieszania metody i zwalniania wątku**. Wątek 1 po prostu kontynuuje wykonywanie reszty kodu synchronicznie.

3. **Wznowienie wykonania (tylko w Scenariuszu A):**

    ◦ Gdy sterownik sieciowy zakończy pobieranie danych, informuje o tym system operacyjny.
    ◦ System operacyjny przekazuje tę informację do puli wątków .NET (`ThreadPool`).
    ◦ Pula wątków wie, która maszyna stanów czeka na ten wynik. Bierze **dowolny dostępny wątek** (może to być Wątek 1, jeśli akurat jest wolny, ale często jest to inny wątek, np. Wątek 2).
    ◦ Wybrany wątek wywołuje metodę `MoveNext()` na obiekcie maszyny stanów. Dzięki zapisanemu stanowi, maszyna wie, że ma pominąć pierwszą część kodu i od razu przejść do wykonania części drugiej – pobrania wyniku i wypisania "Hello world"





2 przykład
```csharp
var client = new HttpClient();

// Część 1
var task = client.GetStringAsync("http://google.com");
// jakaś pętla lub inna praca synchroniczna
await task;

// Część 2
var task2 = client.GetStringAsync("http://bing.com"); // Drugie wywołanie dla przykładu
await task2;

// Część 3
Console.WriteLine("Hello world");
```
W tym scenariuszu, **dwa słowa kluczowe** **await** **dzielą metodę na trzy logiczne części (lub stany)**:

1. **Część 1 (przed pierwszym** **await****):** Obejmuje utworzenie `HttpClient`, zainicjowanie pierwszego żądania sieciowego (`GetStringAsync`) i wykonanie pętli.

2. **Część 2 (pomiędzy pierwszym a drugim** **await****):** Obejmuje pobranie wyniku z pierwszego zadania i zainicjowanie drugiego żądania sieciowego.

3. **Część 3 (po drugim** **await****):** Obejmuje pobranie wyniku z drugiego zadania i wykonanie finalnej logiki, np. `Console.WriteLine`.

**Transformacja w maszynę stanów:**

    ◦ Kompilator tworzy klasę maszyny stanów, w której wszystkie zmienne lokalne (np. `client`, `task`, `task2`) stają się polami obiektu.

    ◦ Logika wszystkich trzech części zostaje przeniesiona do jednej centralnej metody, `MoveNext()`.

    ◦ Maszyna stanów będzie posiadała dodatkowe pola do przechowywania tzw. "awaiterów" (obiektów oczekujących na zakończenie zadań) dla każdego `await`

Magia:
**Wykonanie** **MoveNext()** **po raz pierwszy (Część 1):**

    ◦ Główny wątek (np. Wątek 1) wywołuje `MoveNext()`.

    ◦ Wewnątrz `MoveNext()` wykonywany jest kod **Części 1**: tworzony jest `HttpClient`, a operacja `GetStringAsync` jest delegowana do systemu operacyjnego.

    ◦ Wątek 1 dochodzi do pierwszego `await`. Sprawdza, czy zadanie (`task`) jest już ukończone.

    ◦ Zakładając, że operacja sieciowa trwa, zadanie nie jest ukończone. Maszyna stanów **ustawia swój wewnętrzny stan na** **0** (lub inny początkowy numer stanu), aby wiedzieć, gdzie wznowić pracę. Następnie metoda `MoveNext()` natychmiast zwraca sterowanie, a **Wątek 1 zostaje zwolniony i wraca do puli wątków** (`ThreadPool`).

**Wykonanie** **MoveNext()** **po raz drugi (Część 2):**

    ◦ Gdy pierwsza operacja sieciowa się zakończy, sterownik sieciowy informuje o tym system operacyjny, który z kolei powiadamia pulę wątków .NET.

    ◦ Pula wątków bierze **dowolny dostępny wątek** (np. Wątek 2) i ponownie wywołuje `MoveNext()` na tym samym obiekcie maszyny stanów.

    ◦ Dzięki zapisanemu stanowi (`0`), maszyna wie, że ma pominąć kod z Części 1 i przejść bezpośrednio do miejsca po pierwszym `await`. Wykorzystuje do tego mechanizmy takie jak instrukcje `goto` w kodzie pośrednim (IL), aby przeskoczyć do odpowiedniej etykiety.

    ◦ Rozpoczyna się wykonywanie kodu **Części 2**: pobierany jest wynik pierwszego zadania i inicjowane jest drugie zadanie asynchroniczne (`task2`).

    ◦ Wątek 2 dochodzi do drugiego `await`. Ponownie sprawdza, czy `task2` jest ukończony.

    ◦ Jeśli nie, maszyna stanów **aktualizuje swój wewnętrzny stan na** **1**, a `MoveNext()` znów zwraca sterowanie. Wątek 2 również zostaje zwolniony.

**Wykonanie** **MoveNext()** **po raz trzeci (Część 3):**

    ◦ Gdy druga operacja sieciowa się zakończy, proces się powtarza: pula wątków bierze **dowolny dostępny wątek** (może to być Wątek 1, 2, lub zupełnie inny, np. Wątek 3) i po raz trzeci wywołuje `MoveNext()`.

    ◦ Maszyna stanów sprawdza swój stan, który teraz wynosi `1`. Dzięki temu wie, że ma pominąć kod z Części 1 i 2 i przejść od razu do kodu po drugim `await`.

    ◦ Wykonywana jest **Część 3**: pobierany jest wynik drugiego zadania, a następnie wywoływane jest `Console.WriteLine("Hello world")`.

    ◦ Po zakończeniu ostatniej części, maszyna stanów ustawia status całego zadania reprezentującego metodę jako zakończony


**Każde dodatkowe słowo kluczowe** **await** **dodaje kolejny potencjalny punkt wstrzymania i wznowienia działania, a co za tym idzie – kolejny stan do maszyny stanów**. Pula wątków (`ThreadPool`) efektywnie zarządza tym procesem, wykorzystując dostępne wątki do "obsługi" kolejnych części kodu, gdy tylko odpowiadające im operacje asynchroniczne (najczęściej I/O) się zakończą. Dzięki temu aplikacja pozostaje responsywna, a wątki nie są blokowane na czas oczekiwania