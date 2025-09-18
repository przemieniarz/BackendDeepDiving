**Sync**
Powiedzmy, że mamy request. Np liczymy sobie jakieś operacje i to zajmuje sporo czasu. Odpalamy to poprzez wątek, ale w trakcie jak to coś jest kalkulowane to de fakto ten wątek nic nie robi, tylko czeka aż coś innego skończy liczyć i poda swój wynik. Czyli marnuje się on, mógłby coś robić, a nie leniuszkować. No i cała reszta kodu też jest zblokowana.

**Async**
Mamy request. Wyołujemy cosco mieli sięzewnętrznie. Thread bierze udział w tym, aby operacja się rozpoczęła ale jeżeli ma tam czekać bez sensu to jest zwalniany i robi coś innego. Gdy dociera do await to wtedy wraca do tamtego tematu i zgarnia rezultat

** Multithreading**
**Technika, w której w ramach jednego procesu uruchamianych jest wiele wątków wykonania**. Pomyśl o procesie jak o kontenerze dla Twojej aplikacji; domyślnie ma on jeden wątek, który wykonuje całą pracę. W wielowątkowości możesz "dorzucić" do tego kontenera dodatkowe wątki, aby wykonywały inne zadania.

• **Kluczowa cecha:** Wszystkie te wątki **dzielą te same zasoby procesu**, takie jak pamięć.

• **Główny cel i problem:** Celem jest umożliwienie wykonywania wielu operacji jednocześnie, co na komputerach z wieloma rdzeniami procesora może oznaczać prawdziwą równoległość (każdy wątek na innym rdzeniu). Jednakże, współdzielenie zasobów jest ogromnym problemem, ponieważ prowadzi do ryzyka **warunków wyścigu (race conditions)** i konieczności stosowania skomplikowanych mechanizmów blokujących (locking), aby zapewnić bezpieczeństwo (thread safety). Z tego powodu pisanie poprawnego kodu wielowątkowego jest uważane za bardzo trudne


**Multiprocessing**
**Podejście polegające na uruchamianiu wielu niezależnych procesów zamiast wielu wątków w jednym procesie**. Każdy z tych procesów jest odrębną "instancją" aplikacji.

• **Kluczowa cecha:** Każdy proces ma **swoją własną, odizolowaną pamięć i zasoby**. Nie ma problemu współdzielenia pamięci, który występuje w wielowątkowości.

• **Komunikacja:** Ponieważ procesy są odizolowane, muszą komunikować się ze sobą za pomocą mechanizmów komunikacji międzyprocesowej (Inter-Process Communication), takich jak gniazda (sockets) czy scentralizowana baza danych (np. Redis).

• **Główny cel i zastosowanie:** To podejście jest idealne do **dzielenia dużego problemu na mniejsze, niezależne części**, które mogą być przetwarzane równolegle. Jest to również bardziej skalowalne rozwiązanie, ponieważ poszczególne procesy można uruchamiać nie tylko na jednej maszynie, ale także na wielu różnych maszynach w sieci. Za przykład zastosowania podaje się łamanie haseł, gdzie każdy proces może przeszukiwać inny fragment "słownika"


**Różnice pomiędzy multithreading a async**
Główna różnica sprowadza się do sposobu, w jaki zarządzane są zadania, które wymagają oczekiwania, oraz jak wykorzystywane są zasoby systemowe, takie jak wątki i rdzenie procesora.

Multithreading (Wielowątkowość)
• Na czym polega: W modelu wielowątkowym, aby wykonać kilka zadań jednocześnie (lub prawie jednocześnie), tworzysz dodatkowe wątki w ramach jednego procesu. Każdy z tych wątków może wykonywać oddzielną pracę. Jeśli Twój komputer ma wiele rdzeni procesora, te wątki mogą działać dosłownie równolegle (in parallel), każdy na innym rdzeniu.
• Współdzielenie zasobów: Wszystkie wątki w jednym procesie dzielą te same zasoby, takie jak pamięć. To jest największe wyzwanie i źródło problemów.
• Główne problemy:
    ◦ Warunki wyścigu (race conditions) i blokady (locking): Gdy wiele wątków próbuje uzyskać dostęp do tej samej pamięci w tym samym czasie, może to prowadzić do nieprzewidywalnych błędów. Aby temu zapobiec, programiści muszą stosować skomplikowane mechanizmy synchronizacji, takie jak blokady (mutexes), co jest bardzo trudne do poprawnego zaimplementowania.
    ◦ Złożoność: Pisanie bezpiecznego kodu wielowątkowego (thread-safe) jest niezwykle trudne i wymaga głębokiej wiedzy o działaniu systemu operacyjnego i procesora

