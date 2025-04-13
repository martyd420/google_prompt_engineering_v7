Toto je překlad (1 prompt) pomocí Gemini 2.5 pro experimental.
Původní pdf zde: https://www.kaggle.com/whitepaper-prompt-engineering

# Prompt Engineering

**Autor: Lee Boonstra**

#### Poděkování

**Přispěvatelé obsahu**

Michael Sherma, Yuan Cao, Erick Armbrust, Anant Nawalgaria, Antonio Gulli, Simone Cammel

**Kurátoři a editoři**

Antonio Gulli, Anant Nawalgaria, Grace Mollison

**Technický redaktor**

Joey Haymaker

**Designér**

Michael Lanning

## Obsah

- Úvod
- Prompt engineering
- Konfigurace výstupu LLM
    - Délka výstupu
    - Řízení výběru (sampling controls)
        - Temperature
        - Top-K a top-P
        - Dejme to dohromady
- Techniky promptování
    - Obecné promptování / zero-shot
    - One-shot & few-shot
    - Systémové, kontextuální a rolí řízené promptování
        - Systémové promptování
        - Rolí řízené promptování
        - Kontextuální promptování
    - Promptování s krokem zpět (Step-back prompting)
    - Řetězec myšlenek (Chain of Thought - CoT)
    - Sebe-konzistence (Self-consistency)
    - Strom myšlenek (Tree of Thoughts - ToT)
    - ReAct (reason & act - uvažuj a jednej)
    - Automatický prompt engineering (Automatic Prompt Engineering)
    - Promptování pro kód (Code prompting)
        - Prompty pro psaní kódu
        - Prompty pro vysvětlení kódu
        - Prompty pro překlad kódu
        - Prompty pro ladění a revizi kódu
        - A co multimodální promptování?
- Osvědčené postupy (Best Practices)
    - Poskytněte příklady
    - Navrhujte s jednoduchostí
    - Buďte specifičtí ohledně výstupu
    - Používejte instrukce místo omezení
    - Kontrolujte maximální délku tokenů
    - Používejte proměnné v promptech
    - Experimentujte s formáty vstupu a styly psaní
    - Pro few-shot promptování u klasifikačních úloh, promíchejte třídy
    - Přizpůsobte se aktualizacím modelu
    - Experimentujte s formáty výstupu
    - Oprava JSON (JSON Repair)
    - Práce se schématy (Schemas)
    - Experimentujte společně s ostatními prompt inženýry
    - Osvědčené postupy pro CoT
    - Dokumentujte různé pokusy s prompty
- Shrnutí
- Poznámky na konci

## Úvod

Když přemýšlíme o vstupu a výstupu velkého jazykového modelu, textový prompt (někdy
doprovázený jinými modalitami, jako jsou obrázkové prompty) je vstup, který model používá
k predikci specifického výstupu. Nemusíte být datový vědec ani inženýr strojového učení
– prompt může napsat každý. Avšak vytvoření nejefektivnějšího promptu může být
složiité. Mnoho aspektů vašeho promptu ovlivňuje jeho účinnost: model, který používáte, trénovací
data modelu, konfigurace modelu, vaše volba slov, styl a tón, struktura a
kontext – to vše má význam. Proto je prompt engineering iterativní proces. Nevhodné prompty
mohou vést k nejednoznačným, nepřesným odpovědím a mohou bránit schopnosti modelu poskytovat
smysluplný výstup.

> Nemusíte být datový
> vědec ani inženýr strojového učení
> – prompt může napsat
> každý.

Když chatujete s chatbotem Gemini,^1 v podstatě píšete prompty, avšak tento
whitepaper se zaměřuje na psaní promptů pro model Gemini v rámci Vertex AI nebo pomocí
API, protože přímým promptováním modelu získáte přístup ke konfiguraci
jako je teplota atd.

Tento whitepaper podrobně pojednává o prompt engineeringu. Podíváme se na různé
techniky promptování, které vám pomohou začít, a sdílíme tipy a osvědčené postupy, abyste
se stali expertem na promptování. Probereme také některé výzvy, kterým můžete čelit
při tvorbě promptů.

## Prompt engineering

Pamatujte si, jak funguje LLM; je to predikční stroj. Model přijímá sekvenční text jako
vstup a poté předpovídá, jaký by měl být následující token, na základě dat, na kterých byl
trénován. LLM je operacionalizován tak, aby to dělal znovu a znovu, přidávaje dříve
predikovaný token na konec sekvenčního textu pro predikci následujícího tokenu. Predikce
dalšího tokenu je založena na vztahu mezi tím, co je v předchozích tokenech, a tím, co
LLM viděl během svého tréninku.

Když píšete prompt, snažíte se nastavit LLM tak, aby předpověděl správnou sekvenci
tokenů. Prompt engineering je proces navrhování vysoce kvalitních promptů, které vedou
LLM k produkci přesných výstupů. Tento proces zahrnuje ladění k nalezení nejlepšího promptu,
optimalizaci délky promptu a hodnocení stylu psaní a struktury promptu ve vztahu
k úkolu. V kontextu zpracování přirozeného jazyka a LLM je prompt vstup
poskytnutý modelu k vygenerování odpovědi nebo predikce.

Tyto prompty lze použít k dosažení různých druhů úkolů porozumění a generování,
jako je sumarizace textu, extrakce informací, odpovídání na otázky, klasifikace
textu, překlad jazyka nebo kódu, generování kódu a dokumentace kódu
nebo zdůvodňování.

Neváhejte se podívat na průvodce promptováním od Googlu^2,3 s jednoduchými a efektivními
příklady promptování.

Při prompt engineeringu začnete výběrem modelu. Prompty možná bude třeba
optimalizovat pro váš specifický model, bez ohledu na to, zda používáte jazykové modely Gemini
ve Vertex AI, GPT, Claude nebo open source model jako Gemma nebo LLaMA.

Kromě promptu budete také muset ladit různé konfigurace LLM.

## Konfigurace výstupu LLM

Jakmile si vyberete model, budete muset zjistit konfiguraci modelu. Většina LLM
přichází s různými možnostmi konfigurace, které řídí výstup LLM. Efektivní prompt
engineering vyžaduje optimální nastavení těchto konfigurací pro váš úkol.

### Délka výstupu

Důležitým nastavením konfigurace je počet tokenů, které se mají vygenerovat v odpovědi.
Generování více tokenů vyžaduje více výpočtů od LLM, což vede k vyšší spotřebě energie,
potenciálně pomalejším dobám odezvy a vyšším nákladům.

Snížení délky výstupu LLM nezpůsobí, že LLM bude stylisticky
nebo textově stručnější ve výstupu, který vytváří, pouze způsobí, že LLM přestane předpovídat další
tokeny, jakmile je dosaženo limitu. Pokud vaše potřeby vyžadují krátkou délku výstupu, budete
možná také muset upravit svůj prompt, aby tomu vyhovoval.

Omezení délky výstupu je zvláště důležité pro některé techniky promptování LLM, jako je
ReAct, kde LLM bude pokračovat ve vydávání neužitečných tokenů i po odpovědi, kterou chcete.

> Buďte si vědomi, že generování více tokenů vyžaduje více výpočtů od LLM, což vede
> k vyšší spotřebě energie a potenciálně pomalejším dobám odezvy, což vede k
> vyšším nákladům.

### Řízení výběru (sampling controls)

LLM formálně nepředpovídají jediný token. Spíše LLM předpovídají pravděpodobnosti pro to, jaký
by mohl být další token, přičemž každý token ve slovníku LLM získává pravděpodobnost. Tyto
pravděpodobnosti tokenů jsou poté vzorkovány (sampled) k určení, jaký bude další vyprodukovaný token.
Teplota (Temperature), top-K a top-P jsou nejběžnější nastavení konfigurace, která určují,
jak jsou zpracovány predikované pravděpodobnosti tokenů pro výběr jediného výstupního tokenu.

#### Teplota (Temperature)

Teplota řídí míru náhodnosti při výběru tokenů. Nižší teploty
jsou dobré pro prompty, které očekávají determinističtější odpověď, zatímco vyšší teploty
mohou vést k rozmanitějším nebo neočekávaným výsledkům. Teplota 0 (greedy decoding) je
deterministická: vždy je vybrán token s nejvyšší pravděpodobností (i když si všimněte, že pokud dva tokeny
mají stejnou nejvyšší predikovanou pravděpodobnost, v závislosti na implementaci řešení remíz
nemusíte vždy získat stejný výstup s teplotou 0).

Teploty blízké maximu mají tendenci vytvářet náhodnější výstup. A jak se teplota
stále zvyšuje, všechny tokeny se stávají stejně pravděpodobnými jako další predikovaný token.

Řízení teploty Gemini lze chápat podobně jako funkci softmax
používanou ve strojovém učení. Nízké nastavení teploty odráží nízkou teplotu softmax (T),
zdůrazňující jedinou preferovanou teplotu s vysokou jistotou. Vyšší nastavení teploty Gemini
je jako vysoká teplota softmax, což činí širší rozsah teplot kolem
vybraného nastavení přijatelnějším. Tato zvýšená nejistota vyhovuje scénářům,
kde rigidní, přesná teplota nemusí být nezbytná, například při experimentování
s kreativními výstupy.

#### Top-K a top-P

Top-K a top-P (také známé jako vzorkování jádra - nucleus sampling)^4 jsou dvě nastavení vzorkování používaná v LLM
k omezení predikovaného dalšího tokenu tak, aby pocházel z tokenů s nejvyššími predikovanými pravděpodobnostmi.
Stejně jako teplota, tato nastavení vzorkování řídí náhodnost a rozmanitost
generovaného textu.

-   Vzorkování **Top-K** vybírá K nejpravděpodobnějších tokenů z predikované
    distribuce modelu. Čím vyšší top-K, tím kreativnější a rozmanitější je výstup modelu; čím
    nižší top-K, tím zdrženlivější a faktografičtější je výstup modelu. Top-K rovné 1 je ekvivalentní
    greedy decoding.

-   Vzorkování **Top-P** vybírá nejlepší tokeny, jejichž kumulativní pravděpodobnost nepřesahuje
    určitou hodnotu (P). Hodnoty pro P se pohybují od 0 (greedy decoding) do 1 (všechny tokeny ve
    slovníku LLM).

Nejlepší způsob, jak si vybrat mezi top-K a top-P, je experimentovat s oběma metodami (nebo
oběma dohromady) a zjistit, která z nich přináší výsledky, které hledáte.

#### Dejme to dohromady

Volba mezi top-K, top-P, teplotou a počtem tokenů k vygenerování
závisí na konkrétní aplikaci a požadovaném výsledku a všechna nastavení se vzájemně ovlivňují.
Je také důležité se ujistit, že rozumíte tomu, jak váš vybraný model kombinuje
různá nastavení vzorkování dohromady.

Pokud jsou k dispozici teplota, top-K a top-P (jako ve Vertex Studiu), tokeny, které splňují
kritéria top-K i top-P, jsou kandidáty na další predikovaný token a poté
se aplikuje teplota pro vzorkování z tokenů, které prošly kritérii top-K a top-P. Pokud
je k dispozici pouze top-K nebo top-P, chování je stejné, ale použije se pouze jedno nastavení top-K nebo P.

Pokud teplota není k dispozici, jakékoli tokeny, které splňují kritéria top-K a/nebo top-P, jsou poté
náhodně vybrány k produkci jediného dalšího predikovaného tokenu.

Při extrémních nastaveních jedné hodnoty konfigurace vzorkování toto jedno nastavení vzorkování buď
ruší ostatní nastavení konfigurace, nebo se stává irelevantním.

-   Pokud nastavíte teplotu na 0, top-K a top-P se stanou irelevantními – nejpravděpodobnější
    token se stane dalším predikovaným tokenem. Pokud nastavíte teplotu extrémně vysoko (nad
    1 – obecně do desítek), teplota se stane irelevantní a jakékoli tokeny, které projdou
    kritérii top-K a/nebo top-P, jsou poté náhodně vzorkovány k výběru dalšího
    predikovaného tokenu.
-   Pokud nastavíte top-K na 1, teplota a top-P se stanou irelevantními. Pouze jeden token projde
    kritérii top-K a tento token je dalším predikovaným tokenem. Pokud nastavíte top-K extrémně vysoko,
    například na velikost slovníku LLM, jakýkoli token s nenulovou pravděpodobností, že bude
    dalším tokenem, splní kritéria top-K a žádný není vyřazen.
-   Pokud nastavíte top-P na 0 (nebo velmi malou hodnotu), většina implementací vzorkování LLM pak
    bude považovat pouze nejpravděpodobnější token za splňující kritéria top-P, čímž se teplota a
    top-K stanou irelevantními. Pokud nastavíte top-P na 1, jakýkoli token s nenulovou pravděpodobností, že bude
    dalším tokenem, splní kritéria top-P a žádný není vyřazen.

