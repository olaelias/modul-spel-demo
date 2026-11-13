# Spelprojekt Mall – sessionssammanfattning och arbetsplan (v2)

*Underlag för att fortsätta arbetet i en ny Claude-session. Ladda upp detta dokument tillsammans med den senaste spelfilen (just nu `Mall10f.html`). Ingen annan bakgrund behövs. Detta dokument ersätter v1 helt.*

**Versionshistorik**
- **v1** (Mall0-stadiet): enbart planering, ingen kod ändrad.
- **v2** (denna, vid `Mall10f.html`): Fas 0 och Fas 1 är genomförda, ruinstaden och flygbanan är byggda, och planen är omskriven för föremål, kroppsmall/designläge och flip-book.

## 1. Syfte och hur dokumentet används

Elias bygger ett retroinspirerat äventyrsspel i en enda HTML-fil (canvas, vanilla JS), som startade från mallen `Mall0.html` och nu är `Mall10f.html` (ca 55 KB). Arbetsspråk: svenska. Elias vill ha en plan och förankrade designbeslut före större kodändringar. Nästa session kan fortsätta med konceptdiskussion eller kodning.

**Arbetssätt:** gör riktade redigeringar (`str_replace` eller skript med `assert` på unik träff) i de taggade sektionerna i stället för att skriva om hela filen. Leverera en körbar fil efter varje steg, med nytt filnamn per steg (Mall10d, 10e, 10f …), och syntaxkontrollera med `node` innan leverans. Elias hämtar ibland konceptidéer från en annan AI (Gemini) för att spara tokens till själva programmeringen. Sådan input är förslag och ska bedömas kritiskt.

**Så hålls dokumentet jämförbart (gäller varje ny version):**
- Sektionsnumren 1–10 och sektionsrubrikerna ligger fast, så att v1, v2, v3 … kan jämföras sektion för sektion. Nya sektioner läggs till sist (11, 12 …).
- Varje beslut har ett **beslut-ID** (B1, B2 …) i sektion 11. ID:n återanvänds aldrig. Ett ändrat beslut får ett nytt ID som anger vilket det ersätter, och det gamla markeras `[ersatt av Bx]` i stället för att tas bort.
- Faser och kontrakt har status: **[KLAR]**, **[PÅGÅR]**, **[PLANERAD]**, **[FÖRSLAG]** (väntar på Elias bekräftelse).
- Vid ny version: uppdatera versionshistoriken, uppdatera status, lägg nya beslut sist i sektion 11 och flytta besvarade frågor i sektion 9 till beslutsloggen.

## 2. Spelfilen (nuläge: Mall10f.html)

Allt ligger i en fil och är uppdelat i taggade sektioner (sök på `[TAGG]`). Regel: nytt innehåll läggs i `[REG:*]`-register eller i egna `[SCENE:*]`-sektioner, och kärnan (`[CORE]` till `[UI]`) ska sällan behöva röras. Taggarna som finns: `[CORE] [INPUT] [RNG] [STATE] [SAVE] [AUDIO] [REG:SFX] [REG:TRACKS] [GFX] [UI] [REG:BLOCKS] [REG:PORTALS] [REG:NPCS] [REG:ENEMIES] [REG:SCENES] [REG:GEN] [SCENE:TITLE] [SCENE:WORLD] [SCENE:ROOM] [SCENE:TOWN] [SCENE:CAVE] [SCENE:RUINS] [SCENE:AIRFIELD] [BOOT]`.