Asynchronous (Asynchroniczność)
• Na czym polega: Programowanie asynchroniczne, w swojej klasycznej formie (np. w Node.js), opiera się na idei jednego wątku, który nie jest blokowany. Zamiast tworzyć nowy wątek do obsługi czasochłonnej operacji (np. zapytania sieciowego lub odczytu pliku), główny wątek inicjuje tę operację i natychmiast wraca do wykonywania innych zadań.
Jak to działa:
    1. Główny wątek wysyła żądanie do zewnętrznego zasobu (np. kontrolera I/O, sterownika sieciowego).
    2. Zamiast czekać na odpowiedź, przekazuje tzw. funkcję zwrotną (callback) i mówi: "Zadzwoń do mnie, gdy skończysz".
    3. Wątek jest natychmiast uwalniany i może obsługiwać inne zdarzenia lub wykonywać dalszy kod.
    4. Gdy operacja zewnętrzna się zakończy, system informuje o tym główny wątek (np. poprzez pętlę zdarzeń - event loop), który następnie wykonuje kod z przekazanej wcześniej funkcji zwrotnej.
• Główny cel: Głównym celem asynchroniczności jest unikanie blokowania wątku podczas operacji, w których wątek i tak nie wykonuje żadnej pracy obliczeniowej (np. czekając na odpowiedź z serwera lub odczyt z dysku). Wątek jest po prostu "w stanie bezczynności i oczekiwania", co jest marnotrawstwem zasobów.
• async/await jako ułatwienie: Mechanizm async/await, który omawialiśmy wcześniej, jest "składniowym lukrem" (syntactical sugar), który pozwala pisać kod asynchroniczny w sposób, który wygląda jak synchroniczny, ukrywając złożoność funkcji zwrotnych (tzw. "callback hell").

Czy w może występować kilka wątków?
Tak, i to jest kluczowy punkt, który często prowadzi do nieporozumień, zwłaszcza w kontekście .NET i C#.
Chociaż idea asynchroniczności wywodzi się z modelu jednowątkowego (jak w Node.js), w implementacji .NET async/await bardzo często wykorzystywanych jest wiele wątków, ale działają one inaczej niż w klasycznym multithreadingu.
Oto jak to działa w .NET, co było szczegółowo opisane w naszej poprzedniej rozmowie o maszynie stanów:
1. Inicjacja na jednym wątku: Twoja metoda async rozpoczyna działanie na jednym wątku (np. Wątek 1).
2. Zwolnienie wątku przy await: Gdy kod dochodzi do await na nieukończonym zadaniu (np. operacji sieciowej), Wątek 1 nie jest blokowany. Zamiast tego jest zwalniany i wraca do puli wątków (ThreadPool), gdzie może być użyty do wykonania zupełnie innej pracy.
3. Wznowienie na innym wątku: Gdy operacja I/O się zakończy, pula wątków zostaje o tym poinformowana. ThreadPool bierze dowolny dostępny wątek (np. Wątek 2) i używa go do wznowienia wykonywania reszty metody async od miejsca, w którym została przerwana

Podsumowując, kluczowa różnica polega na celu i sposobie użycia wątków:
• W multithreadingu tworzysz wątki celowo, aby wykonywać pracę równolegle (np. intensywne obliczenia na wielu rdzeniach) i sam musisz zarządzać ich synchronizacją.
• W asynchroniczności w .NET wiele wątków jest używanych jako mechanizm wykonawczy do obsługi kontynuacji zadań. Nie tworzysz ich bezpośrednio, aby coś zrównoleglić, lecz pozwalasz puli wątków efektywnie zarządzać zadaniami, które nie wymagają ciągłej pracy procesora. Celem jest efektywność i responsywność, a niekoniecznie równoległość obliczeniowa