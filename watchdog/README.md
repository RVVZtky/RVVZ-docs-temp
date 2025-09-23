# Hlídací pes

### Popis UI
Když se přihlásíš na program (na stránce /events/{eventId}/programy), na který už se nevejdeš, přidá ti ho do hlídacího psa.   \
Na tvoje přihlášené programy a programy v hlídacím psovi se můžeš podívat v "Mém rozvrhu" (/events/{eventId}/moje-programy).   \
Pokud máš v jeden timeslot (programový blok) více programů v hlídacím psovi, můžeš si v "Mém rozvrhu" nastavit jejich prioritu přetahováním, neboli na který z těch programů chceš více. viz níže 

![output](https://github.com/user-attachments/assets/889f1409-784e-422b-a99c-0972be87db55)

### Popis logiky
každá RVVZtka má vypsané timesloty, v těch timeslotech jsou vypsané programy   \
teď budeme pojednávat jenom situacích v jednom timeslotu, mezi programama v různých timeslotech nejsou žádné vazby

1. Můžeš se přihlásit na program, s tím že jsi se vešel do kapacity => Jsi PŘIHLÁŠEN
2. Můžeš se přihlásit ne program a nevejít se do kapacity => program jsi si přidal do HLÍDACÍHO PSA
-> hlídací pes (z pohledu uživatele):   \
hlídá místo ve frontě mimo kapacitu (ten program může mít v hlídacím psovi víc lidí, tak hlídá pořadí těch lidí)   \
pokud se uvolní v programu místo (někdo kdo byl PŘIHLÁŠEN se odhlásil/navýšila se kapacita) tak 1. ve frontě přihlásí (pokud jsi 1. ve frontě - jsi první kdo si ten program přidal do hlídacího psa po tom co došla kapacita tak tě přihlásí, člověk za tebou teď bude 1. ve frontě)

3. v jednom timeslotu můžeš mít max 1 PŘIHLÁŠENÝ program
4. zároveň můžeš mít v tom samém timeslotu víc programů v HLÍDACÍM PSOVI

5. pokud se takhle uvolní místo v nějakém programu který máš v hlídacím psovi tak tě z přihlášeného programu odhlásí (a program z hlídacího psa máš teď PŘIHLÁŠEN)
6. jelikož můžeš mít víc programů v hlídacím psovi tak si uživatel může nastavit jeho prioritu programů v HLÍDACÍM PSOVI (viz screenshot UI) (přihlášený program bude z pohledu uživatele vždy ten s nejnižší/nejhorší prioritou).

**příklad:**

<img width="517" height="431" alt="image" src="https://github.com/user-attachments/assets/f094e519-1a52-473c-8887-928ffaf8a9bd" />

HLÍDACÍ PES:   \
nejvyšší priorita (1): program1   \
nižší priorita (2): program2   \
PŘIHLÁŠEN: program3

case 1: pokud se uvolní kapacita v program2 (a uživatel je 1. ve frontě na ten program - bude přihlášen) tak ho systém odhlásí z program3 a přihlásí na program2 (program1 zůstane v hlídacím psovi)   \
case 2: pokud se uvolní kapacita v program1 (a uživatel je 1. ve frontě na ten program - bude přihlášen) tak ho systém odhlásí z program3 i program2 (odstraní progrma2 z hlídacího psa) a přihlásí na program1 -> uživatel bude na tom programu na který chtěl nejvíc

**[Kompletní popis logiky](https://github.com/RVVZtky/RVVZ-docs-temp/edit/master/watchdog/WATCHDOG.md)** (více příkladů, generováno Claude Sonnet 3.7)