- **Kärnan:** canvas och skalning (`setSize`, `fit`), scenbyte `go(id,p)` (autospar vid varje scenbyte, utom titel), fast tidssteg 1/60 s, pausmeny (ENTER; på världskartan *Fortsätt / Spara till fil / Ladda från fil / Till titelskärmen*, i andra scener *Fortsätt / Återgå till kartan*). Musstöd finns i menyer och dialogrutor.
- **Slump:** `mulberry32`, `seedOf(base,tag)`, `rngFor(tag)`. Allt som sparas genereras en gång från `G.seed`.
- **Tillstånd (`G`, ren JSON):** `{v, seed, gold, lv, exp, hp, lastPortal, skip, flags, maps}`. `G.maps[id]` håller den genererade layouten för `world`, `town`, `cave`, `ruins` och `airfield`. Blockkartor sparas som strängar med en **append-only teckentabell** (`CODE`). `lastPortal` är ett koordinat-ID (`"x,y"`).
- **Spara/ladda [KLAR]:** `SAVE_V=6`, `MIGRATIONS`, `migrate()`, JSON-fil (export/import) och localStorage-autospar. localStorage fungerar inte i claude.ai:s förhandsvisning, men fungerar i en egen fil.
- **Nivåer och strid [KLAR]:** `hpMax` (30 → 600), `heartU` (HP per visat hjärta, högst 8 hjärtan), `stats(G)` (`hp, atk, def, rate, spd, str`), `dealt(A,D) = A²/(A+D)` (minst 1 skada), `power(G) = rate·hp·(atk+def)`, `zoneF(L) = 1+0,35(L−1)`, `needExp(L)` (nivå 80 är max, kurvan blir lodrät), `zoneExp`, `gainExp`.
- **Spelarens attack:** vit magisk sfär i åtta riktningar, autofire vid hållen knapp (Space/Z), ingen attack i städer. Fiender blinkar vid träff och lätta fiender knuffas (tyngre knuffas inte, `wt` mot `str`). Besegrade fiender kan tappa ett hjärta.
- **Fiender [REG:ENEMIES]:** sex handgjorda typer: `blob`, `brute` (tung), `shooter`, `fladdrare`, `stenvakt`, `eldspott`. De har `hp/atk/df/exp/wt/hw/drop`, skalas med zonnivån vid varje besök och slumpas färskt (inga dödlistor).
- **Scener:**
  - `title` (nytt spel / ladda).
  - `world` (60×40 rutor à 32 px, portaler `S/G/R/F`, frågan "Besöka …?" visar zonnivån).
  - `town` (fredlig, NPC:er: elder, trader, villager; handelsmenyn är tom).
  - `cave` (16×12 rutor, 8 kistor som solida block, zonnivå 1, fiender 3 blob + 1 brute + 2 shooter).
  - `ruins` (sidscroll, 140 kolumner, hopp och magi åt höger/vänster som i Mega Man, egna fiender: fladdrare, stenvakt, eldspott).
  - `airfield` (vertikal flygning 480×640, 60–120 s).
- **Flygbanan [KLAR tills vidare]:** grå berg med skuggning, vattenband (floder och sjöar, ofarliga) över grön mark, stilla luftballonger (guppar, skugga), zeppelinare som glider in från sidan på två höjdnivåer (ungefär hälften som förut, hälften högre upp; höjden härleds ur tiden, sparformatet är oförändrat) med ett dovt ekande ballongljud (`SND.boing`) som varning, och ett spelarplan med rundad flygkropp, 3D-skuggning och en planformad skugga på marken. Kollision mot berg, ballonger och zeppelinare kostar ett hjärta. Spelaren kan inte skjuta här.
- **Kvarvarande TODO i koden:** `[TODO:BOSS]` (bossplats i grottan). `[TODO:COLLISION]` på flygbanan är genomförd som träff = ett hjärta.

## 3. Projektmål (Elias vision)

1. **Flip-book-filosofi:** fiender och spelfigur byggs av utbytbara moduler (huvud, kropp, ben m.m.) som kombineras proceduriellt. Delarna påverkar även stats och rörelsemönster. Monster behöver inte ha alla delar (se B22).
2. **Fundament före innehåll:** systemen byggs först, så att sparformat och spelstate inte behöver göras om senare.
3. **Balansmekanism:** fiendekombinationer får aldrig bli för starka eller svaga i förhållande till spelfigurens attack, HP och skydd, oavsett hur delarna kombineras, och balansen ska räkna in utrustningen.
4. **Strid, föremål och handel** i alla relevanta scentyper, boss i grottan, fler miljöer.
5. **Slutmål:** besegra en mycket svår grotta som alltid är tillgänglig men kräver mycket EXP (se B23).