Jako obecný výchozí bod vám teplota 0.2, top-P 0.95 a top-K 30 poskytne
relativně koherentní výsledky, které mohou být kreativní, ale ne přehnaně. Pokud chcete obzvláště
kreativní výsledky, zkuste začít s teplotou 0.9, top-P 0.99 a top-K 40. A pokud chcete
méně kreativní výsledky, zkuste začít s teplotou 0.1, top-P 0.9 a top-K 20.
Nakonec, pokud má váš úkol vždy jednu správnou odpověď (např. odpověď na matematický problém), začněte
s teplotou 0.

**POZNÁMKA:** S větší volností (vyšší teplota, top-K, top-P a výstupní tokeny) může LLM
generovat text, který je méně relevantní.

**VAROVÁNÍ:** Viděli jste někdy odpověď končící velkým množstvím výplňkových slov? Toto
je také známé jako "chyba opakovací smyčky" (repetition loop bug), což je běžný problém u velkých jazykových
modelů, kdy se model zasekne v cyklu a opakovaně generuje stejné (výplňkové) slovo,
frázi nebo větnou strukturu, což je často zhoršeno nevhodným nastavením teploty a top-k/top-p.
K tomu může dojít jak při nízkém, tak při vysokém nastavení teploty, i když z různých
důvodů. Při nízkých teplotách se model stává příliš deterministickým, rigidně se drží
cesty s nejvyšší pravděpodobností, což může vést ke smyčce, pokud tato cesta znovu navštíví dříve generovaný
text. Naopak při vysokých teplotách se výstup modelu stává nadměrně náhodným,
což zvyšuje pravděpodobnost, že náhodně zvolené slovo nebo fráze náhodou povede zpět
k předchozímu stavu, čímž vznikne smyčka kvůli obrovskému počtu dostupných možností. V obou případech
se proces vzorkování modelu "zasekne", což vede k monotónnímu a neužitečnému výstupu,
dokud není výstupní okno zaplněno. Řešení tohoto problému často vyžaduje pečlivé ladění hodnot teploty
a top-k/top-p k nalezení optimální rovnováhy mezi determinismem a náhodností.

## Techniky promptování

LLM jsou vyladěny tak, aby se řídily instrukcemi, a jsou trénovány na velkém množství dat, aby
rozuměly promptu a generovaly odpověď. Ale LLM nejsou dokonalé; čím jasnější je
váš text promptu, tím lépe může LLM předpovědět další pravděpodobný text. Navíc specifické
techniky, které využívají toho, jak jsou LLM trénovány a jak fungují, vám pomohou získat
relevantní výsledky od LLM.

Nyní, když rozumíme tomu, co je prompt engineering a co obnáší, pojďme se ponořit do několika
příkladů nejdůležitějších technik promptování.

### Obecné promptování / zero-shot

_Zero-shot_^5 prompt je nejjednodušší typ promptu. Poskytuje pouze popis úkolu
a nějaký text, se kterým může LLM začít. Tento vstup může být cokoli: otázka,
začátek příběhu nebo instrukce. Název zero-shot znamená „žádné příklady“.

Použijme Vertex AI Studio (pro jazyk) ve Vertex AI,^6 které poskytuje hřiště (playground) pro testování
promptů. V tabulce 1 uvidíte příklad zero-shot promptu pro klasifikaci filmových recenzí.

Formát tabulky, jak je použit níže, je skvělý způsob dokumentování promptů. Vaše prompty
pravděpodobně projdou mnoha iteracemi, než skončí v kódové základně, takže je důležité sledovat
vaši práci na prompt engineeringu disciplinovaným, strukturovaným způsobem. Více o tomto formátu tabulky,
důležitosti sledování práce na prompt engineeringu a procesu vývoje promptu je v sekci
Osvědčené postupy dále v této kapitole („Dokumentujte různé pokusy s prompty“).

Teplota modelu by měla být nastavena na nízké číslo, protože není potřeba žádná kreativita, a
používáme výchozí hodnoty top-K a top-P modelu gemini-pro, které efektivně deaktivují obě nastavení
(viz „Konfigurace výstupu LLM“ výše). Věnujte pozornost generovanému výstupu. Slova
_disturbing_ (znepokojující) a _masterpiece_ (mistrovské dílo) by měla predikci trochu zkomplikovat, protože obě
slova jsou použita ve stejné větě.

```
Název               1_1_movie_classification
Cíl                 Klasifikovat filmové recenze jako pozitivní, neutrální nebo negativní.
Model               gemini-pro
Teplota             0.1       Limit tokenů      5
Top - K             N/A       Top - P           1
Prompt              Klasifikujte filmové recenze jako POZITIVNÍ, NEUTRÁLNÍ nebo NEGATIVNÍ.
                    Recenze: "Ona" je znepokojující studie odhalující směr,
                    kterým se lidstvo ubírá, pokud bude AI dovoleno se dále vyvíjet,
                    nekontrolovaně. Kéž by bylo více filmů jako toto mistrovské dílo.
                    Sentiment:
Výstup              POZITIVNÍ
```
Tabulka 1. Příklad zero-shot promptování

Když zero-shot nefunguje, můžete v promptu poskytnout ukázky nebo příklady,
což vede k „one-shot“ a „few-shot“ promptování.

### One-shot & few-shot

Při vytváření promptů pro AI modely je užitečné poskytnout příklady. Tyto příklady mohou
modelu pomoci pochopit, co požadujete. Příklady jsou zvláště užitečné, když
chcete model nasměrovat k určité výstupní struktuře nebo vzoru.

**One-shot prompt** poskytuje _jediný_ příklad, odtud název one-shot. Myšlenka je, že
model má příklad, který může napodobit, aby co nejlépe dokončil úkol.

**Few-shot prompt**^7 poskytuje modelu _více_ příkladů. Tento přístup ukazuje
modelu vzor, který má následovat. Myšlenka je podobná one-shot, ale více příkladů
požadovaného vzoru zvyšuje šanci, že model vzor dodrží.

Počet příkladů, které potřebujete pro few-shot promptování, závisí na několika faktorech,
včetně složitosti úkolu, kvality příkladů a schopností
generativního AI (gen AI) modelu, který používáte. Jako obecné pravidlo byste měli použít
alespoň tři až pět příkladů pro few-shot promptování. Nicméně, možná budete muset použít více
příkladů pro složitější úkoly, nebo možná budete muset použít méně kvůli omezení
délky vstupu vašeho modelu.

Tabulka 2 ukazuje příklad few-shot promptu, použijme stejná nastavení konfigurace modelu
gemini-pro jako předtím, kromě zvýšení limitu tokenů, abychom vyhověli
potřebě delší odpovědi.

```
Cíl                 Parsovat objednávky pizzy do JSONu
Model               gemini-pro
Teplota             0.1       Limit tokenů      250
Top - K             N/A       Top - P           1
Prompt              Naparsuj zákazníkovu objednávku pizzy do platného JSONu:
                    PŘÍKLAD:
                    Chci malou pizzu se sýrem, rajčatovou omáčkou a feferonkami.
                    Odpověď JSON:
                    ```json
                    {
                      "velikost": "malá",
                      "typ": "normální",
                      "ingredience": [["sýr", "rajčatová omáčka", "feferonky"]]
                    }
                    ```
                    Pokračuje na další straně...
```

```
Prompt              PŘÍKLAD:
                    Mohl bych dostat velkou pizzu s rajčatovou omáčkou, bazalkou a mozzarellou?
                    ```json
                    {
                      "velikost": "velká",
                      "typ": "normální",
                      "ingredience": [["rajčatová omáčka", "bazalka", "mozzarella"]]
                    }
                    ```
                    Teď bych chtěl velkou pizzu, první polovina sýr a
                    mozzarella. A druhá rajčatová omáčka, šunka a ananas.
                    Odpověď JSON:
Výstup              ```json
                    {
                      "velikost": "velká",
                      "typ": "půl-půl",
                      "ingredience": [["sýr", "mozzarella"], ["rajčatová omáčka", "šunka", "ananas"]]
                    }
                    ```
```
Tabulka 2. Příklad few-shot promptování

Když vybíráte příklady pro svůj prompt, použijte příklady, které jsou relevantní pro úkol, který
chcete provést. Příklady by měly být rozmanité, vysoce kvalitní a dobře napsané. Jedna malá
chyba může model zmást a bude mít za následek nežádoucí výstup.

Pokud se snažíte generovat výstup, který je robustní vůči různým vstupům, pak je důležité
zahrnout do vašich příkladů okrajové případy (edge cases). Okrajové případy jsou vstupy, které jsou neobvyklé nebo neočekávané,
ale které by model měl stále umět zpracovat.

### Systémové, kontextuální a rolí řízené promptování

Systémové, kontextuální a rolí řízené promptování jsou všechny techniky používané k řízení toho, jak LLM generují
text, ale zaměřují se na různé aspekty:

-   **Systémové promptování** nastavuje celkový kontext a účel pro jazykový model. Definuje
    „velký obrázek“ toho, co by model měl dělat, jako je překlad jazyka,
    klasifikace recenze atd.
-   **Kontextuální promptování** poskytuje specifické detaily nebo podkladové informace relevantní pro
    aktuální konverzaci nebo úkol. Pomáhá modelu porozumět nuancím toho, co
    je požadováno, a přizpůsobit odpověď tomu.
-   **Rolí řízené promptování** přiřazuje specifickou postavu nebo identitu, kterou má jazykový model přijmout.
    To pomáhá modelu generovat odpovědi, které jsou konzistentní s přiřazenou rolí a jejími
    přidruženými znalostmi a chováním.

Mezi systémovým, kontextuálním a rolí řízeným promptováním může existovat značný překryv. Např.
prompt, který systému přiřazuje roli, může mít také kontext.

Avšak každý typ promptu slouží mírně odlišnému primárnímu účelu:

-   Systémový prompt: Definuje základní schopnosti modelu a jeho zastřešující účel.
-   Kontextuální prompt: Poskytuje okamžité, úkolově specifické informace k řízení odpovědi.
    Je vysoce specifický pro aktuální úkol nebo vstup, což je dynamické.
-   Rolí řízený prompt: Rámuje styl a hlas výstupu modelu. Přidává vrstvu specifičnosti
    a osobnosti.

Rozlišování mezi systémovými, kontextuálními a rolí řízenými prompty poskytuje rámec pro
navrhování promptů s jasným záměrem, což umožňuje flexibilní kombinace a usnadňuje
analýzu toho, jak každý typ promptu ovlivňuje výstup jazykového modelu.

Pojďme se ponořit do těchto tří různých druhů promptů.

#### Systémové promptování

Tabulka 3 obsahuje systémový prompt, kde specifikuji dodatečné informace o tom, jak vrátit
výstup. Zvýšil jsem teplotu, abych získal vyšší úroveň kreativity, a specifikoval jsem vyšší
limit tokenů. Avšak kvůli mé jasné instrukci, jak vrátit výstup, model
nevrátil žádný extra text.

```
Cíl                 Klasifikovat filmové recenze jako pozitivní, neutrální nebo negativní.
Model               gemini-pro
Teplota             1         Limit tokenů      5
Top - K             40        Top - P           0.
Prompt              Klasifikujte filmové recenze jako pozitivní, neutrální nebo negativní. Vraťte
                    pouze štítek velkými písmeny.
                    Recenze: "Ona" je znepokojující studie odhalující směr,
                    kterým se lidstvo ubírá, pokud bude AI dovoleno se dále vyvíjet,
                    nekontrolovaně. Je to tak znepokojující, že jsem to nemohl dokoukat.
                    Sentiment:
Výstup              NEGATIVNÍ
```
Tabulka 3. Příklad systémového promptování

Systémové prompty mohou být užitečné pro generování výstupu, který splňuje specifické požadavky.
Název „systémový prompt“ ve skutečnosti znamená „poskytnutí dodatečného úkolu systému“. Například
byste mohli použít systémový prompt k vygenerování úryvku kódu, který je kompatibilní se
specifickým programovacím jazykem, nebo byste mohli použít systémový prompt k vrácení určité
struktury. Podívejte se do tabulky 4, kde vracím výstup ve formátu JSON.

