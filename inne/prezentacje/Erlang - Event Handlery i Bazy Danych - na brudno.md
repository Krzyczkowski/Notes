# Event Handlery

Jest pewna kwestia, której unikałem w poprzednich przykładach. Jeśli spojrzysz na aplikację przypominającą, zauważysz, że wspomniałem, iż możemy powiadamiać klientów poprzez IM, e-mail itp. W poprzednim rozdziale nasz system handlowy używał io:format/2 do informowania ludzi o bieżących wydarzeniach.

Możesz zauważyć wspólny element w obu przypadkach: chodzi o informowanie ludzi (lub pewien proces czy aplikację) o wydarzeniu, które miało miejsce w określonym czasie. W jednym przypadku jedynie wyświetlaliśmy wyniki, a w drugim pobieraliśmy Pid subskrybentów przed wysłaniem im wiadomości.

Podejście z wyświetlaniem wyników jest minimalistyczne i trudno je rozbudować. Podejście z subskrybentami jest zdecydowanie poprawne i bardzo przydatne, gdy każdy z subskrybentów ma długotrwałą operację do wykonania po otrzymaniu wydarzenia. W prostszych przypadkach, gdy nie chcesz, aby proces oczekiwał na wydarzenia dla każdego z callbacków, można zastosować trzecie podejście.

To trzecie podejście polega na wykorzystaniu procesu, który akceptuje funkcje i pozwala im działać przy każdym nadchodzącym wydarzeniu. Proces ten nazywa się zazwyczaj menedżerem wydarzeń i może wyglądać mniej więcej tak:![[Pasted image 20240518132630.png]]

Robienie rzeczy w ten sposób ma kilka zalet:

1. **Efektywność obsługi wielu subskrybentów:** Jeśli nasz serwer ma wielu subskrybentów, używając tego podejścia, serwer może działać płynnie, ponieważ musi przekazać wydarzenia tylko raz, niezależnie od liczby subskrybentów. Każdy subskrybent otrzymuje powiadomienie z jednego źródła, co redukuje obciążenie serwera.

Załóżmy, że mamy serwer, który chce powiadomić swoich subskrybentów o pewnym zdarzeniu, na przykład nowej wiadomości. Zamiast bezpośrednio wysyłać wiadomość do każdego subskrybenta, serwer przekazuje zdarzenie (np. informację o nowej wiadomości) do menedżera wydarzeń.
Menedżer wydarzeń jest odpowiedzialny za śledzenie wszystkich subskrybentów i przekazywanie im odpowiednich zdarzeń. Dzięki temu, nawet jeśli serwer ma tysiące subskrybentów, musi tylko raz przekazać zdarzenie do menedżera wydarzeń, który następnie rozsyła je do wszystkich subskrybentów.

2. **Optymalne wykorzystanie przesyłanych danych:** Jeśli przesyłamy dużo danych w ramach jednego wydarzenia, to robi się to tylko raz, a wszystkie funkcje callback operują na tych samych danych. W ten sposób unikamy niepotrzebnego powielania przesyłanych danych, co może przyspieszyć przetwarzanie i zminimalizować zużycie zasobów.
3. **Uniknięcie tworzenia dodatkowych procesów:** Dzięki temu podejściu nie musimy tworzyć osobnych procesów dla krótkotrwałych zadań. Funkcje callback mogą być wywoływane bezpośrednio w ramach menedżera wydarzeń, co eliminuje dodatkowy narzut związany z tworzeniem i zarządzaniem procesami.

Oczywiście, istnieje też kilka wad:

1. **Ryzyko blokowania się funkcji:** Jeśli wszystkie funkcje callback działają przez długi czas lub w nieskończoność, mogą wzajemnie się blokować. Aby temu zapobiec, można skorzystać z podejścia, w którym menedżer wydarzeń działa jako przekaźnik wydarzeń, przekazując wydarzenie do osobnego procesu. W przeciwnym razie, długotrwałe lub nieskończone funkcje mogą spowodować zatrzymanie obsługi nowych wydarzeń, co prowadzi do niedziałającej lub opóźnionej obsługi.
2. **Ryzyko zawieszenia obsługi nowych wydarzeń:** Funkcja działająca w nieskończonej pętli może uniemożliwić obsługę nowych wydarzeń, dopóki ta funkcja się nie zakończy lub nie zostanie przerwana. W takiej sytuacji, inne wydarzenia mogą czekać w kolejce, co prowadzi do opóźnień w obsłudze lub nawet ich utraty.

