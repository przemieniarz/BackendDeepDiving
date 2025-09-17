Powiedzmy, że chcemy ugotować herbatę. Możnaby w tym celu napisać dwie funkcje:

```csharp
public string MakeTea()
{
	var water = BoilWater();
	
	"take the cups out".Dump();
	"put tea in cups".Dump();
	var tea = $"pour water in cups".Dump();
	
	return tea;
}

public string BoilWater()
{
	"start the kettle".Dump();
	"waiting".Dump();
	Task.Delay(2000).GetAwaiter().GetResult(); //czekamy 2s
	"kettle finished".Dump();
	
	return "hot water";
}
```
w ten sposób wynik będzie podobny jak na obrazku:
![[Pasted image 20250917202133.png]]
Ważne jest tutaj to, że czekamy na czajnik 2 sekundy i nic w tym czasie nie robimy. No oczywiście 2s to nie tak długo, ale to przykładowy czas. Normalnie w czasie gdy czajnik gotuje wodę wyjęlibyśmy kubki itp, bo czajnik jest tutaj elementem `zewnętrznym`.

W związku z tym możemy wprowadzić tutaj asynchroniczność
```csharp
public async Task<string> MakeTeaAsync()
{
	var boilingWater = await BoilWaterAsync();
	
	"take the cups out".Dump();
	"put tea in cups".Dump();
	
	var water = await boilingWater;
	var tea = $"pour water in cups".Dump();
	
	return tea;
}

public async Task<string> BoilWaterAsync()
{
	"start the kettle".Dump();
	"waiting".Dump();
	await Task.Delay(2000); 
	"kettle finished".Dump();
	
	return "hot water";
}
```
Pierwsza linijka w kodzie działa tak, że odpalamy tak jakby ten czajnik czyli metodę. Ona sobie działa na jakimś wątku, ale my za bardzo się tym nie przejmujemy i robimy inne operacje. Ale gdy już faktyczne potrzebujemy tej zagrzanej wody to awaitujemy tę operacje. Output wyglądałby tak:
![[Pasted image 20250917202908.png]]
czyli nastawiamy czajnik i w międzyczasie ogarniamy elementy z funkcji MakeTeaAsync - wyciągnięcie kubków, wrzucenie herbaty.

Mamy tutaj trzy komponenty do zrozumienia:
* Task - obiekt  działa jak "most" lub "obietnica" przyszłego wyniku. Reprezentuje operację, która jest w toku i w przyszłości dostarczy wynik lub zakończy swoje działanie. Jest to klucz do interakcji z maszyną stanów
* async - To słowo kluczowe informuje kompilator, że metoda może zawierać operacje asynchroniczne. Co najważniejsze, **async** **przekształca całą metodę z prostej funkcji w obiekt – maszynę stanów**
* await - to słowo kluczowe działa jak **"punkt kontrolny" (checkpoint) wewnątrz maszyny stanów**. To w tym miejscu wykonanie metody może zostać wstrzymane, a wątek zwolniony.


Co jeżeli zmieniłoby się lekko funkcję?

```csharp
public async Task<string> MakeTeaAsync()
{
	var boilingWater = await BoilWaterAsync();
	
	"take the cups out".Dump();
	
	var a = 0
	//a tu wstawiamy loopa z dużą ilością iteracji i po prostu iterujemy po kolei 
	// dodatkowo zmniejszamy czas delay w BoilWaterAsync do 300
	
	"put tea in cups".Dump();
	
	var water = await boilingWater;
	var tea = $"pour water in cups".Dump();
	
	return tea;
}
```

Ta dodatkowa pętla którą dodajemy ma reprezentować dodatkowe, długie zajęcie które się dzieje na głównym wątku. I w wyniku widać, że gdy skończona została akcja z czajnikiem to wskoczyło to wcześniej do kodu przed pur tea in cups. Mimo że await mieliśmy po tym. Ten czajnik skończył się gotować na innym wątku
![[Pasted image 20250917204040.png]]

**Realniejszy przykład**
![[Pasted image 20250917204416.png]]