```
Cíl                 Klasifikovat filmové recenze jako pozitivní, neutrální nebo negativní, vrátit JSON.
Model               gemini-pro
Teplota             1         Limit tokenů      1024
Top - K             40        Top - P           0.
Prompt              Klasifikujte filmové recenze jako pozitivní, neutrální nebo negativní. Vraťte
                    platný JSON:
                    Recenze: "Ona" je znepokojující studie odhalující směr,
                    kterým se lidstvo ubírá, pokud bude AI dovoleno se dále vyvíjet,
                    nekontrolovaně. Je to tak znepokojující, že jsem to nemohl dokoukat.
                    Schéma:
                    ```
                    FILM:
                    {
                      "sentiment": String "POZITIVNÍ" | "NEGATIVNÍ" | "NEUTRÁLNÍ",
                      "nazev": String
                    }
                    FILMOVÉ RECENZE:
                    {
                      "filmove_recenze": [FILM]
                    }
                    ```
                    Odpověď JSON:
Výstup              ```json
                    {
                      "filmove_recenze": [
                        {
                          "sentiment": "NEGATIVNÍ",
                          "nazev": "Ona"
                        }
                      ]
                    }
                    ```
```
Tabulka 4. Příklad systémového promptování s formátem JSON

Existují určité výhody vracení objektů JSON z promptu, který extrahuje data. V
reálné aplikaci nemusím tento formát JSON vytvářet ručně, mohu již
vrátit data v seřazeném pořadí (velmi užitečné při práci s objekty datetime), ale co je nejdůležitější,
promptováním pro formát JSON nutí model vytvořit strukturu a
omezit halucinace.

Systémové prompty mohou být také velmi užitečné pro bezpečnost a toxicitu. Pro kontrolu výstupu jednoduše
přidejte do svého promptu další řádek jako: „Ve své odpovědi byste měl být uctivý.“.

#### Rolí řízené promptování

Rolí řízené promptování je technika v prompt engineeringu, která zahrnuje přiřazení specifické role
modelu gen AI. To může modelu pomoci generovat relevantnější a informativnější výstup,
protože model může své odpovědi přizpůsobit specifické roli, která mu byla přiřazena.

Například byste mohli rolí řízeným promptem přimět gen AI model, aby byl knižním editorem, učitelem v mateřské škole
nebo motivačním řečníkem. Jakmile byla modelu přiřazena role, můžete mu poté
dávat prompty, které jsou specifické pro tuto roli. Například byste mohli vyzvat učitele,
aby vytvořil plán lekce, který pak může zkontrolovat. Podívejte se do tabulky 5, kde model
přebírá roli cestovního průvodce.

```
Cíl                 Chovej se jako cestovní průvodce a poskytni 3 návrhy na cesty
Model               gemini-pro
Teplota             1         Limit tokenů      1024
Top - K             40        Top - P           0.8
Prompt              Chci, abys vystupoval jako cestovní průvodce. Napíšu ti
                    o své poloze a ty mi navrhneš 3 místa k návštěvě poblíž
                    mě. V některých případech ti také dám typ míst, která
                    chci navštívit.
                    Můj návrh: "Jsem v Amsterdamu a chci navštívit
                    pouze muzea."
                    Návrhy na cestování:
Výstup              1. Rijksmuseum: Světoznámé muzeum, známé holandskými mistrovskými díly
                    jako Rembrandtova „Noční hlídka“. Nenechte si ujít neuvěřitelnou sbírku
                    obrazů a artefaktů představujících holandské umění a historii.
                    2. Van Goghovo muzeum: Ponořte se do podmanivých děl Vincenta
                       van Gogha. Muzeum uchovává největší sbírku jeho obrazů a
                       kresby, včetně „Hvězdné noci“ a „Slunečnic“.
                    3. Stedelijk Museum Amsterdam: Objevte moderní a současné umění
                       z celého světa. Sbírka muzea zahrnuje díla Picassa,
                       Kandinského a Rietvelda, umístěná v pozoruhodné moderní budově.
```
Tabulka 5. Příklad rolí řízeného promptování

Výše uvedený příklad ukazuje příklad převzetí role cestovního agenta. Když byste
změnili roli na učitele zeměpisu, všimli byste si, že obdržíte
jinou odpověď.

Definování perspektivy role pro AI model mu dává plán tónu, stylu a zaměřené
odbornosti, které hledáte, abyste zlepšili kvalitu, relevanci a efektivitu vašeho
výstupu.

Zde jsou některé styly, ze kterých si můžete vybrat a které považuji za efektivní:

Konfrontační, Popisný, Přímý, Formální, Humorný, Vlivný, Neformální,
Inspirativní, Přesvědčivý

Změňme náš prompt v tabulce 6 tak, aby zahrnoval humorný a inspirativní styl.

```
Prompt              Chci, abys vystupoval jako cestovní průvodce. Napíšu ti o
                    mé poloze a ty mi navrhneš 3 místa k návštěvě poblíž mě v
                    humorném stylu.
                    Můj návrh: "Jsem na Manhattanu."
                    Návrhy na cestování:
Výstup              1. Pohleďte na Empire State of Mind: Vystoupejte do závratných výšin Empire
                       State Building a vyhřívejte se ve slávě manhattanského panoramatu. Připravte se cítit se jako
                       King Kong na vrcholu Velkého jablka, mínus banán velikosti obří opice.
                    2. Buďte umělecko-fajnoví v MoMA: Uvolněte svého vnitřního uměleckého nadšence v Muzeu
                       moderního umění (MoMA). Hleďte na mistrovská díla, která vám zamotají
                       hlavu a přimějí vás zpochybnit, zda vaše kresby panáčků mají nějakou
                       uměleckou hodnotu.
                    3. Nakupujte, dokud nepadnete na Páté Avenue: Dopřejte si maloobchodní terapii na
                       ikonické Páté Avenue. Připravte se na cenový šok, když budete okukovat výlohy (nebo
                       skutečně nakupovat) v designérských buticích, které rozpláčou vaši peněženku. Ale hej,
                       jste na Manhattanu, tak proč si neužít tu báječnost!
```
Tabulka 6. Příklad rolí řízeného promptování s humorným tónem a stylem

#### Kontextuální promptování

Poskytnutím kontextuálních promptů můžete pomoci zajistit, že vaše interakce s AI budou co
nejhladší a nejefektivnější. Model bude schopen rychleji porozumět vašemu
požadavku a bude schopen generovat přesnější a relevantnější odpovědi, jak můžete vidět v
příkladu v tabulce 7.

```
Cíl                 Navrhnout články pro blog o retro hrách
Model               gemini-pro
Teplota             1         Limit tokenů      1024
Top - K             40        Top - P           0.8
Prompt              Kontext: Píšete pro blog o retro arkádových
                    video hrách z 80. let.
                    Navrhněte 3 témata pro článek s několika řádky
                    popisu toho, co by tento článek měl obsahovat.
Výstup              1. **Vývoj designu arkádových kabinetů:**
                       Tento článek by prozkoumal vývoj designu arkádových kabinetů, od
                       raných dřevěných a kovových kabinetů 70. let po elegantní, neonově osvětlené designy
                       80. let a dále.
                    2. **Výbuch z minulosti: Ikonické arkádové hry 80. let:**
                       - Tento článek by představil pohled do historie, dopadu a odkazu
                         některých nejikoničtějších a nejvlivnějších arkádových her 80. let,
                         zdůrazňující jejich inovace, populární mechaniky a trvalé kouzlo.
                    3. **Vzestup a retro oživení pixel artu:**
                       - Tento článek by se ponořil do vývoje pixel artu jako definujícího vizuálního
                         stylu raných arkádových her, sledoval by jeho kořeny, prozkoumával jeho techniky a
                         diskutoval o oživení pixel artu v moderních hrách a digitálním umění.
```
Tabulka 7. Příklad kontextuálního promptování

### Promptování s krokem zpět (Step-back prompting)

Promptování s krokem zpět^8 (Step-back prompting) je technika pro zlepšení výkonu tím, že se LLM nejprve vyzve,
aby zvážil obecnou otázku související s konkrétním úkolem, a poté se odpověď
na tuto obecnou otázku vloží do následného promptu pro konkrétní úkol. Tento „krok
zpět“ umožňuje LLM aktivovat relevantní základní znalosti a procesy uvažování
před pokusem o vyřešení konkrétního problému.

Zvážením širších a základních principů mohou LLM generovat přesnější a
pronikavější odpovědi. Promptování s krokem zpět povzbuzuje LLM k kritickému myšlení a aplikaci
svých znalostí novými a kreativními způsoby. Mění konečný prompt provádějící úkol tím, že využívá
více znalostí v parametrech LLM, než by jinak přišlo do hry, když je LLM
promptován přímo.

Může pomoci zmírnit zkreslení (biases) v odpovědích LLM tím, že se zaměří na obecné principy místo
specifických detailů.

Podívejme se na tyto příklady, abychom pochopili, jak může promptování s krokem zpět zlepšit
výsledky. Nejprve si prohlédněme tradiční prompt (Tabulka 8), než jej porovnáme s promptem
s krokem zpět (Tabulka 9):

```
Cíl                 Napsat dějovou linii pro úroveň střílečky z pohledu první osoby.
Model               gemini-pro
Teplota             1         Limit tokenů      1024
Top - K             40        Top - P           0.8
Prompt              Napište jedno-odstavcovou dějovou linii pro novou úroveň střílečky
                    z pohledu první osoby, která je náročná a poutavá.
Výstup              Úroveň začíná přepadením hráčovy jednotky v husté městské oblasti
                    nemilosrdnou nepřátelskou frakcí. Hráči se musí probojovat bludištěm
                    zadních uliček a zchátralých budov, využívajíc plížení a přesnost k eliminaci
                    nepřátelských sil a vyhnutí se detekci. Hráč musí využít své prostředí ke svému
                    prospěchu, nastražovat pasti, využívat krytí a dělat promyšlené kroky, aby přežil.
                    Po cestě odhalí informace naznačující, že nepřítel plánuje zničující
                    útok, a hráč musí závodit s časem, aby shromáždil důkazy a předal je zpět
                    do svého velitelství, než nepřítel stihne své plány uskutečnit.
```
Tabulka 8. Tradiční prompt před porovnáním s promptem s krokem zpět

Když nastavíte teplotu na 1, můžete skončit s nejrůznějšími kreativními nápady na
dějovou linii, ale je to také docela náhodné a obecné. Takže udělejme krok zpět:

```
Prompt              Na základě populárních akčních stříleček z pohledu první osoby, jakých je
                    5 fiktivních klíčových prostředí, která přispívají k náročné a
                    poutavé dějové linii úrovně ve střílečce z pohledu první osoby?
Výstup              1. **Opuštěná vojenská základna**: Rozlehlý, postapokalyptický vojenský komplex
                       hemžící se zmutovanými vojáky a zběhlými roboty, ideální pro náročný
                       boj se střelnými zbraněmi.
                    2. **Kyberpunkové město**: Neonem osvětlené, futuristické městské prostředí s tyčícími se
                       mrakodrapy a hustými uličkami, představující kyberneticky vylepšené nepřátele
                       a mechaniky hackování.
                    3. **Mimozemská vesmírná loď**: Obrovské mimozemské plavidlo uvízlé na Zemi, s děsivými
                       chodbami, sekcemi s nulovou gravitací a mimozemskými tvory k setkání.
                    4. **Město zamořené zombie**: Opuštěné město zaplavené hordami agresivních
                       zombie, představující intenzivní boj zblízka a řešení hádanek k nalezení
                       bezpečné cesty.
                    5. **Podvodní výzkumné zařízení**: Hlubokomořská laboratoř zaplavená vodou,
                       plná zmutovaných vodních tvorů a vyžadující plížení a podvodní
                       průzkumné dovednosti k přežití.
```
Tabulka 9. Příklad promptování pro sebe-konzistenci (Pozn. překl.: Zde se zdá být chyba v originálním popisku tabulky, mělo by jít o krok zpět, nikoli sebe-konzistenci)

Ano, tato témata se zdají být vhodná pro střílečku z pohledu první osoby. Vraťme se k
původnímu promptu, ale tentokrát zahrneme odpověď z kroku zpět jako kontext a uvidíme, co
vrátí.