Jest sposób na rozwiązanie tych wad, choć może wydawać się mało satysfakcjonujący. W zasadzie trzeba przekształcić podejście menedżera wydarzeń w podejście z subskrybentami. Na szczęście podejście menedżera wydarzeń jest na tyle elastyczne, że można to zrobić z łatwością, a jak to zrobić, pokażemy później w tym rozdziale.

### Przykład Przepływu Zdarzenia

1. **Interakcja użytkownika w aplikacji klienckiej**:
    
    - Użytkownik klika przycisk, co generuje zdarzenie.
2. **Serwer odbiera zdarzenie**:
    
    - Serwer odbiera zdarzenie od aplikacji klienckiej.
    - Serwer może przetworzyć zdarzenie, na przykład weryfikując dane lub aktualizując stan.
3. **Serwer wysyła zdarzenie do menedżera zdarzeń**:
    
    - Po przetworzeniu zdarzenia, serwer wysyła zdarzenie do menedżera zdarzeń.
4. **Menedżer zdarzeń dystrybuuje zdarzenie do handlerów**:
    
    - Menedżer zdarzeń przekazuje zdarzenie do zarejestrowanych handlerów.
    - Każdy handler, który jest zainteresowany danym typem zdarzenia, wykonuje odpowiednie akcje (np. logowanie, powiadomienie, aktualizacja stanu).
5. **Odpowiedź do klienta (opcjonalne)**:
    
    - W niektórych przypadkach, serwer może wysłać odpowiedź do aplikacji klienckiej, informując o wyniku przetwarzania zdarzenia

Główna idea polega na tym, że w `gen_event`, to handlery są tymi, które działają zgodnie ze standardowym wzorcem procesów w Erlangu (`spawn -> init -> loop -> terminate`). Menedżer zdarzeń sam jest bardziej koordynatorem, który zarządza tymi handlerami i zapewnia, że każde zdarzenie jest odpowiednio obsłużone.


 Oto gen_event. Zachowanie gen_event różni się znacznie od zachowań gen_server i gen_fsm, ponieważ nigdy tak naprawdę nie uruchamiasz procesu. Cała opisana wcześniej część dotycząca 'akceptowania callbacku' jest powodem tego. Zachowanie gen_event zasadniczo uruchamia proces, który akceptuje i wywołuje funkcje, a Ty dostarczasz tylko moduł z tymi funkcjami. Oznacza to, że nie musisz się zajmować manipulacją wydarzeniami, wystarczy, że podasz swoje funkcje callback w formacie, który odpowiada menedżerowi wydarzeń. Wszystkie zarządzanie jest wykonane automatycznie; dostarczasz tylko to, co specyficzne dla Twojej aplikacji. Nie jest to zaskakujące, biorąc pod uwagę, że OTP polega na oddzielaniu tego, co ogólne, od tego, co specyficzne.

To rozdzielenie oznacza jednak, że standardowy wzorzec spawn -> init -> loop -> terminate będzie stosowany tylko do obsługiwaczy wydarzeń. Jeśli przypomnisz sobie to, co zostało wcześniej powiedziane, obsługiwacze wydarzeń to zbiór funkcji działających w menedżerze. To oznacza obecnie przedstawiony model:



# Mnesia


Mnesia jest zaimplementowana w Erlangu i ściśle z nim powiązana. Zapewnia funkcjonalność niezbędną do wdrażania odpornych na błędy systemów telekomunikacyjnych. Mnesia jest wieloużytkownikowym, rozproszonym DBMS, specjalnie zaprojektowanym do zastosowań telekomunikacyjnych klasy przemysłowej, napisanych w Erlangu, który jest również docelowym językiem programowania. Mnesia stara się rozwiązać wszystkie problemy związane z zarządzaniem danymi wymagane przez typowe systemy telekomunikacyjne i ma szereg funkcji, które nie są normalnie spotykane w tradycyjnych DBMS.

Aplikacje telekomunikacyjne potrzebują kombinacji szerokiego zakresu funkcji, które zazwyczaj nie są dostarczane przez tradycyjne DBMS. Mnesia została zaprojektowana, aby sprostać wymaganiom takim jak:

- Szybkie wyszukiwanie klucz-wartość w czasie rzeczywistym
- Złożone zapytania nie w czasie rzeczywistym (głównie do zadań operacyjnych i konserwacyjnych)
- Rozproszone dane (ze względu na rozproszony charakter aplikacji)
- Wysoka tolerancja na błędy
- Dynamiczna rekonfiguracja
- Złożone obiekty