## 4. Beslut som Elias har fattat

Sammanfattning. Fullständig lista med ID:n står i sektion 11.

- **Värld och sparning:** persistent värld per session (hela layouten sparas), fiender och kistor återskapas vid varje besök (grinding), spara från pausmenyn på världskartan plus autospar vid scenbyte, JSON-fil och localStorage.
- **Spelfigur:** val mellan slump och designläge vid nytt spel, och designläget finns även i pausmenyn (kläder är bara visuella).
- **Strid och nivåer:** magisk sfär, EXP och nivåer upp till 80, stridsformel A²/(A+D), zonnivå per grotta/ruin, EXP skalas inte upp för lätta fiender.
- **Föremål:** Robin Hood-system med föremål på kroppsdelar, vikt och styrkekrav (mallfunktion först), källor är handel, kistor och bossar, obegränsad ryggsäck som scrollande lista, och inventariemenyn återanvänder kroppsmallen.
- **Fiender:** flip-book-delar där monster inte behöver ha alla delar. De befintliga sex översätts när fasen kommer.
- **Mål:** en mycket svår, alltid tillgänglig slutgrotta. Slumpade grottor och ruinstäder med olika svårighet och migrering av karaktären till en ny värld kommer senare.

## 5. Idéer för senare (ej beslutade i detalj)

- **Hemliga områden:** vissa flygplatser länkar till en hemlig stad, grotta eller ruinstad man når vid landning, på en egen liten "ö" som ser ut som en del av världskartan men inte kan besökas på annat sätt. Vissa grottor kan ha en hemlig utgång dit. Hemliga områden behandlas som vilken scen som helst, och "återgå till kartan" tar spelaren till den portsymbol som ledde dit.
- **Slutbossen i en ny generation:** gömd i en grotta som bara finns efter att man flugit från en viss flygplats (se B23).
- **Migrering:** högnivåkaraktären förs över till en ny världsgenerering med nytt innehåll och högre svårighet.
- **Bonusar på flygbanan:** guld och liknande som plockas upp under flygningen (B24).
- **Rekommenderad spelarnivå** per grotta och ruin kan visas vid porten (finns redan som zonnivå).
- **Strid på flygbanan:** inte beslutat. Den är idag en ren undvikbana.

## 6. Tekniska kontrakt (status och nya)

Gamla kontrakt (numrering från v1):
1. **Spara den genererade layouten, inte bara seeden** [KLAR]: `G.maps`, append-only `CODE`.
2. **Versionerat sparformat** [KLAR]: `SAVE_V=6`, `MIGRATIONS`. Varje ny sparbar data (t.ex. `G.look`, `G.inv`, `G.eq`) kräver ett nytt versionssteg och en migrering.
3. **Portal-ID** [KLAR]: `G.lastPortal` är en koordinat. `WORLD.pt` mappar koordinat → typ.
4. **Sparpunkt** [KLAR, B6]: pausmenyn på världskartan plus autospar vid scenbyte.
5. **Spara genom, inte härledda värden** [PLANERAD]: utseende, föremål och utrustning sparas som delnamn och bas-ID, aldrig index, och stats räknas ut från registret vid körning. Gäller `G.look`, `G.inv`, `G.eq`.
6. **Skadeenhet** [ERSATT av B7]: i stället för fasta kvartshjärtan finns en fri HP-skala och `heartU` avrundar visningen till högst 8 hjärtan.
7. **Statmodell** [DELVIS]: `stats()` har `hp, atk, def, rate, spd, str`. Hitboxen härleds ännu inte från delar (`hw`, `hh` ligger i fiendedata).
8. **Fiendedata per instans** [DELVIS]: instansen bär `hp/atk/df` skalade vid spawn, men `ai`/`draw` läser fortfarande `this.sp` och `this.c` från registret. Löses i flip-book-fasen.
9. **Ritkontrakt för delar** [PLANERAD, Fas 2]: se nedan.
10. **`power(G)`** [KLAR som funktion]: ska utökas med utrustning och användas av balansverktyget.

