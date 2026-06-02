# Zadanie 2 - Filip Kwietniak

## Opis

Repozytorium zawiera pipeline GitHub Actions, który automatycznie buduje wieloarchitekturowy obraz kontenera aplikacji, skanuje go pod kątem podatności CVE przy użyciu narzędzia Trivy i publikuje w GitHub Container Registry.

## Pipeline GitHub Actions
Pipeline uruchamia się automatycznie przy każdym pushu na gałąź `main`.

### Kroki pipeline:

1. Checkout — pobranie kodu źródłowego
2. QEMU — konfiguracja emulacji ARM64
3. Docker Buildx — konfiguracja buildera wieloarchitekturowego
4. Login DockerHub — logowanie do cache na DockerHub
5. Login ghcr.io — logowanie do GitHub Container Registry
6. Metadata — wyznaczenie tagów obrazu
7. Build (amd64) — budowa obrazu tylko na amd64 do skanowania
8. Trivy CVE scan — skanowanie obrazu pod kątem podatności CRITICAL i HIGH
9. Build & Push  — budowa i publikacja obrazu na obie architektury, pod warunkiem, że w skanie nie ma podatności Critical lub HIGH

## Tagowanie obrazów

Na obrazie zastosowano dwa tagi:

- `latest` — zawsze wskazuje na najnowszy obraz z gałęzi `main`
  Uzasadnienie: Wskazuje na najnowszą wersję obrazu z gałęzi main. Ułatwia szybkie pobieranie najświeższego buildu.
- `sha-xxxxxxx` — unikalny tag oparty na skrócie commita Git
  Uzasadnienie: Tag zapewnia bezpieczeństwo i pewność że używamy odpowiedniej wersji. W przeciwieństwie do tagu latest, 'SHA' gwarantuje, że uruchamiamy dokładnie ten sam kod, który został przetestowany.

## Tagowanie cache

Dane cache przechowywane są w publicznym repozytorium DockerHub `fkinf/weather-app-cache` pod tagiem `cache`.

Użyto eksportera typu `registry` w trybie `max`, który zapisuje wszystkie warstwy pośrednie. Pozwala to na skrócenie czasu budowania przy kolejnych uruchomieniach pipeline, ponieważ Docker ponownie wykorzystuje już zbudowane warstwy.


## Skanowanie CVE

Do skanowania obrazu użyto skanera Trivy

Trivy zostało wybrane ponieważ:
- działa natywnie w GitHub Actions bez dodatkowej konfiguracji konta
- jest w pełni open-source oraz darmowe
- jest powszechnie stosowany w projektach open-source
- Umożliwia łatwą konfigurację przerwania pracy pipeline'u przez exit-code 1

Skan sprawdza podatności sklasyfikowane jako CRITICAL i HIGH. Jeśli zostaną wykryte — pipeline zatrzymuje się i obraz nie jest publikowany na ghcr.io.

## Sekrety

W repozytorium skonfigurowano następujące sekrety:
`DOCKERHUB_USERNAME` - Nazwa użytkownika DockerHub
`DOCKERHUB_TOKEN`  - Token dostępu DockerHub

Token `GITHUB_TOKEN` jest dostarczany automatycznie przez GitHub Actions.

## Potwierdzenie działania pipelne

Pipeline został uruchomiony i zakończony pomyślnie. Link do uruchomienia: https://github.com/FKinf/Zadanie2AplikacjaPogodowa/actions/runs/26842797565

## Obraz na ghcr.io
Obraz dostępny jest pod adresem: ghcr.io/fkinf/zadanie2aplikacjapogodowa:latest
