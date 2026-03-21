---
title: "Intensywny tydzień: Rust, Solana, Docker, Rustikon i nowy blog"
date: 2026-03-21 22:00:00 +0100
categories: [Programming, Web3]
tags: [rust, solana, docker, jekyll, confitura]
---

### Intensywny tydzień w świecie Rusta i nie tylko

Miniony tydzień był dla mnie prawdziwą jazdą bez trzymanki, jeśli chodzi o przyswajanie całkowicie nowej wiedzy i technologiczny networking. Od dłuższego czasu starałem się wejść w ekosystem Rusta i Web3, a ostatnie dni dostarczyły mi do tego idealnych okazji. Połączyłem z sobą intensywną naukę na warsztatach, rozwiązywanie problemów infrastrukturalnych, udział w świetnej konferencji, a do tego spotkanie z naszym zespołem organizacyjnym [Confitury](https://2026.confitura.pl).

### Rust, Solana i faucet_clicker

Zeszły weekend w całości spędziłem na warsztatach poświęconych technologiom Rust i Solana. Przyznam szczerze: na początku byłem do tego wszystkiego dość sceptycznie nastawiony. Ludzi ze świata Web3 uważałem trochę za specyficzną "krypto-sektę". Szybko jednak okazało się, że moje obawy były całkowicie bezpodstawne – zamiast szumu marketingowego dostałem świetnych inżynierów i dużą dawkę merytorycznej, technicznej wiedzy.

Chłopaki dobrze poukładali mi w głowie to, jak działa blockchain Solany oraz jak pisać na niego *smart contracty* (programy). Nie skończyło się jednak tylko na teorii. Pod ich okiem rozpocząłem budowę swojego pierwszego projektu. Wybrałem aplikację typu `faucet_clicker` – idealny "Hello World" na sterydach, który pozwala przećwiczyć interakcję z programami na Solanie, zarządzanie stanem i podpisywanie transakcji. Projekt dostał roboczą nazwę **GimmeSol**, a jego kod i moje bieżące postępy możecie na bieżąco obserwować w repozytorium: [gimme-sol](https://github.com/tymoteuszlenart/gimme-sol).

Praca z frameworkiem Anchor zapowiadała się świetnie, ale jak to w programowaniu bywa, rzeczywistość szybko zweryfikowała moje zapały oraz środowisko deweloperskie i rzuciła mi pierwsze kłody pod nogi.

### Kiedy macOS mówi "nie": Solana CLI w Dockerze

Podczas pierwszej próby odpalenia polecenia `anchor build` okazało się, że mój poczciwy **macOS Big Sur (wersja 11.7.11)** jest po prostu za stary na oficjalne wsparcie najnowszych narzędzi Solany, a dokładniej . Próbowałem przez chwilę walczyć z systemowymi zależnościami, ale bezskutecznie. Rzuciłem wszystko i postanowiłem przygotować własny obraz Dockera z narzędziami Solana CLI. **BONUS**: zyskałem świetne środowisko do łatwego uruchamiania klastra walidatora!

Stworzyłem konfigurację opartą o `docker-compose`, która stawia dwa kontenery: jeden do developmentu, a drugi z lokalnym walidatorem (`solana-test-validator`). Kontener devowy współdzieli sieć z walidatorem, co eliminuje wszelkie problemy z łącznością z lokalnym RPC.

```yaml
version: "3.9"

services:
  solana-dev:
    platform: linux/amd64
    build: .
    container_name: solana-dev
    network_mode: service:validator
    volumes:
      - ./workspace:/workspace
      - cargo-cache:/root/.cargo
      - target-cache:/workspace/target
      - ~/.config/solana:/root/.config/solana
    stdin_open: true
    tty: true
    command: bash

  validator:
    platform: linux/amd64
    build: .
    container_name: solana-validator
    volumes:
      - validator-ledger:/workspace/test-ledger
    ports:
      - "8900:8899"
      - "8901:8900"
    command: solana-test-validator --reset

volumes:
  cargo-cache:
  target-cache:
  validator-ledger:
```

Warto tu zwrócić uwagę na jedną linijkę w `volumes`: `- ~/.config/solana:/root/.config/solana`. Dzięki temu środowisko w kontenerze po uruchomieniu automatycznie zaciąga klucz mojego portfela, co pozwala na natychmiastowe zasilenie go airdropem 100 SOL na potrzeby lokalnego developmentu.

Dodatkowo, sam walidator jest zawsze wywoływany z flagą `--reset` (`command: solana-test-validator --reset`). Dzięki temu przy każdym uruchomieniu środowiska klaster startuje zupełnie od zera. To bardzo dobra praktyka deweloperska, która pozwala uniknąć wielu frustrujących problemów z "nieświeżym" stanem aplikacji i konfliktami danych pomiędzy kolejnymi sesjami.

Skoro nasz walidator już działa w osobnym kontenerze, musimy pamiętać o jeszcze jednym ważnym detalu. Podczas uruchamiania testów w Anchorze konieczne jest dodanie flagi `--skip-local-validator` (czyli wywołujemy: `anchor test --skip-local-validator`). W przeciwnym razie Anchor spróbuje postawić swój własny klaster testowy, co natychmiast skończy się błędem zajętych portów.

Jeśli również macie problemy ze środowiskiem lub po prostu wolicie trzymać śmieci z dala od głównego systemu, przygotowałem specjalne repozytorium na GitHubie. Wystarczy sklonować [solana-docker-dev](https://github.com/tymoteuszlenart/solana-docker-dev) i od razu możecie cieszyć się gotowym środowiskiem Solany zamkniętym w kontenerze.

### Rustikon.dev – web development z klocków i nie tylko

Skoro w piątek i tak musiałem się pojawić w Warszawie to wykorzystałem okazję i uczestniczyłem w konferencji [rustikon.dev](https://rustikon.dev). Absolutnie było warto! Świetna okazja, żeby posłuchać, co w trawie piszczy w polskiej społeczności Rusta.

Szczególnie w pamięć zapadła mi prezentacja chłopaków z [cot.rs](https://cot.rs) dotycząca web developmentu. Ekosystem webowy Rusta jest mocno modułowy – zamiast jednego monolitycznego frameworka dobierasz osobne crate’y (framework HTTP, runtime async, DB, serializacja itd.) i składasz z nich stack dopasowany do projektu. To daje sporą elastyczność, ale wymaga więcej świadomych wyborów niż w „jedynym słusznym” frameworku. To bardzo ciekawe i świeże podejście w porównaniu do potężnych kombajnów znanych z innych języków.

Piątkowe prelekcje obejmowały zresztą bardzo szerokie i zróżnicowane spektrum tematów:
* **macro_rules!** – konkretna prezentacja pokazująca *few tricks to help you write cleaner, more powerful declarative macros*.
* **Superteam Poland** – krótki i treściwy lightning talk odnośnie ekosystemu Solany. Dowiedzieliśmy się, co to za technologia, kto ją rozwija, dlaczego warto się nią zainteresować i jakie duże firmy już jej zaufały.
* **Amateur automotive Rust** – Frank Lyaruu (gość z Holandii) opowiedział o sztuce *CAN bus sniffing*. Pokazał, jak podłączyć się do centralnego układu nerwowego nowoczesnego samochodu, jak interpretować przesyłane tam dane i co software engineers mogą nauczyć się od embedded. Niesamowicie inspirujące!

### Confitura – odliczanie rozpoczęte!

Żeby domknąć ten intensywny tydzień, pod jego koniec spotkaliśmy się z zespołem organizacyjnym Confitury. Tym razem spotkanie miało jednak zdecydowanie bardziej rekreacyjny charakter – musieliśmy zresetować baterie przed nadchodzącymi wyzwaniami.

Wybraliśmy się do [Muzeum Gier Wideo (Warsaw Arcade Museum)](https://www.warsawarcademuseum.com/), gdzie postrzelaliśmy do Obcych, poodbijaliśmy piłeczkę w klasycznego Ponga na Atari zmontowanego w stole oraz przetestowałem *Star Wars: Battle Pod* – to sprzęt dający wrażenia trochę w stylu kina 8D!

W tak luźnej atmosferze omówiliśmy strategię działania na nadchodzące pół roku. Aż trudno uwierzyć, że kolejna edycja zbliża się tak wielkimi krokami! Pracy przed nami mnóstwo, ale energia w zespole jest niesamowita. Będę na bieżąco dzielił się z wami ciekawostkami zza kulis organizacji, więc wypatrujcie kolejnych aktualizacji.

### Nowy dom w sieci: Blog, CV i portfolio w jednym

Oprócz nauki Rusta i rozwiązywania problemów infrastrukturalnych, po powrocie zabrałem się wreszcie za uporządkowanie mojej obecności w sieci. Postanowiłem stworzyć jedno, spójne miejsce, które będzie jednocześnie moim blogiem technicznym, wirtualnym CV oraz portfolio projektów.

Wybór technologii padł na **Jekyll** – niezwykle popularny i szybki generator stron statycznych (SSG), który połączyłem z minimalistycznym, bardzo czytelnym motywem **Chirpy**. Konfiguracja okazała się bardzo przyjemna, a efekt wizualny (który właśnie czytacie!) w pełni mnie satysfakcjonuje.

Co najważniejsze, całą logistykę związaną z publikacją zautomatyzowałem. Wykorzystałem do tego **GitHub Actions**. Dzięki temu proces dodawania nowego wpisu jest banalnie prosty – po prostu pushuję nowy plik Markdown do repozytorium, a odpowiedni workflow automatycznie zajmuje się budowaniem statycznych plików i wrzucaniem ich na serwer. Zero stresu, pełna wygoda.

---
No to na ten tydzień chyba starczy.
Jutro 🚲 - do zobaczenia na szlaku :)