Mnesia rozwiązuje typowe problemy zarządzania danymi wymagane w aplikacjach telekomunikacyjnych, co odróżnia ją od większości innych DBMS. Łączy wiele koncepcji spotykanych w tradycyjnych DBMS, takich jak transakcje i zapytania, z koncepcjami spotykanymi w systemach zarządzania danymi dla aplikacji telekomunikacyjnych, takimi jak:

- Szybkie operacje w czasie rzeczywistym
- Konfigurowalna replikacja dla tolerancji na błędy
- Dynamiczna rekonfiguracja bez przerw w usługach

Mnesia jest również unikalna ze względu na ścisłe powiązanie z Erlangiem. Praktycznie zmienia Erlang w język programowania baz danych, co przynosi wiele korzyści. Najważniejszą z nich jest to, że znikają wszelkie niezgodności między formatem danych używanym przez DBMS a formatem danych używanym przez język programowania do manipulacji danymi.


Mnesia posiada następujące cechy, które razem tworzą odporny na błędy rozproszony system zarządzania bazą danych (DBMS) napisany w Erlangu:
- Schemat bazy danych może być dynamicznie rekonfigurowany w czasie działania.
- Tabele mogą mieć deklarowane właściwości, takie jak lokalizacja, replikacja i trwałość.
- Tabele mogą być przenoszone lub replikowane na kilka węzłów w celu poprawy tolerancji na błędy. Inne węzły w systemie mogą nadal uzyskiwać dostęp do tabel w celu odczytu, zapisu i usuwania rekordów.
- Lokalizacje tabel są przejrzyste dla programisty. Programy adresują nazwy tabel, a system sam śledzi lokalizacje tabel.
- Transakcje mogą być rozproszone, a wiele operacji może być wykonywanych w ramach jednej transakcji.
- Wiele transakcji może być uruchamianych jednocześnie, a ich wykonanie jest w pełni synchronizowane przez Mnesia, zapewniając, że żadne dwa procesy nie manipulują tymi samymi danymi jednocześnie.
- Transakcje mogą mieć przypisaną właściwość wykonywania na wszystkich węzłach w systemie lub na żadnym. Transakcje można pomijać, używając operacji "brudnych" (dirty operations), które zmniejszają koszty ogólne i działają szybko.

Wszystkie powyższe cechy są szczegółowo opisane w nadchodzących sekcjach.

### Kiedy używać Mnesia

Mnesia jest idealnym rozwiązaniem dla aplikacji, które:
- Potrzebują replikacji danych.
- Wykonują złożone zapytania do danych.
- Potrzebują używać atomowych transakcji do bezpiecznej aktualizacji kilku rekordów jednocześnie.
- Wymagają miękkich charakterystyk czasu rzeczywistego.

Mnesia nie jest odpowiednia dla aplikacji, które:
- Przetwarzają zwykłe pliki tekstowe lub binarne.
- Potrzebują jedynie słownika wyszukiwania, który można przechowywać na dysku. Takie aplikacje mogą używać modułu standardowej biblioteki dets, który jest wersją opartą na dysku modułu ets. Więcej informacji o dets można znaleźć na stronie manuala dets w STDLIB.
- Potrzebują funkcji logowania na dysku. Takie aplikacje mogą używać modułu disk_log. Więcej informacji o disk_log można znaleźć na stronie manuala disk_log w Kernel.
- Wymagają twardych charakterystyk czasu rzeczywistego.



 erl -mnesia dir '"./"'
 mnesia:create_schema([node()]).

- **Krok 1**: System Erlang jest uruchamiany z poziomu terminala z flagą `-mnesia dir '"./"'`, która wskazuje, w którym katalogu mają być przechowywane dane.
- **Krok 2**: Nowy, pusty schemat jest inicjalizowany na lokalnym nodzie poprzez wywołanie `mnesia:create_schema([node()])`. (Schemat zawiera informacje ogólne o bazie danych)
- **Krok 3**: System zarządzania bazą danych (DBMS) jest uruchamiany przez wywołanie `mnesia:start()`.
- **Krok 4**: Tworzona jest pierwsza tabela, nazwana test, poprzez wywołanie `mnesia:create_table(test, [])`. Tabela otrzymuje domyślne właściwości.
- **Krok 5**: Wywoływane jest `mnesia:info()`, aby wyświetlić na terminalu informacje o statusie bazy danych.