Mamy tutaj realniejszy kod. Po środku znów została wsawiona ta długa pętla
Wynik:
![[Pasted image 20250917204543.png]]
Jak widać, wszystko działa na jednym wątku: 1.
Jeżeli skrócimy pętle to wtedy dla numer 5 będziemy mieć inny wątek.
Wyjaśnienie dlaczego tak się dzieje - analiza zrobiona poprzez notebooklm, ale ma to w sumie sens i fajnie wyjaśnia:
W przedstawionym przykładzie kod wykonuje następujące kroki:
1. Inicjuje operację asynchroniczną, np. pobranie strony z Google (`getStringAsync`).
2. Następnie wykonuje pętlę `for`, która intensywnie obciąża procesor.
3. Na końcu czeka (`await`) na zakończenie operacji sieciowej i przetwarza jej wynik.

Oto wyjaśnienie, dlaczego liczba iteracji w pętli wpływa na liczbę użytych wątków:
Scenariusz 1: Krótka pętla (używane są dwa wątki)
1. **Rozpoczęcie operacji:** Główny wątek programu (nazwijmy go Wątkiem 1) wywołuje `getStringAsync`. Ta metoda zwraca obiekt `Task`, który reprezentuje przyszły wynik, i przekazuje zadanie pobrania danych do systemu operacyjnego. System operacyjny, poprzez sterowniki karty sieciowej, rozpoczyna operację w tle, **nie blokując Wątka 1**.
2. **Wykonanie pętli:** Wątek 1 natychmiast kontynuuje pracę i zaczyna wykonywać krótką pętlę `for`. Ponieważ pętla jest krótka, kończy się bardzo szybko.
3. **Oczekiwanie (****await****):** Po zakończeniu pętli Wątek 1 dociera do instrukcji `await`. W tym momencie operacja sieciowa prawdopodobnie **jeszcze się nie zakończyła**. Słowo kluczowe `await` sprawdza status zadania. Widząc, że jest ono nieukończone, Wątek 1 jest zwalniany i wraca do puli wątków, aby mógł być użyty do innych zadań.
4. **Zakończenie operacji w tle:** Gdy operacja sieciowa zostanie zakończona przez system operacyjny, informacja o tym trafia z powrotem do puli wątków .NET (`ThreadPool`). Pula wątków musi teraz wznowić wykonywanie reszty metody.
5. **Wznowienie na nowym wątku:** Pula wątków wybiera **dostępny wątek** (nazwijmy go Wątkiem 2), aby wykonać pozostałą część kodu po `await`. Dlatego w logach widać, że reszta funkcji (np. wypisanie "Hello world") wykonuje się na innym wątku.

Scenariusz 2: Długa pętla (używany jest jeden wątek)
1. **Rozpoczęcie operacji:** Proces wygląda tak samo – Wątek 1 wywołuje `getStringAsync` i operacja sieciowa rozpoczyna się w tle.
2. **Wykonanie pętli:** Wątek 1 zaczyna wykonywać bardzo długą pętlę `for`, która zajmuje dużo czasu i w pełni obciąża jeden rdzeń procesora.
3. **Zakończenie operacji w tle:** Operacja sieciowa (pobranie strony z Google) jest zazwyczaj bardzo szybka. Istnieje bardzo duże prawdopodobieństwo, że **zakończy się ona, zanim Wątek 1 skończy wykonywać swoją długą pętlę**.
4. **Oczekiwanie (****await****):** Wątek 1 w końcu kończy pętlę i dociera do instrukcji `await`. W tym momencie sprawdza status zadania i odkrywa, że **jest ono już ukończone**.
5. **Kontynuacja na tym samym wątku:** Ponieważ zadanie jest już zakończone, **nie ma potrzeby zwalniania wątku i czekania**. Kod może być kontynuowany synchronicznie, bez przełączania kontekstu. W rezultacie Wątek 1 po prostu wykonuje resztę metody.

**Podsumowując, kluczowa różnica polega na tym, czy operacja asynchroniczna zdąży się zakończyć, zanim główny wątek dotrze do słowa kluczowego** **await****.**

• **Krótka pętla:** Operacja w tle trwa dłużej niż pętla. Gdy kod dociera do `await`, musi poczekać, co powoduje zwolnienie bieżącego wątku i wznowienie pracy na innym wątku z puli, gdy zadanie zostanie ukończone.

• **Długa pętla:** Pętla trwa dłużej niż operacja w tle. Gdy kod dociera do `await`, zadanie jest już zakończone, więc nie ma potrzeby przełączania wątków, a wykonanie jest kontynuowane na tym samym wątku