Nya kontrakt:
11. **Kroppsmall [PLANERAD, Fas 2]:** varje kroppsdel (hår, huvud, överkropp, ben/kjol, händer, fötter) ritas i lokala koordinater med fast origo vid fötterna och deklarerade ankarpunkter (hals, höft, händer, fötter), så att delar kan blandas fritt. Samma mall används av designläget, av inventariemenyn (föremål placeras på ankarpunkterna) och av fienderna (som använder en delmängd). `GFX.person` är basen.
12. **Utseende [PLANERAD]:** `G.look = {hair, top, bottom, colors…}` som delnamn. Slumpat utseende vid nytt spel sparas explicit. Kläder är enbart visuella (B25).
13. **Föremålsmodell [FÖRSLAG, Fas 3]:** föremål är data i ett register `[REG:ITEMS]` med `slot`, `weight`, `req` (styrkekrav), `stats` (multiplikatorer eller tillägg) och `fx` (statuseffekt, t.ex. för skor). Lagring i `G.inv` (lista, obegränsad, scrollande) och `G.eq` (per plats). Platserna är `head` (mössa/hjälm), `torso` (väst/tröja/brynja), `weapon` (stav eller närstridsvapen), `hand` (ring/handske) och `feet` (skor), men det slutgiltiga antalet bestäms senare (se fråga 9.9). Stats räknas ut ur `G.eq` vid körning. Föremål som kräver mer styrka än spelaren har kan inte tas på (blockerat), och bärvikten minskar rörlighet och styrka lite. Vikt och krav är mallvärden först.
14. **Källor för föremål [PLANERAD]:** handel i staden (`NPCS.trader`), slumpade kistor i grottrum och ruinstad (kistorna finns redan som solida block), och en kista som en besegrad boss tappar.

## 7. Fasplan

- **Fas 0, kontrakt [KLAR]:** kartor i `G`, versionering, spara/ladda, portal-ID, HP-skala, statmodell, `power(G)`.
- **Fas 1, stridskärna [KLAR]:** spelarattack, knuff, hjärtdropp, EXP och nivåer, tung och skjutande fiende, dialogförbättringar och mus. **Fas 1c [KLAR]:** strid i ruinstaden (hopp och magi) och en kollisionsbaserad flygbana.
- **Flygbanan v1 [KLAR tills vidare]:** berg, vatten, ballonger, zeppelinare med ljud, nytt spelarplan. Bonusar kommer senare (Fas 8).
- **Fas 2, kroppsmall och designläge [PLANERAD, nästa]:** kroppsmall med ankarpunkter (kontrakt 11), delregister för spelarens utseende (hårtyper, överkroppar, byxor/kjol, färger), `G.look`, "Starta med designläge" eller slump vid nytt spel, designläge även i pausmenyn, och ett sparformat v7 med migrering.
- **Fas 3, föremål och inventarium [PLANERAD]:** `[REG:ITEMS]`, `G.inv`/`G.eq`, inventariemeny (kroppsmallen med platser plus scrollande ryggsäck), mallvikt och styrkekrav, `stats()` som läser utrustning, handlarmenyn, slumpade kistor i grottrum och ruinstad.
- **Fas 4, flip-book för fiender [PLANERAD]:** delregister (huvud, kropp, ben, enstaka delar), översättning av de sex nuvarande fienderna (blob = bara kropp, eldspott = bara huvud, fladdrare = enstaka del), färgvariation, fiendegenerator med budget, hitbox ur delar, och att fienderna läser från instansen.
- **Fas 5, balansverktyg [PLANERAD]:** headless-simulator över alla delkombinationer mot spelartyper (se sektion 8). Det kan köras i Claudes sandlåda.
- **Fas 6, boss och bossdropp [PLANERAD]:** boss i grottan (`[TODO:BOSS]`) och en kista med föremål som tappas.
- **Fas 7, flera grottor och ruinstäder samt slutmålet [PLANERAD]:** slumpade zoner med olika svårighet på världskartan, en slutgrotta som alltid är tillgänglig, och vilka uppnåenden som sparas.
- **Fas 8, flygbanans bonusar och hemliga zoner [PLANERAD]:** guld och liknande att plocka upp, hemliga öar och hemliga utgångar.
- **Fas 9, migrering och polish [PLANERAD]:** migrering av högnivåkaraktären till en ny världsgenerering med svårare slutboss, plus feedback (skärmskak, partiklar), mer ljud och musik och balansering.

