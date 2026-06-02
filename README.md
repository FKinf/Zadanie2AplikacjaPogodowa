# Zadanie 2 — Technologie Chmurowe

## Opis

Repozytorium zawiera pipeline GitHub Actions, który automatycznie buduje obraz kontenera aplikacji pogodowej, skanuje go pod kątem podatności CVE i publikuje na GitHub Container Registry (ghcr.io).

---

## Struktura repozytorium
├── app/                        # Kod źródłowy aplikacji Flask
├── Dockerfile                  # Główny Dockerfile (użyty w pipeline)
├── Dockerfile.multiarch        # Alternatywny Dockerfile z klonowaniem SSH
└── .github/
└── workflows/
└── docker-build.yml    # Pipeline GitHub Actions
---

## Pipeline GitHub Actions

Pipeline uruchamia się automatycznie przy każdym pushu na gałąź `main`.

### Kroki pipeline:

1. **Checkout** — pobranie kodu źródłowego
2. **QEMU** — konfiguracja emulacji ARM64
3. **Docker Buildx** — konfiguracja buildera wieloarchitekturowego
4. **Login DockerHub** — logowanie do cache na DockerHub
5. **Login ghcr.io** — logowanie do GitHub Container Registry
6. **Metadata** — wyznaczenie tagów obrazu
7. **Build (amd64)** — budowa obrazu tylko na amd64 do skanowania
8. **Trivy CVE scan** — skanowanie obrazu pod kątem podatności CRITICAL i HIGH
9. **Build & Push (multi-arch)** — budowa i publikacja obrazu na obie architektury (tylko jeśli skan przeszedł)

---

## Tagowanie obrazów

Obraz jest tagowany dwoma tagami jednocześnie:

- `latest` — zawsze wskazuje na najnowszy obraz z gałęzi `main`
- `sha-xxxxxxx` — unikalny tag oparty na skrócie commita Git (np. `sha-a1b2c3d`)

### Uzasadnienie wyboru tagowania

Tag `latest` zapewnia łatwy dostęp do najnowszej wersji obrazu bez znajomości konkretnego SHA. Jest standardem w środowiskach deweloperskich i testowych.

Tag `sha-xxxxxxx` oparty na skrócie commita Git zapewnia pełną identyfikowalność (traceability) — można jednoznacznie powiązać działający kontener z konkretnym stanem kodu źródłowego. Jest to zalecana praktyka w środowiskach produkcyjnych zgodnie z zasadami GitOps ([źródło: Docker docs — Image tagging best practices](https://docs.docker.com/build/ci/github-actions/manage-tags-labels/)).

Połączenie obu tagów daje kompromis między wygodą (`latest`) a bezpieczeństwem i odtwarzalnością (`sha-*`).

---

## Tagowanie cache

Dane cache przechowywane są w publicznym repozytorium DockerHub `fkinf/weather-app-cache` pod tagiem `cache`.

Użyto eksportera typu `registry` w trybie `max`, który zapisuje wszystkie warstwy pośrednie (nie tylko końcowe), co maksymalizuje skuteczność cache przy kolejnych buildach — szczególnie przy budowie wieloarchitekturowej.

---

## Skanowanie CVE — Trivy

Do skanowania obrazu użyto skanera **Trivy** (aquasecurity/trivy-action). 

Trivy został wybrany zamiast Docker Scout z następujących powodów:
- jest w pełni open-source
- działa natywnie w GitHub Actions bez dodatkowej konfiguracji konta
- nie wymaga płatnego planu dla prywatnych repozytoriów
- jest powszechnie stosowany w projektach open-source i środowiskach CI/CD

Skan sprawdza podatności sklasyfikowane jako **CRITICAL** i **HIGH**. Jeśli zostaną wykryte — pipeline zatrzymuje się i obraz **nie jest** publikowany na ghcr.io.

---

## Sekrety

W repozytorium skonfigurowano następujące sekrety:

| Nazwa | Opis |
|---|---|
| `DOCKERHUB_USERNAME` | Nazwa użytkownika DockerHub (do cache) |
| `DOCKERHUB_TOKEN` | Token dostępu DockerHub |

Token `GITHUB_TOKEN` jest dostarczany automatycznie przez GitHub Actions.

---

## Obraz na ghcr.io

Obraz dostępny jest pod adresem:
## Tagowanie cache

Dane cache przechowywane są w publicznym repozytorium DockerHub `fkinf/weather-app-cache` pod tagiem `cache`.

Użyto eksportera typu `registry` w trybie `max`, który zapisuje wszystkie warstwy pośrednie (nie tylko końcowe), co maksymalizuje skuteczność cache przy kolejnych buildach — szczególnie przy budowie wieloarchitekturowej.

---

## Skanowanie CVE — Trivy

Do skanowania obrazu użyto skanera **Trivy** (aquasecurity/trivy-action). 

Trivy został wybrany zamiast Docker Scout z następujących powodów:
- jest w pełni open-source
- działa natywnie w GitHub Actions bez dodatkowej konfiguracji konta
- nie wymaga płatnego planu dla prywatnych repozytoriów
- jest powszechnie stosowany w projektach open-source i środowiskach CI/CD

Skan sprawdza podatności sklasyfikowane jako **CRITICAL** i **HIGH**. Jeśli zostaną wykryte — pipeline zatrzymuje się i obraz **nie jest** publikowany na ghcr.io.

---

## Sekrety

W repozytorium skonfigurowano następujące sekrety:

| Nazwa | Opis |
|---|---|
| `DOCKERHUB_USERNAME` | Nazwa użytkownika DockerHub (do cache) |
| `DOCKERHUB_TOKEN` | Token dostępu DockerHub |

Token `GITHUB_TOKEN` jest dostarczany automatycznie przez GitHub Actions.

---

## Obraz na ghcr.io

Obraz dostępny jest pod adresem:
ghcr.io/fkinf/zadanie2aplikacjapogodowa:latest