```
Prompt              Kontext: 5 poutavých témat pro střílečku z pohledu první osoby:
                    1. **Opuštěná vojenská základna**: Rozlehlý, postapokalyptický
                       vojenský komplex hemžící se zmutovanými vojáky a zběhlými
                       roboty, ideální pro náročný boj se střelnými zbraněmi.
                    2. **Kyberpunkové město**: Neonem osvětlené, futuristické městské prostředí
                       s tyčícími se mrakodrapy a hustými uličkami, představující
                       kyberneticky vylepšené nepřátele a mechaniky hackování.
                    3. **Mimozemská vesmírná loď**: Obrovské mimozemské plavidlo uvízlé na
                       Zemi, s děsivými chodbami, sekcemi s nulovou gravitací a
                       mimozemskými tvory k setkání.
                    4. **Město zamořené zombie**: Opuštěné město zaplavené hordami
                       agresivních zombie, představující intenzivní boj zblízka a
                       řešení hádanek k nalezení bezpečné cesty.
                    5. **Podvodní výzkumné zařízení**: Hlubokomořská laboratoř zaplavená
                       vodou, plná zmutovaných vodních tvorů a vyžadující
                       plížení a podvodní průzkumné dovednosti k přežití.
                    Vyberte jedno z témat a napište jedno-odstavcovou dějovou linii
                    pro novou úroveň střílečky z pohledu první osoby, která je
                    náročná a poutavá.
Výstup              V srdci kalné propasti leží zchátralé podvodní výzkumné zařízení,
                    stojící jako svědectví lidské ambice a jejích katastrofálních následků.
                    Zahalené tmou, pulzující hučením nefunkčních strojů a
                    pronásledované ozvěnami utopených duší, je zařízení nyní líhní pro
                    noční můru zrozenou z hlubokého moře. Hráč, elitní mariňák vybavený
                    pokročilým potápěčským vybavením a experimentálními zbraněmi, má za úkol vydat se do
                    této vodní říše teroru. Vyzbrojen odolností a vynalézavostí, musí
                    proplouvat zrádnými chodbami, řešit kryptické hádanky a čelit hrůzostrašným mořským
                    obludám, které číhají v hlubinách. Od zápasu s drtivým tlakem
                    hlubokého moře po přechytračení lstivých vodních predátorů, každý okamžik v tomto
                    nezmapovaném podsvětí testuje hranice lidské vytrvalosti a odvahy.
```
Tabulka 10. Příklad promptování s krokem zpět (Pozn. překl.: Zde se zdá být chyba v originálním popisku tabulky, mělo by jít o prompt s krokem zpět, nikoli sebe-konzistenci)

To vypadá jako zajímavá videohra! Použitím technik promptování s krokem zpět můžete
zvýšit přesnost vašich promptů.

### Řetězec myšlenek (Chain of Thought - CoT)

Řetězec myšlenek (CoT)^9 promptování je technika pro zlepšení schopností uvažování
LLM generováním **mezikroků uvažování**. To pomáhá LLM generovat přesnější
odpovědi. Můžete jej kombinovat s few-shot promptováním, abyste získali lepší výsledky u složitějších
úkolů, které vyžadují uvažování před odpovědí, protože je to výzva s zero-shot
řetězcem myšlenek.

CoT má mnoho výhod. Především je nenáročný na úsilí, přičemž je velmi efektivní a funguje
dobře s běžně dostupnými LLM (takže není třeba jemné ladění - finetuning). S CoT
promptováním také získáte interpretovatelnost, protože se můžete učit z odpovědí LLM a vidět
kroky uvažování, které byly následovány. Pokud dojde k chybě, budete ji schopni identifikovat. Zdá se,
že řetězec myšlenek zlepšuje robustnost při přechodu mezi různými verzemi LLM. Což znamená, že
výkon vašeho promptu by se měl méně lišit mezi různými LLM, než kdyby váš prompt
nepoužíval řetězce uvažování. Samozřejmě existují i nevýhody, ale jsou
poněkud intuitivní.

Odpověď LLM zahrnuje uvažování řetězce myšlenek, což znamená více výstupních
tokenů, což znamená, že predikce stojí více peněz a trvají déle.

Pro vysvětlení následujícího příkladu v tabulce 11 se nejprve pokusme vytvořit prompt, který nepoužívá
CoT promptování, abychom ukázali nedostatky velkého jazykového modelu.

```
Prompt              Když mi byly 3 roky, můj partner byl 3krát starší než já. Nyní je mi
                    20 let. Kolik let je mému partnerovi?
Výstup              63 let
```
Tabulka 11. Příklad promptu snažícího se vyřešit matematický problém

Jejda. To je očividně špatná odpověď. Ve skutečnosti mají LLM často potíže s
matematickými úkoly a mohou poskytnout nesprávné odpovědi – dokonce i pro tak jednoduchý úkol jako
násobení dvou čísel. Je to proto, že jsou trénovány na velkých objemech textu a matematika
může vyžadovat jiný přístup. Takže se podívejme, zda mezikroky uvažování zlepší
výstup.

```
Prompt              Když mi byly 3 roky, můj partner byl 3krát starší než já. Nyní
                    je mi 20 let. Kolik let je mému partnerovi? Zamysleme se krok
                    za krokem.
Výstup              1. Předpokládejme, že můj současný věk je „x“ let.
                    2. Když mi byly 3 roky, můj věk byl 3 roky.
                    3. Věk mého partnera v té době byl 3krát můj věk, což znamená, že věk mého partnera
                       byl 3 * 3 = 9 let.
                    4. Nyní je mi 20 let, což znamená, že můj věk se zvýšil o 20 - 3 = 17
                       let od doby, kdy mi byly 3 roky.
                    5. Jelikož věk mého partnera byl 9 let, když mi byly 3 roky, a můj věk se
                       zvýšil o 17 let, věk mého partnera se musel také zvýšit o 17 let.
                    6. Proto je současný věk mého partnera 9 + 17 = 26 let.

                    Takže mému partnerovi je 26 let.
```
Tabulka 12. Příklad promptování pomocí Řetězce myšlenek (Chain of Thought)

Dobře, teď je konečná odpověď správná. Je to proto, že jsme LLM jasně instruovali, aby
vysvětlil každý krok, místo aby jen vrátil odpověď. Je zajímavé vidět, že model
sčítá 17 přidaných let. V mé hlavě bych vzal rozdíl let mezi mým
partnerem a mnou a ty přičetl. (20+(9-3)). Pomozme modelu, aby přemýšlel trochu více
jako já.

Tabulka 12 je příkladem „zero-shot“ Řetězce myšlenek. Promptování pomocí Řetězce myšlenek může být
velmi silné, když se zkombinuje s one-shot nebo few-shot, jak můžete vidět v tabulce 13:

```
Prompt              Otázka: Když mému bratrovi byly 2 roky, byl jsem dvakrát starší než on. Nyní
                    je mi 40 let. Kolik let je mému bratrovi? Zamysleme se krok
                    za krokem.
                    Odpověď: Když mému bratrovi byly 2 roky, já jsem měl 2 * 2 = 4 roky.
                    To je věkový rozdíl 2 roky a já jsem starší. Nyní je mi 40
                    let, takže mému bratrovi je 40 - 2 = 38 let. Odpověď
                    je 38.
                    Otázka: Když mi byly 3 roky, můj partner byl 3krát starší než já. Nyní,
                    je mi 20 let. Kolik let je mému partnerovi? Zamysleme se krok
                    za krokem.
                    Odpověď:
Výstup              Když mi byly 3 roky, můj partner měl 3 * 3 = 9 let. To je věkový
                    rozdíl 6 let a můj partner je starší. Nyní je mi 20 let, takže můj
                    partner má 20 + 6 = 26 let. Odpověď je 26.
```
Tabulka 13. Příklad promptování pomocí Řetězce myšlenek s jedním příkladem (one-shot)

Řetězec myšlenek může být užitečný pro různé případy použití. Přemýšlejte o generování kódu, pro rozdělení
požadavku do několika kroků a mapování těchto kroků na specifické řádky kódu. Nebo pro
vytváření syntetických dat, když máte nějaký druh základu jako „ _Produkt se jmenuje XYZ,
napište popis vedoucí model skrze předpoklady, které byste udělali na základě
daného názvu produktu._ “ Obecně platí, že jakýkoli úkol, který lze vyřešit „promluvením si o něm“, je dobrým
kandidátem pro řetězec myšlenek. Pokud dokážete vysvětlit kroky k vyřešení problému, zkuste řetězec
myšlenek.

Prosím, podívejte se na notebook^10 hostovaný v repozitáři GoogleCloudPlatform na Githubu, který
se podrobněji zabývá CoT promptováním:

V sekci osvědčených postupů této kapitoly se naučíme některé osvědčené postupy specifické pro
promptování pomocí Řetězce myšlenek.

### Sebe-konzistence (Self-consistency)

Zatímco velké jazykové modely ukázaly působivý úspěch v různých úkolech NLP, jejich
schopnost uvažovat je často vnímána jako omezení, které nelze překonat pouze zvětšením
velikosti modelu. Jak jsme se naučili v předchozí sekci o promptování pomocí Řetězce myšlenek, model může
být vyzván, aby generoval kroky uvažování jako člověk řešící problém. Avšak CoT používá
jednoduchou strategii „chamtivého dekódování“, což omezuje jeho efektivitu. Sebe-konzistence^11 kombinuje
vzorkování a hlasování většiny k generování rozmanitých cest uvažování a výběru nejvíce
konzistentní odpovědi. Zlepšuje přesnost a koherenci odpovědí generovaných LLM.

Sebe-konzistence dává pseudo-pravděpodobnostní odhad správnosti odpovědi, ale
samozřejmě má vysoké náklady.

Postupuje podle následujících kroků:

1.  Generování rozmanitých cest uvažování: LLM je poskytnut stejný prompt vícekrát.
    Vysoké nastavení teploty povzbuzuje model k generování různých cest
    uvažování a perspektiv na problém.
2.  Extrahujte odpověď z každé generované odpovědi.
3.  Vyberte nejčastější odpověď.

Podívejme se na příklad systému klasifikace e-mailů, který klasifikuje e-mail jako
DŮLEŽITÝ nebo NEDŮLEŽITÝ. Zero-shot prompt řetězce myšlenek bude zaslán LLM
vícekrát, abychom zjistili, zda se odpovědi po každém odeslání liší. Všimněte si přátelského tónu,
volby slov a sarkasmu, který byl použit v e-mailu. To vše by mohlo LLM zmást.

```
Prompt              EMAIL:
                    ```
                    Ahoj,
                    Všiml jsem si, že pro svůj web používáte Wordpress. Skvělý open
                    source systém pro správu obsahu. V minulosti jsem ho také používal.
                    Dodává se se spoustou skvělých uživatelských pluginů. A je docela
                    snadné ho nastavit.
                    Všiml jsem si ale chyby v kontaktním formuláři, ke které dochází, když
                    vyberete pole jméno. Viz přiložený snímek obrazovky, jak
                    zadávám text do pole jméno. Všimněte si JavaScriptového alert
                    okna, které jsem vyv0lal.
                    Ale jinak je to skvělý web. Rád ho čtu. Klidně
                    nechte chybu na webu, protože mi to dává více
                    zajímavých věcí ke čtení.
                    S pozdravem,
                    Harry Hacker.
                    ```
                    Klasifikujte výše uvedený e-mail jako DŮLEŽITÝ nebo NEDŮLEŽITÝ. Zamysleme se
                    krok za krokem a vysvětleme proč.
                    **Pokračuje na další straně...**
```

```
Výstup
Pokus 1

                    **Krok 1: Identifikujte účel e-mailu.**
                    Účelem e-mailu je informovat příjemce o chybě v kontaktním formuláři
                    na jeho webu. Odesílatel také zmiňuje, že rád čte web
                    a že prozatím nechává chybu na místě.
                    **Krok 2: Posuďte potenciální dopad chyby.**
                    Chyba v kontaktním formuláři by mohla útočníkovi umožnit spustit libovolný
                    JavaScriptový kód na webu. To by mohlo vést k tomu, že útočník bude schopen převzít
                    kontrolu nad webem, ukrást citlivé informace nebo web poškodit (deface).
                    **Krok 3: Zvažte důvěryhodnost odesílatele.**
                    Odesílatel e-mailu tvrdí, že je hacker jménem Harry. Neexistuje však
                    žádný způsob, jak ověřit identitu odesílatele. Je možné, že odesílatel je legitimní
                    hacker, který se snaží pomoci příjemci, ale je také možné, že odesílatel
                    je škodlivý aktér, který se snaží přimět příjemce ke kliknutí na škodlivý
                    odkaz nebo stažení škodlivého souboru.
                    **Závěr: DŮLEŽITÝ**
                    Na základě potenciálního dopadu chyby a důvěryhodnosti odesílatele
                    by měl být e-mail klasifikován jako **DŮLEŽITÝ**. Příjemce by měl okamžitě
                    podniknout kroky k opravě chyby a ochraně svého webu před útokem.
```
```
Výstup
Pokus 2

                    **Vysvětlení:**
                    1. **Nedostatek naléhavosti:** E-mail nevyjadřuje žádný pocit naléhavosti nebo
                       časové citlivosti. Začíná neformálním pozdravem a prezentuje informace
                       věcným tónem.
                    2. **Nekritické hlášení chyby:** Chyba hlášená v kontaktním formuláři je
                       popsána jako zajímavá, ale ne kritická. Odesílatel e-mailu dokonce navrhuje, že
                       majitel webu může chybu nechat na místě.
                    3. **Nedostatek osobního dopadu:** E-mail neuvádí žádný přímý nebo
                       osobní dopad na majitele webu. Pouze zmiňuje, že odesílatel
                       rád čte web.
                          **Pokračuje na další straně...**
```