*Ordningen mellan Fas 2 och 3 är ett [FÖRSLAG]: kroppsmallen byggs först eftersom både designläget och inventariemenyn behöver den.*

## 8. Balansmodell (förslag, uppdaterad)

Bygg balansen in i genereringen i stället för en vakt som avvisar kombinationer:

- **Styrka** definieras av `power(G) = rate·hp·(atk+def)`. Skadeformeln är `dealt = A²/(A+D)`. Utrustning går in som tillägg eller multiplikatorer i `stats()`, så att allt balanserande använder en enda funktion.
- Delar ger **multiplikatorer** på fiendens stats (t.ex. tunga ben: hp ×1,4, spd ×0,6) och kostar **budget**. Budgeten beror på zonnivån via `zoneF(L)`. Fiendens värde i förhållande till spelarens `power` ska vara jämförbart mellan zoner.
- **Normalisering:** efter slumpningen skalas en fri stat (oftast hp) så att styrkan hamnar i målbandet (±10 %).
- **Hårda gränser:** ingen fiende dödar spelaren på färre än N träffar, och spelaren ska döda en typisk fiende på ungefär 1–8 s. Spelare under rekommenderad styrka kan svimma av en träff, men det ska vara spelbart för den envise (Princess Maker 2-känsla).
- **EXP:** lätta fienders EXP skalas inte upp, och svårare zoner ger mer EXP i förhållande till styrkan som krävs (`EXPK=1`, höj om testet visar behov).
- **Utrustning och vikt:** ett föremål ska kosta något. Tunga föremål ger mer skydd eller skada men sänker rörlighet och kräver styrka, så att inget föremål är ett rent val.
- **Uttömmande test:** med få delar blir antalet kombinationer litet (6×6×6 = 216). Simulatorn kör alla kombinationer mot spelartyper (nivå × utrustning) och flaggar avvikare. Delarna ska visuellt avslöja egenskaper (horn = skada, tjocka ben = tåligt).

## 9. Öppna frågor

Besvarade sedan v1 (flyttade till sektion 11): spara-var (B6), nivåsystem och grinding (B9, B11), spelets mål (B23), delar och designläge (B25), flygbanans framtid (B24).

Kvarstående:
1. **Delutbud i designläget** (hårtyper, överkroppar, kjol/byxor, färgpaletter, hur många att börja med).
2. **Fiendebudgetens skalning** (zon, avstånd på världskartan, antal fiender per rum).
3. **Hemliga områden:** hur ön ritas på världskartan och hur flygplatsens destination avgörs.
4. **Vapen och attacken:** hur ändras den magiska sfären av en stav? Finns närstridsvapen i top-down och plattformsscenerna, och hur ser de ut och träffar? Hur påverkar ringar och handskar attacken?
5. **Statuseffekter hos skor:** vilka (t.ex. fart, hopphöjd, skydd mot något)?
6. **Styrkekravet:** är "styrka" samma värde som `str` (som idag styr knuffen), eller en egen stat som växer med nivån och påverkas av vikt? Hur mycket får bärvikten sänka?
7. **Utrustningens utseende:** syns föremålen på karaktären i spelet, som lager ovanpå de visuella kläderna, eller bara i inventariemenyn?
8. **Fladdrarens delmodell:** vilken enstaka del är en fladdermus (huvud eller kropp)?
9. **Slutgiltigt antal platser** (en eller två ringplatser, handskar och ringar på samma plats eller inte) och prisnivåer och guldkällor.
10. **Flygbanans strid:** ska man någon gång kunna skjuta där, eller förblir den en undvikbana med bonusar?

## 10. Förslag på nästa steg

