# hl7-v2-validator ✅

**Walidator wiadomości HL7 v2 (oparty o BridgeLink / Mirth Connect)**

Ten projekt powstał na potrzeby Projecthaton HL7 Polska i zawiera przygotowany obraz BridgeLink z niezbędnymi konfiguracjami i zasobami.

---

## Spis treści
- [Opis](#opis)
- [Przykładowy komunikat ACK HL7](#przykładowy-komunikat-ack-hl7)
- [Szybkie uruchomienie (Docker Compose)](#szybkie-uruchomienie-docker-compose)
- [Konfiguracja i sekrety](#konfiguracja-i-sekrety)
- [Pliki i lokalizacje w repozytorium](#pliki-i-lokalizacje-w-repozytorium)
- [Licencja](#licencja)

---

## Opis
Projekt przygotowuje instancję BridgeLink z gotową konfiguracją i zasobami do walidacji HL7 v2 oraz integracji z serwerem terminlogogii FHIR. Zawiera skrypty automatyzujące instalację, przywracanie konfiguracji oraz opcjonalne pobieranie rozszerzeń i dodatkowych bibliotek.

W ramach deploymentu tworzone są trzy usługi:
- `HL7v2 Validator - UTF-8` - domyślnie działa na porcie 6661 i jest przeznaczona dla walidacji komunikatów kodowanych w UTF-8
- `HL7v2 Validator - CP1250` - domyślnie działa na porcie 6662 i jest przeznaczona dla walidacji komunikatów kodowanych w CP1250
- `CDA View API` - domyślnie działa na porcie 8081 i jest przeznaczona do podglądu dokumentów, przekazanych jako załącznik

Usłgi HL7 z założenia komunikują się w wersji standardu 2.5.1 (ACK aplikacyjne). Niemniej, bramki przyjmują każdy komunikat w wersji v2.

---

## Przykładowy komunikat ACK HL7
Poniższy przykład przedstawia komunikat potwierdzający typu ACK^O01, generowany przez bramkę walidacyjną HL7. Komunikat zawiera standardowe segmenty potwierdzenia oraz rozszerzone informacje walidacyjne dotyczące poszczególnych segmentów wiadomości źródłowej.

```hl7
MSH|^~\&|HIS|HOSPITAL|LIS|LAB|20260106181313548||ACK^O01|VALHL7PL20260106181313548|P|2.3.1|||||||
SFT|HL7 Poland|1.0.0|HL7v2 Validation Engine|hl7-v2-validator:1.0.0|LOINC Projectathon
MSA|AA|123456|Validation success
ERR||OBR^1^4^^1|0^Message accepted^HL70357|I|||LOINC display: Glucose [Mass/volume] in Blood --2 hours post dose glucose|Informacja o kodzie LOINC
ERR||OBX^1^5^^5|0^Message accepted^HL70357|I|||CDA saved as: e16e8054-0f34-4ef3-9fcc-5d80f7e0235c.xml|Załącznik CDA przekazany do zapisu
```

Segment MSA określa wynik przetwarzania komunikatu źródłowego.  
Segmenty ERR zawierają szczegółowe informacje dotyczące walidacji poszczególnych segmentów komunikatu źródłowego. W prezentowanym przykładzie wszystkie wpisy mają charakter informacyjny.

---

## Szybkie uruchomienie (Docker Compose) 🚀
Plik: `docker-compose.yml` (serwisy `bl` i `postgres`). Repo zawiera też skrypt testowy:

```bash
./hl7v2validator-docker-test.sh
# lub ręcznie
docker build -t hl7-v2-validator:latest .
docker compose up
```

Dostęp do GUI BridgeLink: https://localhost:8443  
Dostęp do usługi podglądu dokumentów CDA: http://localhost:8081/cda/

Porty expose'owane przez kontener: **8443** (opcjonalnie - UI BridgeLink), **6661**, **6662**, **8081**.

> Uwaga: `docker-compose.yml` używa Docker Secrets z katalogu `secrets/` (np. `admin_password.txt`, `fhir_client_id.txt`, `fhir_client_secret.txt`, `postgres_user.txt`, `postgres_password.txt`).

---

## Konfiguracja i sekrety 🔐
- Sekrety są zdefiniowane w `docker-compose.yml` i znajdują się w katalogu `secrets/`.
- `entrypoint.sh` automatycznie:
  - mapuje zmienne `MP_*` do `/opt/bridgelink/conf/mirth.properties`,
  - ustawia `server.id` jeśli `SERVER_ID` jest podane,
  - pobiera `CUSTOM_PROPERTIES`, `CUSTOM_VMOPTIONS`, `EXTENSIONS_DOWNLOAD`, `CUSTOM_JARS_DOWNLOAD` jeśli ustawione,
  - ustawia hasło administratora z `ADMIN_PASSWORD_FILE` oraz zapisuje FHIR credentials z `FHIR_CLIENT_ID_FILE` i `FHIR_CLIENT_SECRET_FILE`,
  - przy pierwszym starcie przywraca konfigurację z `scripts/config/Projectathon_HL7_LAB_Gateway.xml` przez REST API.

Jeśli źródła pobierania mają self-signed certy, użyj `ALLOW_INSECURE=true`.

---

## Pliki i lokalizacje w repozytorium 🔍
- `Dockerfile` — budowa obrazu (instalacja JDK, instalacja BridgeLink przez `scripts/install.sh`).
- `docker-compose.yml` — przykładowy stack (BridgeLink + Postgres, secrets i volumes).
- `hl7v2validator-docker-test.sh` — buduje obraz i uruchamia `docker compose up`.
- `scripts/install.sh` — pobiera i instaluje BridgeLink w `/opt/bridgelink`.
- `scripts/entrypoint.sh` — logika startowa: merge konfigów, ustawienie hasła, restore configu, pobieranie rozszerzeń/jarów.
- `scripts/config/Projectathon_HL7_LAB_Gateway.xml` — konfiguracja reguł walidacji, funkcji, skryptów, kanałów (używana przez entrypoint).
- `scripts/config/CDA_PL_IG_1.3.2.xsl` — przykładowy XSL do walidacji.

---

## Licencja
Repo zawiera komponenty BridgeLink (licencja Mozilla Public License 2.0).

---