```
Výstup
Pokus 2
                    4. **Absence požadavku na akci:** E-mail explicitně nepožaduje žádnou
                       akci od majitele webu. Nežádá o opravu chyby ani
                       neposkytuje žádné konkrétní návrhy na zlepšení.
                    5. **Záměr odesílatele:** Záměr odesílatele se zdá být více zaměřen na
                       sdílení pozorování a vyjádření zájmu o web než na
                       vznesení vážné obavy nebo žádost o pomoc.
                    **Závěr: NEDŮLEŽITÝ**
```
```
Výstup
Pokus 3

                    **Krok 1: Identifikujte účel e-mailu.**
                    Účelem e-mailu je informovat příjemce o chybě v kontaktním formuláři na
                    jeho webu. Odesílatel také zmiňuje, že rád čte web a
                    že prozatím nechává chybu na místě.
                    **Krok 2: Posuďte potenciální dopad chyby.**
                    Chyba v kontaktním formuláři by mohla útočníkovi umožnit spustit libovolný
                    JavaScriptový kód na webu. To by mohlo vést k tomu, že útočník bude schopen převzít
                    kontrolu nad webem, ukrást citlivé informace nebo web poškodit.
                    **Krok 3: Zvažte důvěryhodnost odesílatele.**
                    Odesílatel e-mailu tvrdí, že je hacker jménem Harry. Neexistuje však
                    žádný způsob, jak ověřit identitu odesílatele. Je možné, že odesílatel je legitimní
                    hacker, který se snaží pomoci příjemci, ale je také možné, že odesílatel je
                    škodlivý aktér, který se snaží zneužít chybu pro svůj vlastní zisk.
                    **Závěr:**
                    Na základě výše uvedených faktorů by měl být e-mail klasifikován jako **DŮLEŽITÝ**.
                    Chyba v kontaktním formuláři představuje vážné bezpečnostní riziko a důvěryhodnost odesílatele
                    je neznámá. Příjemce by měl okamžitě podniknout kroky k opravě chyby a
                    prošetřit tvrzení odesílatele.
```
Tabulka 14. Příklad promptování pro sebe-konzistenci

Můžete použít výše uvedený prompt a zkusit zjistit, zda vrací konzistentní klasifikaci.
V závislosti na modelu, který používáte, a konfiguraci teploty může
vrátit „DŮLEŽITÝ“ nebo „NEDŮLEŽITÝ“.

Generováním mnoha Řetězců myšlenek a přijetím nejčastěji se vyskytující odpovědi
(„DŮLEŽITÝ“) můžeme od LLM získat konzistentněji správnou odpověď.

Tento příklad ukazuje, jak lze promptování pro sebe-konzistenci použít ke zlepšení přesnosti
odpovědi LLM zvážením více perspektiv a výběrem nejvíce
konzistentní odpovědi.

### Strom myšlenek (Tree of Thoughts - ToT)

Nyní, když jsme obeznámeni s promptováním pomocí řetězce myšlenek a sebe-konzistence, podívejme se na
Strom myšlenek (Tree of Thoughts - ToT).^12 Zobecňuje koncept CoT promptování, protože umožňuje LLM
prozkoumávat více různých cest uvažování současně, spíše než jen sledovat
jediný lineární řetězec myšlenek. To je znázorněno na obrázku 1.

Obrázek 1. Vizualizace promptování řetězce myšlenek vlevo vs. promptování Stromu myšlenek vpravo

Tento přístup činí ToT zvláště vhodným pro složité úkoly, které vyžadují průzkum. Funguje
udržováním stromu myšlenek, kde každá myšlenka představuje koherentní jazykovou sekvenci, která
slouží jako mezikrok k řešení problému. Model pak může
prozkoumávat různé cesty uvažování rozvětvováním z různých uzlů ve stromu.

Existuje skvělý notebook, který jde trochu podrobněji a ukazuje Strom myšlenek
(ToT), který je založen na článku „Large Language Model Guided Tree-of-Thought“.^9

### ReAct (reason & act - uvažuj a jednej)

Uvažuj a jednej (ReAct) [10]^13 promptování je paradigma umožňující LLM řešit složité
úkoly pomocí uvažování v přirozeném jazyce v kombinaci s externími nástroji (vyhledávání, interpret
kódu atd.), což LLM umožňuje provádět určité akce, jako je interakce s externími
API k získání informací, což je první krok k modelování agentů.

ReAct napodobuje, jak lidé fungují v reálném světě, protože uvažujeme verbálně a můžeme
podnikat kroky k získání informací. ReAct si vede dobře ve srovnání s jinými přístupy k prompt
engineeringu v různých oblastech.

ReAct promptování funguje kombinací uvažování a jednání do smyčky myšlenka-akce. LLM
nejprve uvažuje o problému a generuje plán akce. Poté provede
akce v plánu a pozoruje výsledky. LLM poté použije pozorování k aktualizaci
svého uvažování a vygenerování nového plánu akce. Tento proces pokračuje, dokud LLM
nedosáhne řešení problému.

Abyste to viděli v akci, musíte napsat nějaký kód. V úryvku kódu 1 používám framework langchain
pro Python, spolu s VertexAI (google-cloud-aiplatform) a
pip balíčky google-search-results.

Pro spuštění tohoto vzorku musíte vytvořit (bezplatný) klíč SerpAPI z https://serpapi.com/manage-
api-key a nastavit proměnnou prostředí SERPAPI_API_KEY.

Dále napíšeme nějaký Python kód s úkolem pro LLM zjistit: _Kolik dětí mají slavní
otcové, kteří vystupují v kapele Metallica._

```python
from langchain.agents import load_tools
from langchain.agents import initialize_agent
from langchain.agents import AgentType
from langchain.llms import VertexAI

prompt = "Kolik dětí mají členové kapely Metallica?"
llm = VertexAI(temperature=0.1)
tools = load_tools(["serpapi"], llm=llm)
agent = initialize_agent(tools, llm,
                         agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True)
agent.run(prompt)
```
Úryvek 1. Vytvoření ReAct Agenta s LangChain a VertexAI

Úryvek kódu 2 ukazuje výsledek. Všimněte si, že ReAct provádí řetězec pěti vyhledávání. Ve skutečnosti
LLM extrahuje výsledky vyhledávání Google, aby zjistil jména členů kapely. Poté vypíše
výsledky jako pozorování a zřetězí myšlenku pro další vyhledávání.

```
> Entering new AgentExecutor chain...
Metallica má 4 členy.
Akce: Vyhledat
Vstup akce: Kolik dětí má James Hetfield?
Pozorování: tři děti
Myšlenka: 1/4 členů kapely Metallica má 3 děti
Akce: Vyhledat
Vstup akce: Kolik dětí má Lars Ulrich?
Pozorování: 3
Myšlenka: 2/4 členů kapely Metallica má 6 dětí
Akce: Vyhledat
Vstup akce: Kolik dětí má Kirk Hammett?
Pozorování: Hammett je od roku 1998 ženatý se svou druhou manželkou Lani.
Mají dva syny, Angela (nar. 29. září 2006) a Vincenza (nar. 28. června
2008).
Myšlenka: 3/4 členů kapely Metallica má 8 dětí
Akce: Vyhledat
Vstup akce: Kolik dětí má Robert Trujillo?
Pozorování: 2
Myšlenka: 4/4 členů kapely Metallica má 10 dětí
Konečná odpověď: 10
```
Úryvek 2. Spuštění ReAct Agenta v terminálu

Úryvek kódu 2 zjišťuje, že kapela Metallica má čtyři členy. Poté vyhledá
každého člena kapely, aby zjistil celkový počet dětí a sečetl celkový součet. Nakonec vrátí
celkový počet dětí jako konečnou odpověď.

ReAct promptování v praxi vyžaduje pochopení, že musíte neustále znovu posílat
předchozí prompty/odpovědi (a ořezávat extra generovaný obsah), stejně jako nastavit
model s vhodnými příklady/instrukcemi. Prosím, podívejte se na notebook^14 hostovaný
v repozitáři GoogleCloudPlatform na Githubu, který jde trochu podrobněji a ukazuje
skutečné vstupy a výstupy LLM s propracovanějším příkladem.

### Automatický prompt engineering (Automatic Prompt Engineering)

V tomto bodě si možná uvědomíte, že psaní promptu může být složité. Nebylo by hezké to
zautomatizovat (napsat prompt k psaní promptů)? No, existuje metoda: Automatický Prompt
Engineering (APE). Tato metoda^15 nejenže zmírňuje potřebu lidského vstupu, ale také
zlepšuje výkon modelu v různých úkolech.

Vyzvete model, aby generoval další prompty. Vyhodnotíte je, případně upravíte ty dobré.
A opakujete.

Například můžete použít automatický prompt engineering, který pomůže s trénováním chatbota
pro e-shop s merchandise tričky. Chceme zjistit všechny různé způsoby, jakými by zákazníci
mohli formulovat svou objednávku na nákup trička kapely.

1.  Napište prompt, který vygeneruje varianty výstupu. V tomto příkladu používám
    gemini-pro k vygenerování 10 instrukcí. Viz tabulka 15:

```
Prompt              Máme e-shop s tričky kapel a pro trénování
                    chatbota potřebujeme různé způsoby, jak objednat: "Jedno tričko Metallica
                    velikost S". Vygenerujte 10 variant se stejnou sémantikou, ale zachovejte
                    stejný význam.
Výstup              1. Chtěl bych si koupit tričko Metallica ve velikosti S.
                    2. Mohu si objednat tričko Metallica velikosti S?
                    3. Chtěl bych koupit jedno tričko Metallica ve velikosti S.
                    4. Jedno tričko Metallica, velikost S, prosím.
                    5. Chci jedno tričko Metallica ve velikosti S.
                    6. Chtěl bych zadat objednávku na jedno tričko Metallica ve velikosti S.
                    7. Malé tričko Metallica, jedno prosím.
                    8. Hledám jedno tričko Metallica ve velikosti S.
                    9. Můžete mi sehnat jedno tričko Metallica ve velikosti S?
                    10. Jedno tričko Metallica malé, prosím.
```
Tabulka 15. Použití LLM k generování promptů

2.  Vyhodnoťte všechny kandidáty na instrukce bodováním kandidátů na základě zvolené
    metriky. Například můžete použít BLEU (Bilingual Evaluation Understudy) nebo ROUGE
    (Recall-Oriented Understudy for Gisting Evaluation).
3.  Vyberte kandidáta na instrukci s nejvyšším hodnocením. Tento kandidát bude
    finálním promptem, který můžete použít ve své softwarové aplikaci nebo chatbotu. Můžete také vyladit
    vybraný prompt a znovu vyhodnotit.

### Promptování pro kód (Code prompting)

Gemini se primárně zaměřuje na textové prompty, což zahrnuje i psaní promptů pro
vracení kódu. Pojďme do Vertex AI Studia a otestujme tyto prompty, abychom se podívali na některé
příklady kódování.

#### Prompty pro psaní kódu

Gemini může být také vývojářem a pomoci vám s psaním kódu v jakémkoli programovacím jazyce
dle vašeho výběru. Jako vývojáři vám to může pomoci zrychlit proces psaní kódu.

Představte si složku na vašem počítači se stovkami souborů, které je třeba přejmenovat.
Přejmenování každého souboru by vám zabralo spoustu času. Znáte trochu Bash a mohli byste napsat
skript k automatizaci tohoto procesu, ale to může také chvíli trvat. Takže napišme prompt. Můžete
napsat prompt ve veřejném spotřebitelském chatbotu Gemini, nebo pokud se více obáváte
o důvěrnost, můžete tyto prompty napsat ve svém účtu Google Cloud a
otevřít Vertex AI Studio. Výhodou Vertex AI Studia je, že můžete konfigurovat
teplotu atd.

