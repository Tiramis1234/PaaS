# Homelab PaaS – roadmapa 16 tygodni

**Start:** 2.11.2026 · **Koniec:** 21.02.2027

**Cel:** trzynodowy klaster k3s (HA na etcd) na bare metal z warstwą PaaS (GitOps, CI, self-service), backupami do R2 i udokumentowanymi testami awarii.

## Spis treści

- [Kontekst projektu](#kontekst-projektu)
- [Sposób pracy](#sposób-pracy)
- [Faza 1 — Fundamenty (tyg. 1–4, do 29.11)](#faza-1--fundamenty-tyg-14-do-2911)
- [Faza 2 — GitOps i dostęp (tyg. 5–8, do 27.12)](#faza-2--gitops-i-dostęp-tyg-58-do-2712)
- [Faza 3 — Platforma (tyg. 9–12, do 24.01)](#faza-3--platforma-tyg-912-do-2401)
- [Faza 4 — Operacje i portfolio (tyg. 13–16, do 21.02)](#faza-4--operacje-i-portfolio-tyg-1316-do-2102)
- [CKA równolegle](#cka-równolegle)
- [Decyzje i ryzyka](#decyzje-i-ryzyka)

## Kontekst projektu

Ta sekcja streszcza dotychczasowe ustalenia, żeby nowy asystent AI nie musiał pytać od zera.

### Autor i cel

- Junior DevOps z ok. 4 miesiącami doświadczenia, szuka pracy jako junior infra engineer.
- Projekt służy do nauki i do portfolio.
- Równolegle nauka do CKA (patrz [CKA równolegle](#cka-równolegle)). Scheduling z kursu CKA po tygodniu 1 (Ansible).

### Jak pomagać

- Rozmowa po polsku.
- Nie podawać gotowego kodu ani konfiguracji. Tłumaczyć co i dlaczego, wskazywać dokumentację, zadawać pytania kontrolne.
- Pomoc stopniowana, gdy autor utknie: podpowiedź → kierunek → fragment konfiguracji. Pełne rozwiązanie tylko na wyraźną prośbę.
- Review kodu autora: wskazywać problemy, nie przepisywać.
- Wyjątek: aplikacja demo może być pisana z pomocą AI (vibe-coding), bo jest tylko nośnikiem dla platformy.

### Sprzęt

| Node | CPU | RAM | Dysk | Rola |
|---|---|---|---|---|
| ThinkStation A | i5 6. gen | 12 GB DDR4 | SSD SATA | server k3s + workloady (Prometheus, Grafana, bazy) |
| ThinkStation B | i5 6. gen | 8 GB DDR4 | SSD M.2 | server k3s + workloady |
| Laptop | i7 4. gen | 8 GB DDR3 | SSD SATA | server k3s, głównie control plane (taint) |

- Laptop bez baterii, ruch tylko po kablu, gigabit. Do zrobienia: czyszczenie i wymiana pasty, brak usypiania po zamknięciu klapy.
- Brak swapa na nodach. iGPU z minimalnym DVMT w BIOS-ie, bez GUI.

### Sieć i ograniczenia

- Wszystkie nody w jednym niezarządzalnym switchu (plug and play, brak VLAN-ów).
- Brak dostępu do routera: DHCP bez gwarancji stałych IP, brak przekierowania portów i BGP.
- Rozwiązanie: druga statyczna podsieć L2 (10.10.10.0/24) na tych samych interfejsach dla ruchu klastra; DHCP tylko do internetu.
- Ruch z internetu wyłącznie przez Cloudflare Tunnel (limit 100 MB na request, więc registry i push z CI tylko w LAN).

### Architektura

- **Klaster:** bare metal bez Proxmoxa, k3s z wbudowanym etcd na 3 serwerach.
- **Sieć:** Cilium (CNI, Gateway API), kube-vip w trybie ARP dla VIP-a API.
- **GitOps:** Argo CD (app-of-apps, ApplicationSet), sekrety przez SOPS lub External Secrets.
- **Dostęp:** Cloudflare Tunnel → Gateway; panele admina za Cloudflare Access.
- **Backup:** snapshoty etcd 2× dziennie do Cloudflare R2 (darmowe 10 GB), retencja 14 dni + kopia lokalna; Velero → R2 dla zasobów i wolumenów.
- **Storage:** local-path + NFS; bazy aplikacji przez CloudNativePG.
- **Git i CI:** Gitea + Gitea Actions + registry Gitei.
- **Monitoring:** Prometheus + Grafana (albo VictoriaMetrics jako lżejsza alternatywa), plus zewnętrzny dead man's switch.
- **Bezpieczeństwo:** Trivy (sekrety w repo, skan hostów, obrazy w CI), wersje narzędzi i akcji CI przypięte do SHA/sum kontrolnych po incydencie supply chain Trivy z marca 2026.

### Cel końcowy: aplikacja mikroserwisowa

- **frontend:** Nuxt (SSR)
- **api:** FastAPI (uvicorn)
- **worker:** Python, przetwarza zadania z kolejki
- **dane:** Postgres przez CloudNativePG, kolejka Redis albo NATS
- **Wymagania od strony infra:** konfiguracja przez env, health endpointy, poprawne zamykanie przy SIGTERM, logi JSON na stdout, obrazy multi-stage non-root, OpenTelemetry, migracje bazy (Alembic) jako świadoma decyzja w ADR.

## Sposób pracy

### Zasady nauki

- Każdy tydzień kończy się punktem „Definition of done” — bez niego issue nie jest zamknięte.
- Na pytanie kontrolne odpowiadasz własnymi słowami w [journal.md](journal.md).
- Gotowy kod dopiero, gdy utkniesz na dłużej niż 2 sesje.
- Co 4. tydzień ma bufor: jeśli wszystko idzie zgodnie z planem, robisz w nim dokumentację.

### Przeniesienie do GitHub Projects

- **Faza** = Milestone (4 sztuki, z datą końca).
- **Tydzień** = Issue; checklista z tygodnia wkleja się 1:1 jako task list w opisie.
- **Iteration:** pole o długości 1 tygodnia, start w poniedziałek.
- **Labels:** `ansible`, `k8s`, `network`, `backup`, `security`, `gitops`, `ci`, `observability`, `docs`.
- **Kolumny:** Backlog → This week → In progress → Done.

## Faza 1 — Fundamenty (tyg. 1–4, do 29.11)

Na koniec fazy: klaster k3s z trzema serwerami i wbudowanym etcd stawia się z kodu, przeżywa awarię dowolnego noda, a snapshoty etcd lądują w R2 z przećwiczonym restore.

### Tydzień 1 — Ansible: przygotowanie nodów

- [x] Struktura repo: `ansible/`, `docs/adr/`, `docs/journal.md`, README
- [ ] Inventory z 3 nodami (2× ThinkStation + laptop), group_vars / host_vars
- [ ] Rola bazowa: hostname, strefa czasowa, NTP, pakiety, automatyczne aktualizacje
- [ ] Rola hardeningu: user z kluczem SSH, brak logowania hasłem i rootem, firewall
- [ ] Druga podsieć L2 dla ruchu klastra (statyczne IP obok DHCP)
- [ ] Przygotowanie pod k8s: swap, moduły kernela i sysctl pod Cilium
- [ ] Laptop: bateria wyjęta, brak usypiania po zamknięciu klapy, Wi-Fi wyłączone, ruch tylko po kablu
- [ ] Dysk pod etcd: na ThinkStation 8 GB katalog danych etcd na M.2
- [ ] BIOS: DVMT na minimum, auto-włączanie po zaniku zasilania (sprawdź, czy laptop to wspiera)
- [ ] ansible-lint + Trivy secret scan w pre-commit
- [ ] **Definition of done:** drugie uruchomienie daje `changed=0`; reinstalacja jednej maszyny i playbook przywraca ten sam stan

**Pytanie kontrolne:** czym różni się handler od taska i dlaczego restart SSH powinien być handlerem?

### Tydzień 2 — k3s HA z wbudowanym etcd

- [ ] k3s: 3 serwery z embedded etcd, pierwszy inicjuje klaster, kolejne dołączają
- [ ] node-ip w podsieci klastra, wyłączone flannel/Traefik/servicelb
- [ ] Rezerwacja pamięci systemowej w kubelecie na każdym nodzie
- [ ] Laptop: taint, żeby trzymał głównie control plane, a cięższe workloady szły na ThinkStation
- [ ] Pomiar latencji fsync dysków pod etcd (doczytaj, jakie wartości etcd uznaje za za wolne)
- [ ] Instalacja k3s przypięta do wersji, wszystko przez Ansible
- [ ] **Definition of done:** `kubectl get nodes` pokazuje 3 serwery; zmierzone zużycie RAM zapisane w journalu

**Pytanie kontrolne:** ile nodów może paść, zanim etcd straci quorum, i co się wtedy dzieje z działającymi podami?

### Tydzień 3 — Sieć klastra

- [ ] Cilium jako CNI, kube-proxy replacement (doczytaj wymagania)
- [ ] kube-vip w trybie ARP: VIP dla API w podsieci klastra
- [ ] Gateway API przez Cilium, testowa aplikacja z HTTPRoute
- [ ] Kubeconfig wskazuje na VIP, nie na konkretny node
- [ ] **Definition of done:** wyłączasz jeden node i API dalej odpowiada pod VIP-em

**Pytanie kontrolne:** jak drugi node dowiaduje się, że VIP przeszedł na niego, i co widzi switch?

### Tydzień 4 — Backup etcd + bufor

- [ ] Bucket R2, token z dostępem tylko do bucketu, lifecycle rule 14 dni
- [ ] Wbudowane snapshoty etcd w k3s 2× dziennie, wysyłka do R2 przez S3
- [ ] Lokalne snapshoty zostają na nodach, zgodna retencja w obu miejscach
- [ ] Restore test: odtworzenie klastra ze snapshotu z R2
- [x] Pierwsze ADR-y (001–009)
- [ ] Diagram architektury
- [ ] **Definition of done:** runbook „restore etcd” w `docs/` przetestowany od zera

**Pytanie kontrolne:** kiedy wystarczy, że jeden node dołączy ponownie do klastra, a kiedy potrzebujesz restore ze snapshotu?

## Faza 2 — GitOps i dostęp (tyg. 5–8, do 27.12)

Na koniec fazy: wszystko w klastrze jest wdrażane z Gita przez Argo CD, a aplikacje są dostępne z internetu przez Cloudflare Tunnel.

### Tydzień 5 — Argo CD

- [ ] Instalacja Argo CD i bootstrap wzorcem app-of-apps
- [ ] Struktura repo GitOps: infrastruktura vs aplikacje
- [ ] Cilium i kube-vip przeniesione pod zarządzanie Argo CD (jeśli ma to sens — opisz decyzję)
- [ ] **Definition of done:** zmiana w Gicie trafia do klastra bez `kubectl apply`

**Pytanie kontrolne:** co się stanie, gdy ktoś ręcznie zmieni zasób zarządzany przez Argo CD?

### Tydzień 6 — Sekrety + Cloudflare Tunnel

- [ ] SOPS (z age) albo External Secrets — jedno narzędzie w całym projekcie
- [ ] cloudflared w klastrze: 2 repliki, po jednej na nodzie
- [ ] Tunel zarządzany z pliku konfiguracyjnego w repo
- [ ] Tunel → Gateway, wildcard DNS w Cloudflare
- [ ] **Definition of done:** testowa apka dostępna z internetu po HTTPS, zero sekretów jawnie w repo

**Pytanie kontrolne:** co dzieje się z ruchem z zewnątrz, gdy padnie node z jedną repliką cloudflared?

### Tydzień 7 — Cloudflare Access + TLS

- [ ] Argo CD i Grafana (później) za Cloudflare Access, logowanie przez GitHuba
- [ ] Tryb SSL w Cloudflare dobrany świadomie
- [ ] cert-manager dla ruchu wewnętrznego (jeśli potrzebny — decyzja w ADR)
- [ ] **Definition of done:** panel admina niedostępny bez logowania; Kube API nigdy publicznie

**Pytanie kontrolne:** gdzie kończy się TLS na drodze użytkownik → Cloudflare → tunel → pod?

### Tydzień 8 — Storage, Gitea + bufor

- [ ] local-path + NFS jako StorageClass, decyzja w ADR
- [ ] CloudNativePG dla baz aplikacji
- [ ] Gitea z bazą w CloudNativePG, backup bazy do R2
- [ ] SSH do Gitei tylko w LAN, registry tylko w LAN (limit 100 MB w tunelu)
- [ ] **Definition of done:** repo w Gitei przetrwa restart noda; backup bazy Gitei odtworzony testowo

**Pytanie kontrolne:** jak CloudNativePG zachowa się, gdy padnie node z instancją primary?

## Faza 3 — Platforma (tyg. 9–12, do 24.01)

Na koniec fazy: deweloper dodaje repo z aplikacją, a platforma sama ją buduje, skanuje i wdraża do jego namespace'u.

### Tydzień 9 — CI

- [ ] Runner Gitea Actions w klastrze, z limitami zasobów
- [ ] Registry Gitei w LAN, pull w klastrze przez imagePullSecret
- [ ] Pipeline: build → skan Trivy (blokada przy CRITICAL) → push → bump wersji w repo GitOps
- [ ] Akcje i obrazy w CI przypięte do SHA/digesta
- [ ] **Definition of done:** commit w repo aplikacji kończy się nową wersją w klastrze bez ręcznych kroków

**Pytanie kontrolne:** dlaczego pipeline commituje do repo GitOps zamiast robić `kubectl set image`?

### Tydzień 10 — Golden path (szablon aplikacji)

- [ ] Helm chart „standardowej aplikacji”: Deployment, Service, HTTPRoute, probes, requests/limits
- [ ] Sensowne domyślne wartości: securityContext, non-root, read-only filesystem
- [ ] Skan chartu Trivy przed merge
- [ ] **Definition of done:** nowa aplikacja to tylko plik `values.yaml`

**Pytanie kontrolne:** czym różni się readiness od liveness probe i co się stanie, gdy je pomylisz?

### Tydzień 11 — Self-service i multi-tenancy

- [ ] ApplicationSet: nowy katalog / repo = nowa aplikacja
- [ ] Namespace per zespół z ResourceQuota i LimitRange
- [ ] RBAC: rola zespołu ograniczona do swojego namespace'u
- [ ] NetworkPolicy: domyślnie deny, ruch tylko przez Gateway
- [ ] **Definition of done:** użytkownik zespołu A nie widzi i nie dosięgnie zasobów zespołu B

**Pytanie kontrolne:** jak sprawdzisz uprawnienia użytkownika bez logowania się jako on?

### Tydzień 12 — Aplikacja mikroserwisowa: pierwsze wdrożenie

- [ ] Aplikacja (vibe-coding dozwolony): frontend Nuxt, api FastAPI, worker Python, Postgres (CloudNativePG), kolejka Redis/NATS
- [ ] Każdy serwis: konfiguracja przez env, health endpointy, poprawne zamykanie przy SIGTERM, logi JSON
- [ ] Dockerfile multi-stage, non-root; CI buduje obrazy Node i Python tym samym golden path
- [ ] Migracje Alembic: Job, init container czy hook Argo CD — decyzja w ADR
- [ ] NetworkPolicy: frontend → api → baza/kolejka, nic poza tym
- [ ] Dokumentacja dla dewelopera: „jak wdrożyć serwis w 5 krokach”
- [ ] **Definition of done:** cała aplikacja działa w internecie przez tunel, wdrożona wyłącznie z Gita

**Pytanie kontrolne:** co się stanie z zadaniami w kolejce podczas rolloutu workera?

## Faza 4 — Operacje i portfolio (tyg. 13–16, do 21.02)

Na koniec fazy: widzisz stan klastra w Grafanie, każda awaria ma runbook, a repo nadaje się do pokazania na rozmowie.

### Tydzień 13 — Metryki i alerty

- [ ] Prometheus + Grafana w klastrze (albo VictoriaMetrics jako lżejsza alternatywa), przypięte do ThinkStation, krótka retencja, Grafana za Cloudflare Access, data source'y deklaratywnie z Gita
- [ ] Metryki nodów, etcd i k3s
- [ ] Alerty: mało wolnej pamięci noda, ostatni udany backup > 26 h, node NotReady, etcd bez lidera, wolny fsync etcd, temperatura laptopa; zewnętrzny dead man's switch
- [ ] **Definition of done:** każdy alert wywołany celowo i odebrany

**Pytanie kontrolne:** który alert obudziłby Cię w nocy, a który może poczekać do rana?

### Tydzień 14 — Logi, Velero, bezpieczeństwo

- [ ] Loki do logów, retencja dopasowana do RAM-u i dysku
- [ ] Velero → R2: zasoby klastra i wolumeny, harmonogram z TTL
- [ ] Trivy: skan hostów z Ansible i regularny skan obrazów w klastrze (Operator tylko jeśli RAM pozwoli)
- [ ] **Definition of done:** namespace usunięty i odtworzony z Velero

**Pytanie kontrolne:** czego Velero nie odtworzy, a co odtworzy Argo CD z Gita?

### Tydzień 15 — Testy awarii (DR)

- [ ] Scenariusz 1: wyłączenie laptopa
- [ ] Scenariusz 2: wyłączenie ThinkStation z workloadami
- [ ] Scenariusz 3: wyłączenie 2 nodów naraz (utrata quorum) i powrót
- [ ] Scenariusz 4: restore etcd ze snapshotu w R2
- [ ] Scenariusz 5: odbudowa całości od zera z repo
- [ ] Dla każdego: czas odtworzenia, co poszło źle, runbook
- [ ] **Definition of done:** zmierzone RTO i RPO wpisane do README

### Tydzień 16 — Wydanie v1.0 + portfolio

- [ ] Wydanie nowej wersji aplikacji przez canary (Argo Rollouts) z automatycznym rollbackiem na podstawie metryk
- [ ] Tracing OpenTelemetry przez frontend → api → worker (jeśli backend trace'ów zmieści się w RAM-ie)
- [ ] Dashboard SLO per serwis: błędy i latencja
- [ ] README: cel, diagram architektury, decyzje, ograniczenia, jak uruchomić
- [ ] Komplet ADR-ów i runbooków
- [ ] Krótkie nagranie demo: commit → canary → aplikacja w internecie; awaria → odtworzenie
- [ ] Retrospektywa: co zrobiłbyś inaczej z większym budżetem
- [ ] **Definition of done:** projekt da się wytłumaczyć w 5 minut na rozmowie

## CKA równolegle

k3s ukrywa część komponentów, które są na egzaminie, więc instalację klastra i naprawę control plane'u ćwicz osobno na kubeadm (np. na VM-kach na laptopie albo w symulatorze egzaminu). Reszta tematów pokrywa się z projektem:

| Tydzień projektu | Temat CKA |
|---|---|
| 2–3 | Scheduling: taints, tolerations, affinity, anti-affinity, requests/limits |
| 3, 6 | Networking: CNI, Services, Gateway API, DNS w klastrze |
| 4, 14 | Backup i restore etcd (na k3s plus osobno na kubeadm, bo egzamin używa etcdctl) |
| 8 | Storage: StorageClass, PV, PVC |
| 10 | Workloady: Deployment, rollout, probes, ConfigMap, Secret |
| 11 | Bezpieczeństwo: RBAC, ServiceAccount, NetworkPolicy, quoty |
| 13–15 | Troubleshooting: node NotReady, pody w CrashLoop, logi i eventy |

Scheduling możesz ćwiczyć na swoim klastrze od tygodnia 3, np. wymuszając rozłożenie replik cloudflared na dwa nody.

## Decyzje i ryzyka

Podjęte decyzje są opisane w [rejestrze ADR](adr/README.md).

| Ryzyko | Skutek | Mitygacja |
|---|---|---|
| Awaria 2 nodów naraz | utrata quorum, API klastra stoi | snapshoty w R2 + runbook restore |
| Brak swapa | OOM killer zabija etcd lub kubeleta | rezerwacja w kubelecie, limity na podach, alert na pamięć |
| Wolny dysk na jednym nodzie | wolne zapisy etcd, timeouty API | pomiar fsync w tyg. 2, alert na latencję etcd |
| Brak baterii w laptopie | zanik prądu wyłącza wszystkie 3 nody naraz | auto-start po zasilaniu, test zimnego startu klastra |
| Limit 100 MB w tunelu | push dużych obrazów się nie udaje | registry i CI tylko w LAN |
| Przekroczenie 10 GB w R2 | koszt bez twardego limitu | lifecycle rule, alert na rozmiar bucketu |
| Zmiana IP z DHCP | utrata dostępu Ansible po SSH | inventory na adresach ze statycznej podsieci |
| Opóźnienie przez CKA | przesunięcie faz | bufor w tygodniach 4, 8, 12 i 16 |
