---
layout: post
title: "Konverter din feedback loop til en kontinuerlig diskusjon med review-miljøer"
categories: DevOps
---

> [!summary] tl:dr - Hopp til headingen **Forutsetning** for å starte rette på hvordan og hva som skal til for å få satt opp review-miljøer.

En av de vanlige problemene som utviklere møter er hvordan de skal teste kodeendringene sine før de merger det inn i hoved branchen. Testing på en lokal maskin eller et delt utviklings-miljø kan være upålitelig, tidkrevende og utsatt for feil. Utviklere kan støte på problemer som uforenlige avhengigheter, motstridende konfigurasjoner eller manglende data. Videre kan testing på en delt server forårsake forstyrrelser med andre utvikleres arbeid eller påvirke ytelsen til serveren. Disse smertepunktene kan føre til frustrasjon, forsinkelser og dårlig kodekvalitet.

Ville det ikke vært fantastisk om alle på teamet, med all deres mangfoldige ferdigheter og kunnskap om ulike deler av en oppgave, kunne jobbe med den samtidig? All kompetansen er nødvendig for å få oppgaver fra idé til produksjon. Det betyr at oppdeling av arbeidet over tid bare gjør at det tar lengre tid å få det ferdig. Review-miljøer gjør det mulig for alle på teamet å jobbe med oppgaven uten å hindre hverandres arbeid, noe som øker hastigheten betydelig. Hvis dette høres interessant ut, les videre. I denne artikkelen forsøker jeg å beskrive fordelene ved å benytte review-miljøer.

## Nøyaktig hva er et Review-miljø?

På engelsk kan *Review-miljøer* også være kjent som review environments, ephemeral environments eller preview environments. Et review-miljø er en fullstendig distribusjon av en applikasjon, som representerer de valgte endringene som er gjort i en branch koblet til en Pull Request (PR). Den lever sammen med PRen og gir en måte å se endringene som er gjort i den PRen, fra sluttbrukerens perspektiv. Utrullingen av review-miljøene gjøres ved hjelp av samme automatisering som brukes ved utrulling til produksjon.

![](/media/Review%20app%20branch%20strategy.excalidraw.png)

## Fordelen

Den største fordelen med review-miljøer er teammedlemmenes tilfredshet. Dette kommer av en teamdynamikk der alle medlemmene på teamet kan samarbeide om samme funksjon samtidig. Dette reduserer kognitiv belastning betraktelig ved å redusere kontekstbytter. Resultatet er raskere utvikling, og et team som får en følelse av å oppnå noe sammen.

Review-miljøer muliggjør **parallellisering av utvikling og tilbakemelding**. Utviklere kan fikse feil når problemet er friskt i minnet. Når utvikleren tenker at en løsning er oppnådd, pushes koden og det lages en PR med et review-miljø. Samtidig som utviklerene skriver noen unit-tester, sjekker en tester om problemet er løst i review-miljøet samtidig som en annen utvikler vurderer koden. Tilbakemeldingen blir derfor gitt samtidig som utvikleren fortsatt jobber med koden. Dette har i flere tilfeller resultert i tilbakemeldinger som er blitt diskutert og fikset i løpet av minutter etter at koden først ble skrevet.

Å få flere til å **teste** ny funksjonalitet er viktig for å få flere perspektiv på den valgte løsningen og få sjekk at det løser problemet før brukere får se det. Normalt er dette gjort når koden er publisert til testmiljøet. Dette skaper et tidsgap mellom når utvikleren skriver koden og når den testes. Så opprettes nye oppgaver for å fikse de nylig introduserte feilene. Det kan gå mer enn 2 uker før utvikleren får tilbakemelding på om funksjonen fungerer eller ikke. På dette tidspunktet er hele utviklerens kognitive kapasitet opptatt med en ny oppgave. Ved å bruke review miljøer kan denne tilbakemeldingen gis samtidig som koden er i review. Testere, funksjonelle arkitekter, sikkerhetstestere, produkteiere og til og med superbrukere kan gi tilbakemelding og ha en mulighet til å diskutere om denne versjonen av koden løser problemet. Dette gir den ekstra fordelen med åpenhet og involvering av interessenter. Men den viktigste fordelen er at utvikleren kan fikse tilbakemeldingen samtidg som han har koden friskt i minnet.

Utviklere som går igjennom koden kan **kontekstualisere endringene i koden**. Før vi hadde review-miljøer, måtte utviklere som skulle se igjennom koden, stoppe det de holdt på med, klone koden de skulle vurdere og kjøre denne koden lokalt for å se på resultatet av endringene. Dette betydde normalt å stenge en kjørende backend- og frontend-utviklingsserver. Noen ganger måtte den lokale databasen tømmes og seedes på nytt. Før du starter det hele igjen på grenen som måtte gjennomgås. Dette kan ta rundt 20 minutter avhengig av hvilke endringer som er gjort i koden. Det er for mange deler av prosessen der irrelevante feil kan introduseres. Og selv om alt gikk bra, må du fortsatt gjøre hele prosessen på nytt for å få eget arbeidet i gang igjen. Resultatet er ofte at dette ikke ble gjort i det hele tatt og koden ble godkjent uten å teste at den gjorde det den skulle. Med et review-miljø er denne prosessen så enkel som å klikke på en knapp i PR-en.