Börja Fas 2:
1. Bryt ut `GFX.person` till en delbaserad kroppsmall med ankarpunkter (kontrakt 11) utan att ändra utseendet, så att det befintliga spelet ser likadant ut.
2. Lägg `G.look` med delnamn och färger i `G`, och sparformat v7 med migrering som ger äldre sparfiler ett standardutseende.
3. Lägg till valen "Starta med designläge" och "Slump" vid nytt spel, och designläge i pausmenyn.
4. Ta fram frågorna 9.1 och 9.7 innan Fas 3, eftersom utrustningens synlighet påverkar hur kroppsmallen ritas.

## 11. Beslutslogg (kronologisk, ID:n återanvänds aldrig)

**Från v1:**
- **B1** Persistent värld per spelsession: världskartan och varje grotta, stad, ruinstad och flygbana sparas som layout och återställs exakt vid laddning, tillsammans med spelarens nivå och föremål. Hela världen genereras vid sessionsstart och är minnesbillig.
- **B2** Fiender och kistor återskapas vid varje besök (grinding). Av arenorna sparas bara spelarens uppnåenden som har bäring på spelets mål, vilket avgörs senare.
- **B3** Spelfigur vid nytt spel: "slump" eller "designläge" (hårtyp, överkropp, kjol/byxor, färger).
- **B4** Fiender ska ha varierade färger.
- **B5** Under utveckling finns en av varje scentyp (stad, grotta, ruinstad, flygplats). Antalet slumpas fram senare.

**Fas 0–1 (tidigare sessioner):**
- **B6** Spara från pausmenyn på världskartan plus autospar vid scenbyte. Spara/ladda som JSON-fil (export/import) och localStorage. Fas 0 byggs steg för steg.
- **B7** HP på fri skala där HUD visar hjärtan avrundat (inte fasta kvartshjärtan). HP-hjärtan ökar visuellt men proportionellt (t.ex. 600 HP → 1 hjärta = 100 HP).
- **B8** Teststrid: spelaren har en vit magisk sfär i åtta riktningar som stoppas av objekt, med autofire vid hållen knapp och uppehåll (enskilda sfärer). Attack på samma knapp som val (Space/Z). Ingen attack i städer. Fienden blinkar vid träff och lätta fiender knuffas bakåt medan tyngre inte gör det (lätta knuffas om spelarens styrka är minst lika stor). Besegrade fiender kan tappa ett hjärta.
- **B9** Nivåsystem: EXP från besegrade fiender, nivå 2 efter 10 EXP, nivå 3 efter ytterligare 15, med en gradvis ökande kurva som blir "helt lodrät" vid nivå 80 (max). Stridsformel i klassisk japansk RPG-anda: skada beror på kvoten Attack/Defense så att alla alltid gör minst 1 skada.
- **B10** Svårighet anpassas per grotta och ruin. Vid porten visas t.ex. "nivå 3, besök? j/n". Lågnivåspelare kan svimma av en träff där, men det ska vara spelbart för den envise (Princess Maker 2-känsla).
- **B11** EXP-principen: lätta fienders EXP skalas inte upp, så att grinding i grotta nivå 1 gör höga nivåer svåra att nå. Svårare zoner ger mer EXP i förhållande till styrkan som krävs, men inte extra bonus bara för att zonen är svår.
- **B12** Dialoger: öppnas vid beröring av bybo och stängs vid förflyttning bort från dem, och pil ned visar mer text. I handelsmenyn väljer man med upp/ned och lämnar genom att gå i sidled, via "Avsluta handel" eller högerklick.
- **B13** Mus: dialogrutor (t.ex. "Sparat") kan stängas med klick, och handelsdialogen kan öppnas med klick på valet "Handel".
- **B14** Ordning: balansverktyget tas efter att strid finns i ruinerna, så att allt kan testas samtidigt. *(Villkoret är uppfyllt, men balansverktyget flyttas till Fas 5 av B26.)*
- **B15** Fas 1b: testa en tung fiende (långsam, mycket HP, knuffas inte) och en skjutande fiende (brute och shooter är byggda).
- **B16** Flygbanan: tydligt uppstickande grå berg med mörkare skuggning på bara en sida (så att klipporna inte ser ut att sväva), grön mark med vågräta vattenband (flod, ibland bredare sjö/hav) som inte är hinder, stilla strandkanter, och träff = förlust av ett hjärta.
- **B17** Ruinstaden: originalfiender för just den banan, och samma magiattack som förut men med hopp och skott som i klassisk Mega Man (magi bara åt höger och vänster, på olika höjder vid hopp).

