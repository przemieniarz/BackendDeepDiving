' concurrency is about MANAGING multiple task at once, parallelism is about EXECUTING multiple tasks at once.'

**Concurrency (Współbieżność)**
Współbieżność to zdolność programu do zarządzania wieloma zadaniami naraz, nawet na pojedynczym rdzeniu procesora. Kluczowym mechanizmem jest tutaj szybkie przełączanie kontekstu (context switching), gdzie procesor przełącza się między zadaniami, pracując nad każdym z nich przez krótki czas. Tworzy to iluzję, że zadania postępują jednocześnie, chociaż w rzeczywistości tak nie jest.
	• Analogia: Wyobraź sobie szefa kuchni, który przygotowuje kilka dań naraz. Przez chwilę zajmuje się jednym daniem, potem przełącza się na kolejne i tak na zmianę. Chociaż żadne danie nie jest gotowane w tym samym momencie, praca nad wszystkimi postępuje.
	• Zastosowanie: Współbieżność jest idealna dla zadań, które wiążą się z oczekiwaniem, np. na operacje wejścia/wyjścia (I/O), takie jak odczyt pliku czy zapytania do bazy danych. Pozwala to innym zadaniom na postęp w czasie tego oczekiwania, co poprawia ogólną wydajność i responsywność aplikacji, na przykład serwera WWW obsługującego wiele zapytań jednocześnie.

**Parallelism (Równoległość)**
Równoległość polega na jednoczesnym wykonywaniu wielu zadań przy użyciu wielu rdzeni procesora. Każdy rdzeń obsługuje inne zadanie niezależnie w tym samym czasie.
	• Analogia: Wyobraź sobie kuchnię z dwoma szefami kuchni. Jeden kroi warzywa, podczas gdy drugi w tym samym czasie gotuje mięso. Obie czynności dzieją się równolegle, dzięki czemu posiłek jest gotowy szybciej.
	• Zastosowanie: Równoległość doskonale sprawdza się w przypadku ciężkich obliczeń, takich jak analiza danych, renderowanie grafiki, trenowanie modeli uczenia maszynowego czy symulacje naukowe. Zadania te można podzielić na mniejsze, niezależne podzadania i wykonywać je jednocześnie na różnych rdzeniach, co znacznie przyspiesza proces.
	
**Relacja między Concurrency a Parallelism**
Chociaż są to różne pojęcia, są ze sobą ściśle powiązane. Współbieżność dotyczy zarządzania wieloma zadaniami naraz, podczas gdy równoległość dotyczy ich jednoczesnego wykonywania. Współbieżność może umożliwiać równoległość, ponieważ programy napisane współbieżnie można łatwiej podzielić na niezależne zadania, które następnie mogą być rozproszone na wiele rdzeni i wykonywane równolegle. Współbieżność tworzy więc fundament, który ułatwia osiągnięcie równoległości