### Bonusfordeler

Å gjøre **demoer** er viktig for å gi interessenter innsikt i tilstanden og fremdriften i applikasjonsutviklingen. Men det kan også blokkere utvikling når dev miljøet ikke kan merges inn i fordi det brukes til å vise en demo. Selvfølgelig kan du begrense demoer til å bruke testmiljøet. Men det er alltid kulere å vise frem de nyeste og beste funksjonene enn et gammelt og kjedelig testmiljø. Hele dette problemet er fjernes fullstendig når du kan spinne opp review miljøer basert på hvilken som helst commit i hvilken som helst gren du vil. Review miljøene er isolerte og blokkerer ikke annen utvikling. Under demoer fokuserer man på å vise "happy path". Så det er mulig å kombinere flere fremtidige brancher til ett review miljø som er et perfekt glimt inn i den nære fremtiden for applikasjonen. Bare sørg for å holde deg på "happy path" under demoen.

## Forutsetninger

Å lage review-miljøer betyr at N antall unike miljøer kommer til å bli opprettet og oppdatert hundrevis av ganger om dagen. Det sier seg selv at bygging, test og deployment må være automatisert. Dette er en liste over krav som må være på plass når du setter opp reiview-miljøer for en nett-applikasjon.

- Helautomatisk bygging og distribusjon
- Dupliserbar database med testdata
- Automatiserte databasemigreringer
- Raske deployments
- Idempotent deployments
- Automatisert nedriving av miljøer
- DNS-sone med et * jokertegn eller dynamisk oppretting av DNS-undersoner
- Automatisk oppretting av sertifikater for å aktivere HTTPS (letsencrypt)
- Konfigurasjon må kunne settes under deployment

## Hvordan opprette et review-miljø?

Et review-miljø opprettes ved å deploye et miljø med et navn som er unikt for PRen. Dette miljøet kan nås på en unik URL som deretter gjøres lett tilgjengelig i PRen. Deployment utløses i pipeline som går når kode dyttes til branchen som er koblet til PRen. Det unike navnet genereres i pipeline basert på branch navn og en UUID. Denne verdien vil være den samme igjennom hele levetiden til PRen. Den kan derfor brukes som input til det idempotente skriptet som skal opprette et nytt review-miljø hvis det ikke eksisterer og oppgradere miljøet hvis det eksisterer. Følgende kode er et eksempel på en pipeline-jobb i Gitlab som distribuerer et review-miljø ved hvert push til en PR som er satt til å merge med main branchen.

```yml
deploy_review:
  stage: review
  script:
    - ./run ci_deploy review $CI_ENVIRONMENT_SLUG
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_ENVIRONMENT_SLUG.review.example.com
    on_stop: stop_review
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: never
    - when: on_success
```

## Hvordan stoppe et review-miljø

Det er god praksis å stoppe et review-miljø når det ikke lenger er nødvendig. Fordi disse miljøene er opprettet for hver PR, kan antallet miljøer akkumuleres raskt og bruke betydelige ressurser.

Å stoppe et review-miljø gjøres ved å bruke et skript som tar inn det unike navnet fra PR-en. Dette navnet er tilgjengelig som miljøvariabelen CI_ENVIRONMENT_SLUG i Gitlab sine pipelines. Med dette kan skriptet avinstallerer review-miljøet som er unikt for den PR-en. Følgende kode viser en jobb som utløses når PR-en merges.

```yml
stop_review:
  stage: review
  script:
    - ./run ci_un_deploy review $CI_ENVIRONMENT_SLUG
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: never
    - when: manual
```

## Sammendrag

Bruk av review miljøer gjør det mulig for teamet å akselerere utviklingen. Hele teamet kan bidra med sin ekspertise til hver endring av kildekoden. Dette skaper et miljø som muliggjør mer korrekt tilbakemelding til rett tid. Å være en del av review prosessen er engasjerende og fremmer en følelse av eierskap. Evnen til å engasjere legger ofte til rette for mer konsekvent involvering i et prosjekt som fører til en bedre forståelse av endringer for alle involverte.

## Kilder

- [https://docs.gitlab.com/ee/ci/review_apps/](https://docs.gitlab.com/ee/ci/review_apps/)
- [https://docs.gitlab.com/ee/ci/environments/index.html](https://docs.gitlab.com/ee/ci/environments/index.html)