```
Název               1_bash_rename_files_in_folder
Cíl                 Napsat prompt pro napsání kódu v Bashi k přejmenování souborů ve složce.
Model               gemini-pro
Teplota             0.1       Limit tokenů      1024
Top - K             N/A       Top - P           1
Prompt              Napište úryvek kódu v Bashi, který se zeptá na název složky.
                    Poté vezme obsah složky a přejmenuje všechny
                    soubory uvnitř přidáním názvu draft k názvu souboru.
```
```bash
Výstup              ```bash
                    #!/bin/bash
                    # Zeptat se na název složky
                    echo "Zadejte název složky: "
                    read folder_name
                    # Zkontrolovat, zda složka existuje
                    if [ ! -d "$folder_name" ]; then
                      echo "Složka neexistuje."
                      exit 1
                    fi
                    # Získat seznam souborů ve složce
                    files=("$folder_name"/*)
                    # Přejmenovat každý soubor přidáním "draft_" k názvu souboru
                    for file in "${files[@]}"; do
                      # Získat pouze název souboru bez cesty
                      filename=$(basename "$file")
                      # Nový název souboru
                      new_file_name="$folder_name/draft_$filename"
                      # Přejmenovat soubor
                      mv "$file" "$new_file_name"
                    done
                    echo "Soubory úspěšně přejmenovány."
                    ```
```
Tabulka 16. Použití Gemini k programování kódu (Pozn. překl.: Mírně upraven kód pro správnou funkci)

To mi připadá jako dobrý kód - je dokonce dokumentovaný! Avšak, jelikož LLM nemohou uvažovat
a opakují trénovací data, je nezbytné váš kód nejprve přečíst a otestovat.

Okamžik, na který všichni čekáme, opravdu to funguje?

Zkusme to nejprve s testovací složkou s pouze několika soubory, které bude třeba přejmenovat z
filename.txt na draft_filename.txt.

1.  Zkopírujte výstup z tabulky 16 (bez obalovacího textu ``` **bash** ```) a vložte jej do nového
    souboru nazvaného: „**rename_files.sh**“.
2.  Otevřete okno terminálu a napište: **bash rename_files.sh**. Zeptá se na zadání názvu složky, např.
    **test**. a stiskněte enter.
3.  Skript se zdá běžet v pořádku. Uvidíte zprávu: **Soubory úspěšně přejmenovány.**
    Když se podíváte do složky test, všimněte si, že všechny soubory byly dokonale přejmenovány na
    **draft_filename.txt.**

Fungovalo to!

#### Prompty pro vysvětlení kódu

Jako vývojář, když pracujete v týmech, musíte číst kód někoho jiného. Gemini vám
s tím může také pomoci. Vezměme kódový výstup z tabulky 16, odstraňme komentáře
a požádejme velký jazykový model, aby vysvětlil, co se děje, viz tabulka 17:

```
Název               1_bash_explain_the_code
Cíl                 Napsat prompt pro vysvětlení kódu v Bashi
Model               gemini-pro
Teplota             0.1       Limit tokenů      1024
Top - K             N/A       Top - P           1
Prompt              Vysvětlete mi níže uvedený kód v Bashi:

                    ```bash
                    #!/bin/bash
                    echo "Zadejte název složky: "
                    read folder_name
                    if [ ! -d "$folder_name" ]; then
                      echo "Složka neexistuje."
                      exit 1
                    fi
                    files=("$folder_name"/*)
                    for file in "${files[@]}"; do
                      filename=$(basename "$file")
                      new_file_name="$folder_name/draft_$filename"
                      mv "$file" "$new_file_name"
                    done
                    echo "Soubory úspěšně přejmenovány."
                    ```
Výstup              Tento Bash skript provádí následující úkoly:

                    1. **Vstup uživatele**:
                       - Vyzve uživatele k zadání názvu složky pomocí příkazů `echo` a `read`.
                         Zadaný název složky je uložen v proměnné `folder_name`.
                    2. **Kontrola existence složky**:
                       - Zkontroluje, zda složka se zadaným názvem existuje pomocí podmínky `[ ! -d
                         "$folder_name" ]`. Pokud složka neexistuje, zobrazí chybovou
                         zprávu („Složka neexistuje.“) a ukončí skript s návratovým kódem 1.
                          **Pokračuje na další straně...**
```

```
Výstup              3. **Výpis souborů**:
                       - Vypíše všechny soubory uvnitř zadané složky pomocí příkazu `files=( "$folder_name"/* )`.
                         Tím se vytvoří pole nazvané `files` obsahující cesty
                         ke všem souborům ve složce.
                    4. **Přejmenování souborů**:
                       - Prochází každý soubor v poli `files` pomocí cyklu `for`.
                       - Pro každý soubor získá jeho základní jméno (bez cesty) pomocí `filename=$(basename "$file")`.
                       - Vygeneruje nový název souboru pomocí příkazu `new_file_name="$folder_name/draft_$filename"`. Tento nový název souboru je vytvořen
                         přidáním prefixu "draft_" k původnímu názvu souboru a zachováním cesty ke složce.
                       - Poté přejmenuje soubor pomocí příkazu `mv "$file" "$new_file_name"`,
                         který přesune soubor na nový název souboru.
                    5. **Zpráva o úspěchu**:
                       - Po přejmenování všech souborů zobrazí zprávu o úspěchu („Soubory úspěšně přejmenovány.“)
                         k označení, že operace přejmenování byla úspěšná.
```
Tabulka 17. Prompt pro vysvětlení kódu

#### Prompty pro překlad kódu

Bash kód z tabulky 16 se zdá fungovat dobře. Avšak tento skript by mohl být skutečně znovupoužitelný,
kdyby mě vyzval k zadání názvů souborů, ideálně by měl fungovat jako samostatná aplikace s
UI. Jako výchozí bod by Python byl lepším jazykem pro (webovou) aplikaci než
Bash. LLM mohou pomoci s překladem kódu z jednoho jazyka do druhého. Viz příklad v
tabulce 18:

```
Název               1_python_rename_files
Cíl                 Napsat prompt pro překlad kódu z Bashe do Pythonu
Model               gemini-pro
Teplota             0.1       Limit tokenů      1024
Top - K             N/A       Top - P           1
Prompt              Přeložte níže uvedený kód v Bashi do úryvku v Pythonu.

                    ```bash
                    #!/bin/bash
                    echo "Zadejte název složky: "
                    read folder_name
                    if [ ! -d "$folder_name" ]; then
                      echo "Složka neexistuje."
                      exit 1
                    fi
                    files=("$folder_name"/*)
                    for file in "${files[@]}"; do
                      filename=$(basename "$file")
                      new_file_name="$folder_name/draft_$filename"
                      mv "$file" "$new_file_name"
                    done
                    echo "Soubory úspěšně přejmenovány."
                    ```
Výstup              ```python
                    import os
                    import shutil

                    # Získat název složky od uživatele
                    folder_name = input("Zadejte název složky: ")

                    # Zkontrolovat, zda složka existuje
                    if not os.path.isdir(folder_name):
                        print("Složka neexistuje.")
                        exit(1)

                    # Získat seznam všech souborů ve složce
                    files = os.listdir(folder_name)

                    # Iterovat přes soubory
                    for file in files:
                        # Vytvořit plnou cestu k původnímu souboru
                        old_path = os.path.join(folder_name, file)
                        # Zkontrolovat, zda se jedná o soubor (ne složku)
                        if os.path.isfile(old_path):
                            # Vytvořit nový název souboru s prefixem "draft_"
                            new_file_name = f"draft_{file}"
                            # Vytvořit plnou cestu k novému souboru
                            new_path = os.path.join(folder_name, new_file_name)
                            # Přesunout (přejmenovat) soubor
                            shutil.move(old_path, new_path)

                    # Vytisknout zprávu o úspěchu
                    print("Soubory úspěšně přejmenovány.")
                    ```
                    **Pokračuje na další straně...** (Pozn. překl.: Zde není další strana, kód končí)
```
Tabulka 18. Prompt pro překlad kódu z Bashe do Pythonu

Přečtěte si a zkontrolujte kód. Zkopírujte výstup z promptu a vložte jej do nového souboru:
**file_renamer.py**. Otestujte kód otevřením okna terminálu a spusťte následující
příkaz **python file_renamer.py.**

**POZNÁMKA:** Při promptování pro (Python) kód v Language Studiu ve Vertex AI budete muset
kliknout na tlačítko ‚Markdown‘. Jinak obdržíte prostý text, který postrádá
správné odsazení řádků, což je důležité pro spuštění Python kódu.

#### Prompty pro ladění a revizi kódu

Ručně provedeme některé úpravy kódu z tabulky 18. Měl by uživatele vyzvat k zadání
prefixu názvu souboru a tento prefix napsat velkými písmeny. Viz příklad kódu v
úryvku *3, ale jaká smůla. Nyní vrací chyby Pythonu!

```python
import os
import shutil

folder_name = input("Zadejte název složky: ")
prefix = input("Zadejte řetězec pro přidání na začátek názvu souboru: ")
text = toUpperCase(prefix) # Chyba zde

if not os.path.isdir(folder_name):
    print("Složka neexistuje.")
    exit(1)

files = os.listdir(folder_name)
for file in files:
    old_path = os.path.join(folder_name, file)
    if os.path.isfile(old_path):
        # Chyba zde v názvu proměnné
        new_filename = f"{text}_{file}"
        # Chyba zde v názvu proměnné
        new_path = os.path.join(folder_name, new_file_name) # Mělo by být new_filename
        shutil.move(old_path, new_path)

print("Soubory úspěšně přejmenovány.")

```
Úryvek 3. Rozbitý Python skript

Jejda! To vypadá jako chyba:

```
Níže uvedený Python kód vrací chybu:
Traceback (most recent call last):
  File "/Users/leeboonstra/Documents/test_folder/rename_files.py", line 7,
    in <module>
    text = toUpperCase(prefix)
NameError: name ‘toUpperCase’ is not defined
```
Úryvek 4. Rozbil jsem Python kód

Podívejme se, zda můžeme požádat velký jazykový model o ladění a revizi kódu. Podívejte se do
tabulky 19:

```
Název               1_python_debug_code
Cíl                 Napsat prompt pro ladění a revizi Python kódu.
Model               gemini-pro
Teplota             0.1       Limit tokenů      1024
Top - K             N/A       Top - P           1
Prompt              Níže uvedený Python kód vrací chybu:

                    ```
                    Traceback (most recent call last):
                      File "/Users/leeboonstra/Documents/test_folder/rename_files.py", line 7,
                        in <module>
                        text = toUpperCase(prefix)
                    NameError: name 'toUpperCase' is not defined
                    ```
                    Odlaďte, co je špatně, a vysvětlete, jak mohu kód vylepšit.
                    ```python
                    import os
                    import shutil
                    # Získat název složky od uživatele
                    folder_name = input("Zadejte název složky: ")
                    prefix = input("Zadejte řetězec pro přidání na začátek názvu souboru: ")
                    text = toUpperCase(prefix)
                    # Zkontrolovat, zda složka existuje
                    if not os.path.isdir(folder_name):
                        print("Složka neexistuje.")
                        exit(1)
                    # Získat seznam všech souborů ve složce
                    files = os.listdir(folder_name)
                    # Iterovat přes soubory
                    for file in files:
                        # Vytvořit nový název souboru s prefixem
                        new_filename = f"{text}_{file}"
                        # Přesunout soubor na nový název
                        old_path = os.path.join(folder_name, file)
                        # !! Zde je další potenciální chyba v původním kódu z Úryvku 3 !!
                        # new_path = os.path.join(folder_name, new_file_name) # Mělo by být new_filename
                        # Opraveno pro konzistenci s chybou výše:
                        new_path = os.path.join(folder_name, new_filename)
                        # Zkontrolovat, zda je to soubor před přesunem
                        if os.path.isfile(old_path):
                             shutil.move(old_path, new_path)
                    # Vytisknout zprávu o úspěchu
                    print("Soubory úspěšně přejmenovány.")
                    ```
                    Pokračuje na další straně...
```

```
Výstup              Kód volá funkci `toUpperCase` pro převod `prefix` na velká písmena, ale
                    tato funkce není definována. K opravě tohoto problému můžete použít metodu `upper()`
                    třídy string, která převádí daný řetězec na velká písmena. Upravený kód
                    je uveden níže:
                    ```python
                    import os
                    import shutil
                    # Získat název složky od uživatele
                    folder_name = input("Zadejte název složky: ")
                    prefix = input("Zadejte řetězec pro přidání na začátek názvu souboru: ")
                    text = prefix.upper() # Opraveno zde
                    # Zkontrolovat, zda složka existuje
                    if not os.path.isdir(folder_name):
                        print("Složka neexistuje.")
                        exit(1)
                    # Získat seznam všech souborů ve složce
                    files = os.listdir(folder_name)
                    # Iterovat přes soubory
                    for file in files:
                        # Vytvořit plnou cestu k původnímu souboru
                        old_path = os.path.join(folder_name, file)
                        # Zkontrolovat, zda se jedná o soubor
                        if os.path.isfile(old_path):
                            # Vytvořit nový název souboru s prefixem
                            new_filename = f"{text}_{file}"
                            # Vytvořit plnou cestu k novému souboru
                            new_path = os.path.join(folder_name, new_filename)
                            # Přesunout soubor na nový název
                            shutil.move(old_path, new_path)
                    # Vytisknout zprávu o úspěchu
                    print("Soubory úspěšně přejmenovány.")
                    ```
                    **Pokračuje na další straně...**
```

```
Výstup              Navíc existuje několik dalších vylepšení, která lze v kódu provést:
                    1. Kód nekontroluje, zda položky ve složce jsou soubory nebo jiné složky. Je lepší explicitně kontrolovat pomocí `os.path.isfile()`, aby se přejmenovaly pouze soubory.
                    2. Přípona souboru nových názvů souborů není zachována. Je lepší extrahovat
                       příponu z původního názvu souboru a připojit ji k novému názvu souboru.
                    3. Kód nepředpokládá, že v názvu složky nejsou žádné mezery. Avšak
                       je dobrým zvykem zpracovávat mezery elegantně nahrazením je
                       podtržítky nebo pomlčkami (i když `os.path.join` by si s tím měl poradit).
                    4. Použití syntaxe `f-string` pro interpolaci řetězců je čitelnější a
                       stručnější než tradiční operátor `+`.
                    5. Kód nezpracovává chyby, které by mohly nastat během procesu přejmenování.
                       Bylo by lepší zabalit volání `shutil.move` do bloku `try...except`, aby
                       se zachytily jakékoli potenciální chyby.
                    Zde je vylepšený kód s těmito návrhy:
                    ```python
                    import os
                    import shutil

                    # Získat název složky od uživatele
                    folder_name = input("Zadejte název složky: ")
                    prefix = input("Zadejte řetězec pro přidání na začátek názvu souboru: ").upper() # Převod na velká písmena hned

                    # Zkontrolovat, zda složka existuje
                    if not os.path.isdir(folder_name):
                        print("Složka neexistuje.")
                        exit(1)

                    # Získat seznam všech položek ve složce
                    try:
                        items = os.listdir(folder_name)
                    except OSError as e:
                        print(f"Chyba při čtení složky {folder_name}: {e}")
                        exit(1)

                    renamed_count = 0
                    # Iterovat přes položky
                    for item in items:
                        old_path = os.path.join(folder_name, item)
                        # Zkontrolovat, zda se jedná o soubor
                        if os.path.isfile(old_path):
                           # Získat název souboru a příponu
                           base, ext = os.path.splitext(item)
                           # Vytvořit nový název souboru s prefixem a příponou
                           new_filename = f"{prefix}_{base}{ext}"
                           # Sestavit plné cesty pro původní a nové soubory
                           new_path = os.path.join(folder_name, new_filename)
                           # Pokusit se přesunout soubor
                           try:
                               shutil.move(old_path, new_path)
                               renamed_count += 1
                           except Exception as e:
                               print(f"Při přesunu souboru {item} došlo k chybě: {e}")

                    # Vytisknout zprávu o úspěchu
                    if renamed_count > 0:
                        print(f"{renamed_count} souborů úspěšně přejmenováno.")
                    else:
                        print("Žádné soubory k přejmenování nebyly nalezeny nebo přejmenovány.")

                    ```
                    Tato vylepšení činí kód robustnějším a flexibilnějším a zároveň usnadňují
                    jeho čtení a pochopení... (Odpověď byla zkrácena, protože
                    dosáhla limitu tokenů. Zkuste zvýšit limit tokenů, pokud potřebujete
                    delší odpověď.)
```
Tabulka 19. Prompt pro ladění a revizi Python kódu

To je skvělé. Nejenže mi řekl, jak problém vyřešit, ale také zjistil, že můj kód
měl více chyb a jak je vyřešit. Poslední část promptu poskytla návrhy na
obecné vylepšení kódu.

#### A co multimodální promptování?

Promptování pro kód stále používá stejný běžný velký jazykový model. Multimodální promptování
je samostatná záležitost, odkazuje na techniku, kde používáte více vstupních formátů k vedení
velkého jazykového modelu, místo abyste se spoléhali pouze na text. To může zahrnovat kombinace textu,
obrázků, zvuku, kódu nebo dokonce jiných formátů, v závislosti na schopnostech modelu a
daném úkolu.

## Osvědčené postupy (Best Practices)

Nalezení správného promptu vyžaduje ladění. Language Studio ve Vertex AI je perfektním místem pro
hraní si s vašimi prompty, s možností testovat proti různým modelům.

Použijte následující osvědčené postupy, abyste se stali profesionálem v prompt engineeringu.

### Poskytněte příklady

Nejdůležitějším osvědčeným postupem je poskytnout (one-shot / few-shot) příklady v rámci
promptu. Je to vysoce efektivní, protože to funguje jako silný výukový nástroj. Tyto příklady
ukazují požadované výstupy nebo podobné odpovědi, což umožňuje modelu se z nich učit
a přizpůsobit tomu své vlastní generování. Je to jako dát modelu referenční bod nebo
cíl, na který má mířit, což zlepšuje přesnost, styl a tón jeho odpovědi, aby lépe odpovídala
vašim očekáváním.

### Navrhujte s jednoduchostí

Prompty by měly být stručné, jasné a snadno pochopitelné jak pro vás, tak pro model. Jako
základní pravidlo platí, že pokud je to již matoucí pro vás, pravděpodobně to bude matoucí i pro model. Zkuste
nepoužívat složitý jazyk a neposkytujte zbytečné informace.

Příklady:

PŘEDTÍM:

    Momentálně navštěvuji New York a rád bych slyšel více o skvělých
    lokalitách. Jsem se dvěma 3letými dětmi. Kam bychom měli jít během
    naší dovolené?

PO PŘEPSÁNÍ:

    Vystupuj jako cestovní průvodce pro turisty. Popiš skvělá místa k návštěvě v
    New Yorku na Manhattanu s 3letým dítětem.

Zkuste používat slovesa, která popisují akci. Zde je sada příkladů:

**Analyzuj, Doporuč, Definuj, Extrahuj, Generuj, Identifikuj, Klasifikuj, Kategorizuj, Měř, Najdi, Napiš, Organizuj, Parsovat, Poskytni, Porovnej (Compare), Porovnej (Contrast), Předpovídej, Přelož, Přepiš, Seřaď (Rank), Seřaď (Sort), Shrň, Ukázat, Urči, Vytvoř, Vyber (Pick), Vyber (Select), Vyhodnoť, Vystupuj jako (Act as), Získej (Retrieve).**

### Buďte specifičtí ohledně výstupu

Buďte specifičtí ohledně požadovaného výstupu. Stručná instrukce nemusí LLM dostatečně navést
nebo může být příliš obecná. Poskytnutí specifických detailů v promptu (prostřednictvím systémového nebo kontextuálního
promptování) může modelu pomoci zaměřit se na to, co je relevantní, a zlepšit tak celkovou přesnost.

Příklady:

DĚLEJTE:

    Vygeneruj 3odstavcový blogový příspěvek o top 5 videoherních konzolích.
    Blogový příspěvek by měl být informativní a poutavý a měl by být
    napsán konverzačním stylem.

NEDĚLEJTE:

    Vygeneruj blogový příspěvek o videoherních konzolích.

### Používejte instrukce místo omezení

Instrukce a omezení se používají v promptování k řízení výstupu LLM.

-   **Instrukce** poskytuje explicitní pokyny ohledně požadovaného formátu, stylu nebo obsahu
    odpovědi. Vede model k tomu, co by měl dělat nebo produkovat.
-   **Omezení** je soubor limitů nebo hranic pro odpověď. Omezuje to, co by model
    neměl dělat nebo čemu by se měl vyhnout.

Rostoucí výzkum naznačuje, že zaměření na pozitivní instrukce v promptování může být efektivnější
než silné spoléhání na omezení. Tento přístup odpovídá tomu, jak lidé preferují
pozitivní instrukce před seznamy toho, co nedělat.

Instrukce přímo komunikují požadovaný výsledek, zatímco omezení mohou nechat
model hádat, co je povoleno. Dává flexibilitu a podporuje kreativitu v rámci
definovaných hranic, zatímco omezení mohou omezit potenciál modelu. Také seznam omezení
se může vzájemně střetávat.

Omezení jsou stále cenná, ale v určitých situacích. Aby se zabránilo generování
škodlivého nebo zkresleného obsahu modelem nebo když je potřeba striktní formát nebo styl výstupu.

Pokud je to možné, používejte pozitivní instrukce: místo abyste modelu říkali, co nedělat, řekněte mu, co má
dělat místo toho. To může předejít zmatení a zlepšit přesnost výstupu.

DĚLEJTE:

    Vygeneruj 1odstavcový blogový příspěvek o top 5 videoherních konzolích.
    Diskutuj pouze o konzoli, společnosti, která ji vyrobila, roce a celkových
    prodejích.

NEDĚLEJTE:

    Vygeneruj 1odstavcový blogový příspěvek o top 5 videoherních konzolích.
    Neuváděj názvy videoher.

Jako osvědčený postup začněte upřednostňováním instrukcí, jasně uveďte, co chcete, aby model
udělal, a používejte omezení pouze tehdy, když je to nutné pro bezpečnost, srozumitelnost nebo specifické požadavky.
Experimentujte a iterujte, abyste otestovali různé kombinace instrukcí a omezení a zjistili,
co nejlépe funguje pro vaše specifické úkoly, a zdokumentujte je.

### Kontrolujte maximální délku tokenů

Pro kontrolu délky generované odpovědi LLM můžete buď nastavit maximální limit tokenů v
konfiguraci, nebo explicitně požadovat specifickou délku ve vašem promptu. Například:

_"Vysvětli kvantovou fyziku ve zprávě o délce tweetu."_

### Používejte proměnné v promptech

Pro znovupoužití promptů a jejich dynamičtější povahu používejte v promptu proměnné, které lze
měnit pro různé vstupy. Např. jak je ukázáno v tabulce 20, prompt, který poskytuje fakta o
městě. Místo pevného zakódování názvu města v promptu použijte proměnnou. Proměnné vám mohou ušetřit
čas a úsilí tím, že vám umožní vyhnout se opakování. Pokud potřebujete použít stejný
kus informace ve více promptech, můžete jej uložit do proměnné a poté na
tuto proměnnou odkazovat v každém promptu. To dává velký smysl při integraci promptů do vašich
vlastních aplikací.

```
Prompt              PROMĚNNÉ
                    {mesto} = "Amsterdam"
                    PROMPT
                    Jsi cestovní průvodce. Řekni mi fakt o městě: {mesto}
Výstup              Amsterdam je krásné město plné kanálů, mostů a úzkých uliček. Je to
                    skvělé místo k návštěvě pro jeho bohatou historii, kulturu a noční život.
```
Tabulka 20. Použití proměnných v promptech

### Experimentujte s formáty vstupu a styly psaní

Různé modely, konfigurace modelů, formáty promptů, volby slov a odeslání mohou přinést
různé výsledky. Proto je důležité experimentovat s atributy promptu, jako je styl,
volba slov a typ promptu (zero-shot, few-shot, systémový prompt).

Například prompt s cílem generovat text o revoluční videoherní
konzoli Sega Dreamcast může být formulován jako **otázka**, **tvrzení** nebo **instrukce**,
což vede k různým výstupům:

-   **Otázka:** Co byla Sega Dreamcast a proč to byla tak revoluční konzole?
-   **Tvrzení:** Sega Dreamcast byla videoherní konzole šesté generace vydaná společností
    Sega v roce 1999. Byla...
-   **Instrukce:** Napište jeden odstavec, který popisuje konzoli Sega Dreamcast a
    vysvětluje, proč byla tak revoluční.

### Pro few-shot promptování u klasifikačních úloh, promíchejte třídy

Obecně řečeno, na pořadí vašich few-shot příkladů by nemělo příliš záležet. Avšak
při provádění klasifikačních úkolů se ujistěte, že promícháte možné třídy odpovědí v
few-shot příkladech. Je to proto, že byste jinak mohli přeučit (overfitting) na specifické pořadí
příkladů. Promícháním možných tříd odpovědí můžete zajistit, že se model
učí identifikovat klíčové rysy každé třídy, spíše než si jen pamatovat pořadí
příkladů. To povede k robustnějšímu a zobecnitelnějšímu výkonu na neviděných datech.

Dobré základní pravidlo je začít s 6 few-shot příklady a odtud začít testovat přesnost.

### Přizpůsobte se aktualizacím modelu

Je důležité, abyste měli přehled o změnách architektury modelu, přidaných datech a
schopnostech. Vyzkoušejte novější verze modelu a upravte své prompty, abyste lépe využili nové
funkce modelu. Nástroje jako Vertex AI Studio jsou skvělé pro ukládání, testování a dokumentování různých
verzí vašeho promptu.

### Experimentujte s formáty výstupu

Kromě formátu vstupu promptu zvažte experimentování s formátem výstupu. Pro nekreativní
úkoly, jako je extrakce, výběr, parsování, řazení (ordering), hodnocení (ranking) nebo kategorizace dat, zkuste
nechat váš výstup vrátit ve strukturovaném formátu jako JSON nebo XML.

Existují určité výhody vracení objektů JSON z promptu, který extrahuje data. V
reálné aplikaci nemusím tento formát JSON vytvářet ručně, mohu již
vrátit data v seřazeném pořadí (velmi užitečné při práci s objekty datetime), ale co je nejdůležitější,
promptováním pro formát JSON nutí model vytvořit strukturu a
omezit halucinace.

Shrnutí výhod použití JSON pro váš výstup:

-   Vrací se vždy ve stejném stylu
-   Zaměření na data, která chcete obdržet
-   Menší šance na halucinace
-   Uvědomuje si vztahy
-   Získáte datové typy
-   Můžete to seřadit

Tabulka 4 v sekci few-shot promptování ukazuje příklad, jak vrátit
strukturovaný výstup.

### Oprava JSON (JSON Repair)

Zatímco vracení dat ve formátu JSON nabízí četné výhody, není to bez
nevýhod. Strukturovaná povaha JSONu, i když je výhodná pro parsování a použití v
aplikacích, vyžaduje výrazně více tokenů než prostý text, což vede ke zvýšené
době zpracování a vyšším nákladům. Navíc, mnohomluvnost JSONu může snadno spotřebovat celé
výstupní okno, což se stává obzvláště problematickým, když je generování náhle přerušeno
kvůli limitům tokenů. Toto zkrácení často vede k neplatnému JSONu, chybějícím klíčovým uzavíracím
složeným nebo hranatým závorkám, což činí výstup nepoužitelným. Naštěstí nástroje jako knihovna `json-repair`
(dostupná na PyPI) mohou být v těchto situacích neocenitelné. Tato knihovna se inteligentně
pokouší automaticky opravit neúplné nebo špatně formátované objekty JSON, což z ní činí klíčového
spojence při práci s LLM generovaným JSONem, zejména při řešení potenciálních
problémů se zkrácením.

### Práce se schématy (Schemas)

Použití strukturovaného JSONu jako výstupu je skvělé řešení, jak jsme viděli v tomto
dokumentu vícekrát. Ale co _vstup_? Zatímco JSON je vynikající pro strukturování _výstupu_, který LLM
generuje, může být také neuvěřitelně užitečný pro strukturování _vstupu_, který poskytujete. Zde
přicházejí do hry JSON Schémata. JSON Schéma definuje očekávanou strukturu a datové
typy vašeho JSON vstupu. Poskytnutím schématu dáváte LLM jasný plán
dat, která by měl očekávat, což mu pomáhá zaměřit jeho _pozornost_ na relevantní informace a snižuje
riziko nesprávné interpretace vstupu. Navíc schémata mohou pomoci stanovit vztahy
mezi různými kusy dat a dokonce učinit LLM „časově citlivým“ zahrnutím polí data nebo
časového razítka se specifickými formáty.

Zde je jednoduchý příklad:

Řekněme, že chcete použít LLM ke generování popisů produktů v e-commerce
katalogu. Místo pouhého poskytnutí volného textového popisu produktu můžete použít
JSON schéma k definování atributů produktu:

Úryvek 5. Definice schématu strukturovaného výstupu

```json
{
  "type": "object",
  "properties": {
    "name": { "type": "string", "description": "Název produktu" },
    "category": { "type": "string", "description": "Kategorie produktu" },
    "price": { "type": "number", "format": "float", "description": "Cena produktu" },
    "features": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Klíčové vlastnosti produktu"
    },
    "release_date": { "type": "string", "format": "date", "description": "Datum vydání produktu"}
  },
  "required": ["name", "category", "price"] // Příklad povinných polí
}
```

Poté můžete poskytnout skutečná data produktu jako JSON objekt, který odpovídá
tomuto schématu:

Úryvek 6. Strukturovaný výstup z LLM (Pozn. překl.: Zde by měl být příklad *vstupu* pro LLM podle schématu)

```json
{
  "name": "Bezdrátová sluchátka",
  "category": "Elektronika",
  "price": 99.99,
  "features": ["Potlačení hluku", "Bluetooth 5.0", "20hodinová výdrž baterie"],
  "release_date": "2023-10-27"
}
```

Předzpracováním vašich dat a místo poskytování celých dokumentů poskytnutím jak
schématu, tak dat, dáváte LLM jasné pochopení atributů produktu,
včetně data jeho vydání, což mnohem pravděpodobněji povede ke generování přesného a relevantního
popisu. Tento přístup strukturovaného vstupu, který vede pozornost LLM k relevantním polím,
je obzvláště cenný při práci s velkými objemy dat nebo při integraci LLM do
komplexních aplikací.

### Experimentujte společně s ostatními prompt inženýry

Pokud jste v situaci, kdy se musíte pokusit přijít s dobrým promptem, možná budete
chtít najít více lidí, aby se o to pokusili. Když všichni dodržují osvědčené postupy
(jak jsou uvedeny v této kapitole), uvidíte rozdíly ve výkonu mezi všemi
různými pokusy s prompty.

### Osvědčené postupy pro CoT

Pro CoT promptování je nutné umístit odpověď až za uvažování, protože
generování uvažování mění tokeny, které model dostane, když předpovídá
konečnou odpověď.

S CoT a sebe-konzistencí musíte být schopni extrahovat konečnou odpověď z vašeho
promptu, odděleně od uvažování.

Pro CoT promptování nastavte teplotu na 0.

Promptování řetězce myšlenek je založeno na chamtivém dekódování, předpovídání dalšího slova v
sekvenci na základě nejvyšší pravděpodobnosti přiřazené jazykovým modelem. Obecně
řečeno, při použití uvažování k dosažení konečné odpovědi pravděpodobně existuje jedna jediná
správná odpověď. Proto by teplota měla být vždy nastavena na 0.

### Dokumentujte různé pokusy s prompty

Poslední tip byl zmíněn již dříve v této kapitole, ale nemůžeme dostatečně zdůraznit, jak důležité
je: dokumentujte své pokusy s prompty v plných detailech, abyste se mohli časem naučit, co se povedlo
a co ne.

Výstupy promptů se mohou lišit mezi modely, mezi nastaveními vzorkování a dokonce i mezi různými
verzemi stejného modelu. Navíc i u identických promptů pro stejný model
mohou nastat malé rozdíly ve formátování výstupních vět a volbě slov. (Například, jak
bylo zmíněno dříve, pokud mají dva tokeny stejnou predikovanou pravděpodobnost, remízy mohou být řešeny
náhodně. To pak může ovlivnit následné predikované tokeny.).

Doporučujeme vytvořit Google Sheet s tabulkou 21 jako šablonou. Výhody
tohoto přístupu jsou, že máte kompletní záznam, když se nevyhnutelně budete muset vrátit ke své
práci s promptováním – buď abyste ji v budoucnu navázali (byli byste překvapeni, kolik toho můžete
zapomenout už po krátké přestávce), abyste otestovali výkon promptu na různých verzích modelu,
a abyste pomohli ladit budoucí chyby.

Kromě polí v této tabulce je také užitečné sledovat verzi promptu (iteraci),
pole pro zachycení, zda byl výsledek OK/NE OK/NĚKDY OK, a pole pro zachycení
zpětné vazby. Pokud máte to štěstí, že používáte Vertex AI Studio, ukládejte své prompty (použijte
stejný název a verzi, jaké jsou uvedeny ve vaší dokumentaci) a sledujte hypertextový odkaz na uložený
prompt v tabulce. Tímto způsobem jste vždy jen jedno kliknutí od opětovného spuštění vašich promptů.

Při práci na systému _retrieval augmented generation_ (RAG) byste měli také zachytit
specifické aspekty RAG systému, které ovlivňují, jaký obsah byl vložen do promptu,
včetně dotazu, nastavení chunků, výstupu chunků a dalších informací.

Jakmile máte pocit, že je prompt blízko dokonalosti, přeneste jej do kódové základny vašeho projektu. A v
kódové základně ukládejte prompty do samostatného souboru od kódu, aby bylo snazší je udržovat. Nakonec, ideálně
jsou vaše prompty součástí operacionalizovaného systému a jako prompt inženýr byste se měli
spoléhat na automatizované testy a evaluační procedury, abyste pochopili, jak dobře váš prompt
zobecňuje na daný úkol.

Prompt engineering je iterativní proces. Vytvářejte a testujte různé prompty, analyzujte
a dokumentujte výsledky. Vylaďte svůj prompt na základě výkonu modelu. Pokračujte v
experimentování, dokud nedosáhnete požadovaného výstupu. Když změníte model nebo konfiguraci modelu,
vraťte se a pokračujte v experimentování s dříve použitými prompty.

```
Název               [název a verze vašeho promptu]
Cíl                 [Jedna věta vysvětlující cíl tohoto pokusu]
Model               [název a verze použitého modelu]
Teplota             [hodnota mezi 0 - 1] Limit tokenů [číslo]
Top - K             [číslo]            Top - P [číslo]
Prompt              [Napište celý kompletní prompt]
Výstup              [Vypište výstup nebo více výstupů]
```
Tabulka 21. Šablona pro dokumentování promptů

## Shrnutí

Tento whitepaper pojednává o prompt engineeringu. Naučili jsme se různé techniky promptování,
jako jsou:

-   Zero-shot promptování
-   Few-shot promptování
-   Systémové promptování
-   Rolí řízené promptování
-   Kontextuální promptování
-   Promptování s krokem zpět
-   Řetězec myšlenek
-   Sebe-konzistence
-   Strom myšlenek
-   ReAct

Dokonce jsme se podívali na způsoby, jak můžete automatizovat své prompty.

Whitepaper poté diskutuje výzvy gen AI, jako jsou problémy, které mohou nastat,
když jsou vaše prompty nedostatečné. Zakončili jsme osvědčenými postupy, jak se stát lepším
prompt inženýrem.

## Poznámky na konci

1.  Google, 2023, Gemini by Google. Dostupné na: https://gemini.google.com.
2.  Google, 2024, Gemini for Google Workspace Prompt Guide. Dostupné na:
    https://inthecloud.withgoogle.com/gemini-for-google-workspace-prompt-guide/dl-cd.html.
3.  Google Cloud, 2023, Introduction to Prompting. Dostupné na:
    https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/introduction-prompt-design.
4.  Google Cloud, 2023, Text Model Request Body: Top-P & top-K sampling methods. Dostupné na:
    https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/text#request_body.
5.  Wei, J., et al., 2023, Zero Shot - Fine Tuned language models are zero shot learners. Dostupné na:
    https://arxiv.org/pdf/2109.01652.pdf.
6.  Google Cloud, 2023, Google Cloud Model Garden. Dostupné na: https://cloud.google.com/model-garden.
7.  Brown, T., et al., 2023, Few Shot - Language Models are Few Shot learners. Dostupné na:
    https://arxiv.org/pdf/2005.14165.pdf.
8.  Zheng, L., et al., 2023, Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models.
    Dostupné na: https://openreview.net/pdf?id=3bq3jsvcQ1
9.  Wei, J., et al., 2023, Chain of Thought Prompting. Dostupné na: https://arxiv.org/pdf/2201.11903.pdf.
10. Google Cloud Platform, 2023, Chain of Thought and React. Dostupné na: https://github.com/
    GoogleCloudPlatform/generative-ai/blob/main/language/prompts/examples/chain_of_thought_react.ipynb.
11. Wang, X., et al., 2023, Self Consistency Improves Chain of Thought reasoning in language models.
    Dostupné na: https://arxiv.org/pdf/2203.11171.pdf.
12. Yao, S., et al., 2023, Tree of Thoughts: Deliberate Problem Solving with Large Language Models.
    Dostupné na: https://arxiv.org/pdf/2305.10601.pdf.
13. Yao, S., et al., 2023, ReAct: Synergizing Reasoning and Acting in Language Models. Dostupné na:
    https://arxiv.org/pdf/2210.03629.pdf.
14. Google Cloud Platform, 2023, Advance Prompting: Chain of Thought and React. Dostupné na:
    https://github.com/GoogleCloudPlatform/applied-ai-engineering-samples/blob/main/genai-
    on-vertex-ai/advanced_prompting_training/cot_react.ipynb.
15. Zhou, C., et al., 2023, Automatic Prompt Engineering - Large Language Models are Human-Level Prompt
    Engineers. Dostupné na: https://arxiv.org/pdf/2211.01910.pdf.
