1. Pula Wątków (ThreadPool) jako Centralny Zarządca:
    ◦ W aplikacjach .NET to nie Ty bezpośrednio tworzysz i niszczysz wątki do obsługi operacji asynchronicznych. Odpowiada za to ThreadPool, czyli mechanizm, który zarządza kolekcją wątków roboczych.
    ◦ Głównym zadaniem puli jest efektywne ponowne wykorzystywanie istniejących wątków, aby unikać kosztownego procesu tworzenia nowych. Kiedy wątek kończy zadanie, wraca do puli i jest gotowy do podjęcia nowej pracy.
2. Rola Systemu Operacyjnego i Sprzętu:
    ◦ Pula wątków nie działa w próżni. Współpracuje ściśle z systemem operacyjnym, który jest odpowiedzialny za tzw. "scheduling", czyli przydzielanie czasu procesora poszczególnym wątkom.
    ◦ Ilość dostępnych rdzeni procesora (CPU cores) ma fundamentalne znaczenie. System operacyjny wykorzystuje dostępne rdzenie do równoległego wykonywania pracy. Jeśli masz cztery rdzenie, cztery wątki mogą działać dosłownie w tym samym momencie.
    ◦ Choć wątki mogą "przeskakiwać" między rdzeniami, to właśnie liczba rdzeni fizycznych jest jednym z głównych czynników ograniczających prawdziwą równoległość.
3. Dynamiczne Zarządzanie przez Środowisko Uruchomieniowe (.NET Runtime):
    ◦ Dostępna liczba wątków w puli nie jest stała. Środowisko uruchomieniowe .NET dynamicznie dostosowuje rozmiar puli wątków w zależności od aktualnego obciążenia aplikacji. Jeśli wiele zadań czeka na wykonanie, pula może utworzyć dodatkowe wątki. Jeśli obciążenie spada, nadmiarowe wątki mogą być z czasem usuwane.
    ◦ Źródła podkreślają, że ThreadPool jest inteligentny i stara się utrzymać optymalną liczbę wątków, aby zmaksymalizować wydajność bez nadmiernego zużycia zasobów systemowych.
4. Charakter Wykonywanej Pracy (CPU-bound vs. I/O-bound):
    ◦ Mechanizm async/await jest zaprojektowany głównie z myślą o operacjach I/O-bound (związanych z wejściem/wyjściem), takich jak zapytania sieciowe czy operacje na plikach.
    ◦ Podczas takiej operacji wątek nie wykonuje pracy obliczeniowej, tylko czeka. await zwalnia ten wątek z powrotem do puli, dzięki czemu może on obsłużyć inne zadanie, zamiast być bezczynnie blokowanym. Oznacza to, że jedna niewielka liczba wątków może obsłużyć ogromną liczbę jednoczesnych operacji I/O.
    ◦ Inaczej jest w przypadku operacji CPU-bound (intensywnych obliczeniowo), gdzie wątek aktywnie wykorzystuje procesor. W takim scenariuszu zwolnienie wątku nie ma sensu, a do zrównoleglenia pracy potrzebna jest większa liczba wątków (idealnie odpowiadająca liczbie rdzeni).
Podsumowując:
Dostępna liczba wątków w aplikacji .NET nie jest prostą, statyczną wartością. Jest to dynamicznie zarządzany zasób przez pulę wątków (ThreadPool), która dostosowuje się do obciążenia aplikacji, rodzaju wykonywanych zadań oraz możliwości sprzętowych (głównie liczby rdzeni procesora) i systemu operacyjnego. Głównym celem tego mechanizmu jest maksymalizacja wydajności poprzez efektywne ponowne użycie wątków, a nie tworzenie ich w nieograniczonej liczbie.


Odnośnie fragmentu: "await zwalnia ten wątek z powrotem do puli, dzięki czemu może on obsłużyć inne zadanie, zamiast być bezczynnie blokowanym."
• Kiedy Wątek 1 dochodzi do `await task;`, sprawdza, czy operacja (reprezentowana przez `task`) już się zakończyła.
• Jeśli nie, await mówi: "OK, ja tu nie mam nic do roboty. Zamiast się blokować i czekać, zwracam sterowanie, a wątek, na którym pracuję, może wrócić do puli wątków (ThreadPool)".
• Ten uwolniony wątek jest natychmiast gotowy do podjęcia zupełnie innego zadania z puli – może obsłużyć inne żądanie w aplikacji webowej, odświeżyć interfejs użytkownika itp.
Kiedy wątek jest znowu potrzebny?
Wątek jest potrzebny dopiero po zakończeniu operacji I/O, aby wykonać dalszą część kodu (tzw. kontynuację).
1. Gdy sterownik sieciowy kończy pobierać dane, informuje o tym system operacyjny.
2. System operacyjny powiadamia pulę wątków .NET, że zadanie jest gotowe.
3. Dopiero w tym momencie ThreadPool bierze dowolny dostępny wątek (może to być ten sam Wątek 1, ale często jest to zupełnie inny, np. Wątek 2) i używa go do wznowienia wykonywania Twojej metody od miejsca tuż po await