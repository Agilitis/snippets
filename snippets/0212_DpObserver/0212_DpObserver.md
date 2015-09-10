---
layout: default
---

## Observer vagy publish/subscribe (FONTOS)

### Bevezető példa

Tegyük fel, hogy egy HTML szerkesztő alkalmazást készítünk. Azt szeretnénk, hogy két nézet is legyen egyszerre: az egyik a HTML forráskódot mutatja, a másik pedig a kirenderelt weboldalt. Ilyenkor teljesen jogos elvárás az, hogy ha a forrás nézetben egy H2 taget átírok H1-re, akkor a másik nézetben azonnal lássa, hogy a fejléc betűtípusa ettől megváltozik. Ha pedig a kész weboldal nézetében egy sort átszínezünk kékre, akkor ez a forráskódban is azonnal jelenjen meg.

Egy ilyen alkalmazásban tipikusan a document - view megközelítést alkalmazzuk, ahol egy dokumentum és két hozzá kapcsolódó nézet van. Az Observer design pattern lényege, hogy a nézetek figyelik a dokumentumot, és ha az megváltozik, arról szeretnének azonnal tudomást szerezni, hogy ennek megfelelően módosíthassák a megjelenésüket a felhasználói felületen. Ezt úgy érik el, hogy az observerek feliratkoznak a dokumentum változását jelző eseményekre, így ha a dokumentum változik, minden feliratkozott observer kap róla értesítést.

    class Document;

    class IObserver
    {
    public:
        onDocumentChanged(const Document& doc)
    }

    class Document
    {
    public:
        void registerObserver(IObserver& obs)
        {
          observers.push_back(obs);
        }

    private:
        std::vector<IObserver&> observers;

        void notifyAllObservers() const
        {
            for(auto& obs : observers)
            {
              obs.onDocumentChanged(* this);
            }
        }
    }

A *notifyAllObservers()* metódus azért private, mert minden esetben a dokumentum kezdeményezi az observerek tájékoztatását, miután megváltozott a tartalma.

### Részletek

Az observer design pattern tipikusan egy-több függőségeket kezel le, ahol egy példány változásairól minden függő példány értesül. Szokták ezt még publish/subscribe tervezési mintának is nevezni.

Az implementációra kiváló megoldást ad a Qt signals-and-slots rendszere is.

Előnyök:

* Laza csatolás, a dokumentumnak mindegy, hányan figyelik.
* Jól skálázható, mivel sok feliratkozást is könnyen tud kezelni a megoldás.

Hátrányok:

* Elég nehéz módosítani az üzenetek formátumát, így előfordulhat, hogy az idővel szűk keresztmetszet lesz: egyre több mindenről derül ki, hogy azt is át kellene adni az observernek.

### Példa: robotban szenzor adatok frissítése

Egy autonóm mobil robotban a sok fedélzeti szenzor értékét vagy folyamatosan pollozni kell (vagyis periodikusan lekérdezni, hogy változott-e), vagy ki lehet alakítani egy observeres megoldást, ahol a szenzor értesíti a regisztrált observereket. Vagy akkor, ha (1) új mérési eredmény áll rendelkezésre, vagy (2) ha az új eredmény jelentősen el is tér az előzőtől.

(A 2. esetben az observer regisztrációjánál lehet, hogy azt is meg kell tudni adni, hogy az adott observer mekkora változás esetén kapjon értesítést. Annak ugyanis kicsi az esélye, hogy egy szenzor mindig pontosan ugyanazt méri, így ilyen küszöbszint nélkül folyamatosan kapnánk az értesítéseket. Ha ezt a küszöbszintet observerenként lehet állítani, akkor az egyes alrendszerek különböző érzékenységgel figyelhetik a szenzort.)

Ugyanezt a mechanizmust egyébként nem csak a szenzorokkal lehet kialakítani, hanem magasabb szintű "érzékelőkkel" is: például lehet készíteni egy olyan virtuális szenzort, ami ráépül az oldalsó távolságmérőre és akkor jelez, ha már meggyőződött róla, hogy egy közeli, kiszögelléses fal mellett megy a robot. Ilyenkor a felsőbb szintű logika lehet, hogy elég, ha csak erre a virtuális szenzorra iratkozik fel. Milyen kényelmes, ha van egy függvényünk, ami akkor hívódik meg, ha egyértelműen kiszögelléses fal mellett haladunk.

### Példa:

### További példák

* RobotMon-ban van ilyen? Tipikusan a megjelenítés ilyen. Bár éppen alapjelekre is rá lehet rakni. Vagy új adat beérkezésére (vagy az inkább chain of responsibility).