**Denna session (v2):**
- **B18** Flygbanans hinder: luftballonger (stilla men levande) och zeppelinare som glider in från sidan på två höjdnivåer, ungefär hälften som den ursprungliga nivån och hälften högre upp. Zeppelinarens varningsljud ska låta som en stor, välfylld, ekande ballong (dovt, låg ton). Flygbanan lämnas därefter tills vidare.
- **B19** Spelarplanet på flygbanan: mestadels triangelformat, med en rundad flygkropp i mitten (överdelen av ett rör, med 3D-skuggning) vars nos sticker ut en aning framför vingspetsen (halverad från första försöket), och en planformad skugga nere till höger med samma form och med avstånd, så att planet ser ut att flyga högt över marken.
- **B20** Föremålssystem efter förebild från Robin Hood: Prince of Thieves (NES): föremål bärs på kroppsdelar (huvud: mössa/hjälm, torso: väst/tröja/brynja, hand: magisk stav, närstridsvapen, ring eller handske, fötter: skor med statuseffekt). Vikten minskar styrkan lite, och tunga vapen och rustningar kräver viss styrka, annars är man blockerad. Det byggs som en mallfunktion först, och detaljerna för vikt m.m. tas senare.
- **B21** Föremål kommer från handel i staden, slumpade kistor i grottrum och ruinstad, och en kista som tappas av en besegrad boss.
- **B22** Fiender byggs efter flip-book-modellen (huvud/kropp/ben), men monster behöver inte ha alla delar: blob = bara kropp, kruka (eldspott) = bara huvud (aggressiv), fladdermus likaså. De nuvarande fienderna ersätts när den fasen kommer, och översättningen ska göras så att originalens karaktär bevaras.
- **B23** Spelets mål: besegra en mycket svår grotta som alltid är tillgänglig men kräver mycket EXP. Senare slumpas grottor och ruinstäder av olika svårighet ut på kartan. Ännu senare kan spelaren migrera sin högnivåkaraktär till en ny generering av världskartan med nytt innehåll och högre svårighet för en ny och svårare slutboss (t.ex. gömd i en grotta som bara finns efter att man flugit från en viss flygplats). Allt detta är senare arbete.
- **B24** Flygbanan får i ett senare läge bonusar att plocka upp under flygningen, t.ex. guld.
- **B25** Designläge: finns som val vid start och även i pausmenyn, så att man kan byta frisyr, överkropp och byxor eller kjol i olika utföranden när som helst. Kläder är bara visuella. Om man inte väljer att starta med designläge får man en slumpad klädsel och frisyr och startar direkt. Inventariemenyn återanvänder samma kroppsmall för att visa och placera föremål, och ryggsäcken har ingen gräns (en scrollande lista, "magisk väska").
- **B26** Arbetsplanen skrivs om helt (v2) och ersätter den gamla. Den förklarar vad som gjorts och ska räcka för senare sessioner en bit framåt. Föremål och kroppsmall/designläge prioriteras före balansverktyget.

## 12. Vad som har gjorts sedan v1 (kort)

- Hela Fas 0: genererad värld i `G.maps`, versionerat sparformat, spara/ladda, portal-ID, fri HP-skala.
- Hela Fas 1: magiattack, knuff, hjärtdropp, EXP, nivåer, stridsformel, zonnivåer, tre nya fiender, dialog- och handelsförbättringar och musstöd.
- Ruinstaden: sidscroll med hopp och egna fiender.
- Flygbanan: berg, vatten, ballonger, zeppelinare med ljud, kollision och ett nytt spelarplan med skugga.
- Inte påbörjat: kroppsmall, designläge, föremål, flip-book, balansverktyg, boss, hemliga zoner, slutmålet.
