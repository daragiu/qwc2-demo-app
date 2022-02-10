# QGIS Web Client 2 Documentație

## Introducere

QGIS Web Client 2 (QWC2) este un client web modular de ultimă generație pentru QGIS Server, creat cu ReactJS și OpenLayers.

Este lansat sub termenii și condițiile [licenței BSD](https://github.com/qgis/qwc2-demo-app/blob/master/LICENSE).

## Cerințe

Minimul necesar este reprezentat de un mediu pentru serverul de QGIS. Proiectele care urmează să fie oferite drept QWC2 trebuie publicate ca WMS.

În plus, este necesar și un server web care va servi aplicația QWC2.


## <a id="quick-start"></a>Inițializare

QWC2 este împărțit în două părți:

- Componentele QWC2, încarcate aici [https://github.com/qgis/qwc2/](https://github.com/qgis/qwc2/). Aici se găsesc elementele de bază comune tuturor aplicațiilor QWC2.
- Un demo al aplicației QWC2, este încărcat aici [https://github.com/qgis/qwc2-demo-app](https://github.com/qgis/qwc2-demo-app). Aici se află configurația specifică utilizatorului a componentelor de bază și poate include, de asemenea, orice alte componente personalizate.

În plus, câteva componente din QWC2 (cum ar fi generarea permalink-urilor, interogarea cotelor, editarea, etc) necesită servicii externe. Implementarea de referință ale acestor servicii se regăsește aici [https://github.com/qwc-services/](https://github.com/qwc-services/).

Pentru a lucra cu QWC2, veți avea nevoie de un mediu de dezvoltare format minim din [git](https://git-scm.com/), [node](https://nodejs.org/) și [yarn 1.x](https://classic.yarnpkg.com).

Cea mai rapidă modalitate de a începe este clonarea recursivă a aplicației demo de aici:

    $ git clone --recursive https://github.com/qgis/qwc2-demo-app.git

Următorul pas este să instalați toate dependențele necesare:

    $ cd qwc2-demo-app
    $ yarn install

Apoi, porniți o aplicație de dezvoltare locală:

    $ yarn start

Aplicația de dezvoltare va rula implicit pe `http://localhost:8081`. Mențiune: dacă vă aflați în spatele unui server proxy, va trebui să [specificați setările acelui proxy](#themes-json).

În acest moment, puteți personaliza și configura aplicația în funcție de nevoile dvs., așa cum este descris în detaliu în capitolele următoare.

Pasul final este să compilați un pachet de aplicații implementabile pentru producție:

    $ yarn run prod

Puteți implementa conținutul fișierului `prod` la root-ul serverului dvs. web.

*Mențiune*: Pe unele distribuții Linux, `yarn` se poate referi la instrumentul pachetului software `cmdtest`, nu la managerul de pachete Yarn. Acesta din urmă ar putea fi numit în schimb `yarnpkg`.


## Configurație QWC2

Schema de dezvoltare se prezintă în felul următor:

| Cale                         | Descriere
|------------------------------|--------------------------------------------------------------------------|
|`├─ static/`                  |                                                                          |
|`│  ├─ assets/`               |                                                                          |
|`│  │  ├─ css/qwc2.css`       | Documentul principal de stilizare                                        |
|`│  │  ├─ img/`               | Sigla aplicației                                                         |
|`│  │  │  └─ mapthumbs/`      | Miniaturile hărții, deobicei generate automat                            |
|`│  │  └─ templates/`         |                                                                          |
|`│  │     └─ legendprint.html`| Șablon HTML pentru imprimarea legendei                                   |
|`│  ├─ translations/`         | Fișiere de traducere                                                     |
|`│  ├─ config.json`           | Fișierul de configurare principal                                        |
|`│  └─ themes.json`           | Configurație completă a temei, generată automat din `themesConfig.json`  |
|`├─ js/`                      |                                                                          |
|`│  ├─ app.jsx`               | Punctul de intrare al aplicației ReactJS                                 |
|`│  ├─ appConfig.js`          | Configurarea modulelor de bază qwc2                                      |
|`│  ├─ Help.jsx`              | Componentă pentru redarea unui dialog de ajutor definit de utilizator    |
|`│  └─ SearchProviders.js`    | Furnizori de căutare                                                     |
|`├─ icons/`                   | Pictogramele aplicației                                                  |
|`├─ qwc2/`                    | Submodulul Git care conține componentele de bază qwc2                    |
|`├─ index.html`               | Punct de intrare                                                         |
|`├─ package.json`             | Fișierul de configurare NodeJS                                           |
|`├─ styleConfig.json`         | Configurarea unor stilizări la nivelul aplicației                        |
|`├─ themesConfig.json`        | Configurarea temelor                                                     |
|`└─ webpack.config.js`        | Configurarea pachetului web                                              |

Aceste fișiere sunt descrise în detaliu în secțiunile următoare, după ordinea importanței.

### Configurarea aplicației: `config.json` și `js/appConfig.js`

Principalul aspect de reținut este că o aplicație QWC2 constă dintr-o colecție de pluginuri, care pot fi activate și configurate individual, în special separat pentru desktop și mobil. Împreună fișierele, `config.json` și `js/appConfig.js` controlează aceste pluginuri, împreună cu o serie de alte setări globale.

Fișierul `js/appConfig.js` este fișierul care configurează build-ul și definește:
- Localizarea implicită a aplicației, încorporată în aplicație. Această localizare este utilizată dacă nicio altă localizare disponibilă nu corespunde localizării browserului.
- Ce pluginuri sunt încorporate în aplicație. Pluginurile lăsate aici vor fi complet omise la compilarea pachetului de aplicații și, prin urmare, vor reduce și dimensiunea pachetului.
- Diverse funcții tip cârlig, așa cum este documentat în exemplu [exemplu `js/appConfig.js`](https://github.com/qgis/qwc2-demo-app/blob/master/js/appConfig.js).

Fișierul `config.json` este fișierul de configurare în timpul rulării. Conține următoarele setări:

*URL-uri ale serviciilor externe*:
Unele servicii externe pot fi folosite pentru a îmbunătăți aplicația. Implementarea de referință a acestor servicii se găsește aici [https://github.com/qwc-services/](https://github.com/qwc-services/). Următoarele servicii pot fi configurate:

| Setări               | Descriere   |
|----------------------|-------------|
|`permalinkServiceUrl` | Generează și rezolvă permalink-uri compacte pentru pluginul Share. Dacă este gol, va fi folosită adresa URL completă. |
|`elevationServiceUrl` | Returnează valorile de altitudine, utilizate pentru a genera un profil de înălțime la măsurarea liniilor și pentru a afișa informații de altitudine în fereastra de informații folosind clic dreapta pe hartă. Dacă este gol, informațiile respective nu vor fi afișate în client. |
|`editServiceUrl`      | Serviciu de editare a caracteristicilor straturilor deservite de QGIS Server. Solicitat de pluginul Editing. |
|`mapInfoService`      | Returnează informații suplimentare pentru a fi afișate în fereastra de informații folosind click dreapta pe hartă. Dacă este gol, nu vor fi afișate informații suplimentare. |
|`featureReportService`| Returnează un document personalizat asociat unei caracteristici. Vezi [`themesConfig.json`](#themesConfig-json). |

*Setări globale*:

Toate setările sunt opționale, cu revenirea la valorile implicite, așa cum este documentat.

| Setări                              | Descriere   |
|-------------------------------------|-------------|
|`translationsPath`                   | Calea către fișierul `translations`. Valoare implicită: `translations`.   |
|`assetsPath`                         | Calea către fișierul `assets`. Valoare implicită: `assets`.               |
|`urlPositionFormat`                  | Cum se codifică întinderea actuală a hărții în adresa URL, fie prin `centerAndZoom` sau `extent`. Vezi [URL parameters](#url-parameters) pentru detalii. Valoare implicită: `extent`. |
|`urlPositionCrs`                     | CRS-ul folosit pentru a codifica coordonatele actuale ale extinderii hărții în adresa URL. Valoare implicită: proiecția hărții. |
|`omitUrlParameterUpdates`            | Dacă omiteți actualizarea parametrilor URL. Valoare implicită: `false`.      |
|`defaultFeatureStyle`                | Stilizarea implicit de utilizat pentru geometriile de selecție și alte caracteristici fără stil. Valoare implicită: vezi `qwc2/utils/FeatureStyles.js`. |
|`projections`                        | O listă de proiecții hărți de înregistrat, în format `{"code": "<code>", "proj": "<proj4def>", "label": "<label>"}`. În mod implicit, `EPSG:3857` și `EPSG:4326` sunt înregistrate. |
|`allowFractionalZoom`                | Dacă se permite folosirea arbitrară a scarilor pentru vizualizarea hărții. Valoare implicită: `false`.      |
|`localeAwareNumbers`                 | Daca folosiți numere locale. Valoare implicită: `false`.             |
|`wmsDpi`                             | DPI-ul transmis la cererile WMS-ului. Valoare implicită: `96`.                           |
|`wmsHidpi`                           | Dacă să facă raportul de pixeli al dispozitivului pentru solicitările WMS GetMap. Valoare implicită: `true`. |
|`externalLayerFeatureInfoFormats`    | Un dicționar de formate de informații despre caracteristici pentru straturi externe, în format `{"<url>": "<format>", ...}`. Dacă adresa URL GetFeatureInfo a unui strat conține conținutul specificat `<url>`, se utilizează formatul corespunzător. |
|`storeAllLayersInPermalink`          | Ori stocați întregul arbore de straturi în datele permalink-ului, ori numai în straturile locale. Dacă este false, straturile la distanță sunt re-interogate de la serviciile respective, dacă sunt true, acestea sunt reîncărcate static (ceea ce înseamnă că straturile restaurate pot fi vechi în comparație cu capabilitățile serviciului actual). |

*Setări globale, suprascrise pentru fiecare temă*:<a id="config-json-overridable"></a>

Următoarele opțiuni pot fi specificate la nivel global și, de asemenea, suprascrise pe temă, vezi [`themesConfig.json`](#themesConfig-json).
Toate setările sunt opționale, cu revenirea la valorile implicite, așa cum este documentat.

| Setări                               | Descriere   |
|--------------------------------------|-------------|
|`preserveExtentOnThemeSwitch`         | Dacă este posibilă extinderea actuală a hărții odată cu schimbarea temei (vezi mai jos). Valoare implicită: `false`. |
|`preserveBackgroundOnThemeSwitch`     | Dacă este posibilă păstrarea stratul de fundal curent odată cu schimbarea temei. Valoare implicită: `false`. |
|`preserveNonThemeLayersOnThemeSwitch` | Dacă este posibilă păstrarea straturile fără temă atunci când schimbați tema. Valoare implicită: `false`.  |
|`allowReorderingLayers`               | Dacă se permite reordonarea straturilor în arborele de straturi. Valoare implicită: `false`.      |
|`flattenLayerTreeGroups`              | Dacă să afișați un arbore de strat simplu, omițând grupurile. Valoare implicită: `false`.  |
|`allowLayerTreeSeparators`            | Permite utilizatorilor să adauge elemente de separare într-un arbore cu strat simplu. Valoare implicită: `false`.   |
|`preventSplittingGroupsWhenReordering`| Dacă se permite separarea grupurilor de subgrupuri și ale grupurilor implicit atunci când se reordonează elementele. Valoare implicită: `false`. |
|`allowRemovingThemeLayers`            | Dacă se permite eliminarea straturilor temă din arborele de straturi. Valoare implicită: `false`. |
|`searchThemes`                        | Dacă se permite căutarea temelor din câmpul de căutare globală. Valoare implicită: `false`. |
|`allowAddingOtherThemes`              | Dacă se permite adăugarea unei alte teme la o temă încărcată curent. Valoare implicită: `false`. |
|`disableImportingLocalLayers`         | Dacă se permite să ascundeți opțiunea de a importa straturi locale din arborele de straturi. Valoare implicită: `false`. |
|`importLayerUrlPresets`               | O listă de adrese URL predefinite din care utilizatorul poate alege atunci când importă straturi din arborele de straturi. Datele introduse trebuie să fie șiruri de caractere sau obiecte ale formatului `{"label": "<Label>", "value": "<URL>"}`. Vedeți de asemenea [Layer catalogs](#layer-catalogs). |
|`identifyTool`                        | Numele pluginului de identificare petru a fi utilizat. Este posibil să aveți mai multe instrumente de identificare și, de exemplu, pe bază de temă, selectați care să fie activ. Valoare implicită: `Identify`. |
|`globallyDisableDockableDialogs`      | Dacă trebuie să dezactivați global setarea de andocare a dialogurilor de tip pop-up. Valoare implicită: `false`. |

*Mențiuni*:

- Arborele de straturi acceptă reordonarea straturilor prin glisare și plasare dacă `allowReorderingLayers = true` *și `preventSplittingGroupsWhenReordering = true` *sau* `flattenLayerTreeGroups = true`.
- Dacă `preserveExtentOnThemeSwitch = true`, și extinderea actuală este păstrată dacă se află în noua extindere a temei și dacă proiecția actuală a hărții temei este egală cu noua proiecție a temei. Dacă `preserveExtentOnThemeSwitch = "force"`, extinderea actuală este păstrată, indiferent dacă se află în noua extindere a temei, dar proiecțiile actuale și noi ale hărții tematice trebuie să se potrivească în continuare.

*Configurare plugin-ului*:
Configurația pluginului este introdusă separat atât pentru desktop cât și pentru mobil. Consultați [exemplul `config.json`](https://github.com/qgis/qwc2-demo-app/blob/master/static/config.json) pentru o listă de opțiuni de configurare disponibile. Fiecare bloc de configurare ale plugin-urilor este scris în formatul:

    {
      "name": "<PluginName">,
      "cfg": {
        ...
      },
      "mapClickAction": <"identify"|"unset"|null>
    }

unde

* `name`: Numele pluginului.
* `cfg`: Opțional: proprietăți de configurare arbitrare, transmise direct clasei de plugin-uri relative ca elemente de React props.
* `mapClickAction`: Opțional: pentru pluginurile care sunt asociate unei sarcini de vizualizare (și de obicei legate în `menuItems` sau `toolbarItems` ale `TopBar`, vezi mai jos), determină dacă un click pe hartă va duce la apariția instrumentului de identificare, la anularea sarcinii sau dacă nu trebuie efectuată nicio acțiune anume (implicit). Mențiune: `"mapClickAction"` trebuie să fie `null` omis pentru pluginurile care gestionează acțiunile mouse-ului de pe hartă. Opțional, poate fi specificat și direct în `menuItems` sau `toolbarItems` datele introduse, vezi mai jos.

Puteți omite un plugin pentru al dezactiva în modul desktop și/sau mobil. Pentru a elimina complet un plugin din aplicația compilată, eliminațil direct din `js/appConfig.js`.

Un aspect deosebit de interesant este configurarea datelor introduse din meniul aplicației și bara de instrumente, spre exemplu datele introduse din `menuItems` și `toolbarItems` în configurarea `TopBar` . Cel mai comun format pentru conectarea unor date introduse la un plugin existent este:

    {"key": "<key>", "icon": "<icon>", "themeWhitelist": ["<themename>", ...], "mapClickAction": <"identify"|"unset"|null>, "mode": "<mode>", "requireAuth": <true|false>}

unde

* `key`: Numele pluginului de activat când se face click pe date, spre exemplu `LayerTree`. De asemenea, folosit pentru a căuta eticheta pentru datele din traduceri, folosind `appmenu.items.<key>` message identifier (vezi <a href="#translations">Gestionarea traducerilor</a>).
* `icon`: Pictograma datelor introduse, fie un nume (fără extensia `.svg`) a unei pictograme din `icons/`, sau `:/<path_to_asset>` conţinând calea către `assetsPath` a unei imagini de tip asset.
* `themeWhitelist`: Opțional, permite specificarea unui whitelist (listă cu permisiuni) de nume de teme sau titluri pentru care intrarea ar trebui să fie vizibilă.
* `mapClickAction`: Opțional, are prioritate asupra setării `mapClickAction` specificată în blocul de configurare a pluginului, dacă există. Vezi mai sus.
* `mode`: Opțional, în funcție de plugin, un mod poate fi configurat pentru a lansa pluginul direct într-un anumit mod specificat. Spre exemplu, pluginul `Measure` acceptă specificarea modului de măsurare (`Point`, `LineString`, `Polygon`).
* `requireAuth`: Opțional, the entry is only visible when user is logged-in when true (works with qwc-services).

În plus, intrările care deschid adrese URL externe pot fi definite așa:

    {"key": "<key>", "icon": "<icon>", "url": "<url>"}

unde

* `key`: Un nume de cheie arbitrar (nu este folosit de către pluginurile existente), folosit pentru a căuta eticheta pentru datele introduse din traduceri.
* `icon`: Ca mai sus.
* `url`: Adresa URL de accesat. Poate conține ca substituenți cheile enumerate în <a href="#url-parameters">parametrii URL-ului</a>, închise în `$` (ex. `$e$` pentru măsură). În plus, substituenții `$x$` și `$y$` pentru coordonatele individuale ale centrului hărții sunt de asemenea acceptate.



### Configurarea temei: proiectele QGIS și a fișierului `themesConfig.json`

O temă corespunde unui proiect QGIS, publicat ca WMS și servit de către QGIS Server.

#### Crearea și publicarea unui proiect QGIS

Primul pas este pregătirea unui proiect QGIS. Pe lângă sarcinile comune de adăugare și grupare a straturilor, următorul tabel oferă o prezentare generală a setărilor care influențează modul în care tema este afișată de QWC2:

| Cum se numește       | Unde se regăsește                         | Descriere                                        |
|----------------------|-------------------------------------------|--------------------------------------------------|
| Service capabilities | Project Properties &rarr; QGIS Server &rarr; Service capabilities | **Trebuie** să fie bifat pentru QGIS Server pentru a publica proiectul. |
| Service Metadata     | Project Properties &rarr; QGIS Server &rarr; Service capabilities | Afișat în dialogul de informații despre temă, accesibil din bara de sus a panoului Layer Tree. |
| Title, keywords      | Project Properties &rarr; QGIS Server &rarr; Service capabilities | Theme title, afișat în Theme Switcher, and keywords, util pentru filtrări. |
| Queryable layers     | Project Properties &rarr; Data sources | Marcați straturile ca identificabile de către client.       |
| FeatureInfo geometry | Project Properties &rarr; QGIS Server &rarr; WMS Capabilities &rarr; Add geometry to feature response | Returnați geometriile elementelor cu GetFeatureInfo.Permite clientului să evidențieze elementele selectate. |
| Layer Display Field  | Vector Layer Properties &rarr; Display    | Câmpul utilizat pentru identificarea rezultatelor. |
| Layer Map Tip        | Vector Layer Properties &rarr; Display    | Conținutul din Map Tip afișat când treceți cu mouse-ul peste straturi, dacă este afișat Map Tips este activ în Layer Tree. |
| Layer Metadata       | Layer Properties &rarr; QGIS Server       | Afișat în Layer Info, accesibil din Layer Tree. |
| Scale range          | Layer Properties &rarr; Rendering &rarr; Scale dependent visibility | Intervalul de scară în care este vizibil un strat, util pentru a îmbunătăți performanța de randare. |
| Initial visibility   | Layers Panel                              | Vizibilitatea inițială a straturilor și a grupurilor. |
| Rendering order      | Layer Order Panel or Layers Panel         | Ordinea de apariție a straturilor. Dacă layer re-ordering este activ în `config.json`, ordinea din Layer Order Panel este ignorată. |
| Print layouts        | Layout manager                            | Moduri de imprimate oferite de pluginul Print.  |
| Print layout labels  | Layout manager                            | Etichetele de imprimare cu un ID o să fie vizibile în pluginul Print. |

#### <a id="themesConfig-json"></a>Configurarea temei din `themesConfig.json`

Al doilea pas este să configurați temele care sunt disponibile pentru QWC2 în fișierul `themesConfig.json`, care conține o listă de teme, opțional organizate în grupuri, precum și o listă de straturi de fundal::

    {
      "themes": {
        "items": [
          { <ThemeDefinition> },
          ...
        ], "groups": [
          {
            "title": <Group title>,
            "items": [{ <ThemeDefinition> }, ...],
            "groups": [ { <Group> }, ...]
           },
           ...
        ]
      },
      "externalLayers": [
        { <ExternalLayerDefinition> },
        ...
      ],
      "themeInfoLinks": [
        { <ThemeInfoLinkDefinition> },
        ...
      ],
      "backgroundLayers": [
        { <BackgroundLayerDefinition> },
        ...
      ],
      "defaultScales": [<Scale denominators>],
      "defaultPrintScales" [<Scale denominators>],
      "defaultPrintResolutions": [<DPIs>],
      "defaultPrintGrid": [<Print grid, see below>]
    }

RConsultați [exemplul `themesConfig.json`](https://github.com/qgis/qwc2-demo-app/blob/master/themesConfig.json).

Formatul temei este următorul:
<!-- Important: Use U+00A0 non-breaking spaces ( ) in code blocks -->
| Date de intrare                              | Descriere                                                                         |
|----------------------------------------------|-----------------------------------------------------------------------------------|
| `"url": "<WMS URL>",`                        | Adresa WMS-ului dorit deservit de QGIS Server.                                    |
| `"wmsBasicAuth": "{`                         | Opțional, permite autentificarea la QGIS Server cu ajutorul themes.json. Mențiune: aceste acreditări vor fi utilizate numai de `yarn run themesConfig` și nu vor fi salvate în `themes.json`.|
| `  "username": <username>`                   | Opțional: Nume de utilizator pentru autentificare de tip http.                    |
| `  "password": <password>`                   | Opțional: Parolă pentru autentificare de tip http..                              |
| `},`                                         |                                                                                   |
| `"title": "<Custom title>",`                 | Opțional, suprascrie titlul WMS.                                                  |
| `"description": "<Description>",`            | Opțional, o descriere suplimentară de afișat sub titlul temei.                    |
| `"thumbnail": "<Filename>",`                 | Opțional, fișierele tip imagine din `assets/img/mapthumbs`.Dacă este omis, se generează automat prin WMS GetMap. |
| `"attribution": "<Attribution>",`            | Opțional, atribuții care vor fi afișate în colțul din dreapta jos al hărții.      |
| `"attributionUrl": "<URL>",`                 | Opțional, link asociat atribuirii.                                                |
| `"default": true,`                           | Dacă să folosiți această temă ca temă implicită                                     |
| `"scales": [<Scale denominators>],`          | Lista numitorilor scărilor permise de hărți. Dacă este omis, este implicit `defaultScales`. |
| `"printScales": [<Scale denominators>],`     | Lista numitorilor scalelor de imprimare permise. Dacă este omis, este implicit `defaultPrintScales`. |
| `"printResolutions": [<DPIs>],`              | Lista rezoluțiilor de imprimare disponibile. Dacă este omis, este implicit `defaultPrintResolutions`. |
| `"printGrid": [`                             | Lista intervalelor de grilă în funcție de scara grilei de utilizat la imprimare. Dacă este omis, este implicit `defaultPrintGrid`. |
| `  {"s": <Scale1>, x: <Interval1>, y: <Interval1>},` | Păstrați această listă sortată în ordine descrescătoare după numitorul de scară.           |
| `  {"s": <Scale2>, x: <Interval2>, y: <Interval2>}`  | În acest exemplu, `{x: <Interval2>, y: <Interval2>}` o să fie folosit așa `<Scale1> > Scale >= <Scale2>`.  |
| `],`                                          |                                                                                  |
| `"printLabelForSearchResult": "<ID>",`        | Opțional, un ID al unei etichete de aspect de tipărire în care va fi scris textul curent al rezultatului căutării (dacă există) la imprimare. |
| `"printLabelConfig": {`                       | Opțional, configurarea câmpurilor de introducere a textului pentru imprimarea etichetelor.        |
| `  "<LabelId>": {"rows": <n>, "maxLength": <n>},` | Înălțimea câmpului de introducere în rânduri și numărul maxim de caractere permise. |
| `},`                                          |                                                                                  |
| `"mapCrs: "<EPSG code>",`                     | Opțional, proiecția hărții, setată implicit la `EPSG:3857`.                               |
| `"extent": [<xmin>, <ymin>, <xmax>, <ymax>],` | Opțional, suprascrie extinderea temei. Din `mapCrs`.                                    |
| `"tiled": <boolean>,`                         | Opțional, utilizați tiled WMS, implicit `false`.                                    |
| `"format": "<mimetype>",`                     | Opțional, formatul de utilizat pentru WMS GetMap. Implicit `image/png`.             |
| `"externalLayers": [{`                        | Opțional, straturi externe de utilizat ca înlocuitori pentru straturile interne, vezi mai jos. |
| `  "name": "<external_layer_name>",`          | Numele stratului extern, care se potrivește cu `ExternalLayerDefinition`, vezi mai jos.     |
| `  "internalLayer": "<QGis_layer_name>"`      | Numele unui strat intern, așa cum este conținut în proiectul QGIS, de înlocuit cu stratul extern. |
| `}],`                                         |                                                                                  |
| `"themeInfoLinks": {`                         | Opțional, link-uri personalizate către resurse suplimentare, afișate ca meniu în selectorul de teme din theme switcher. |
| `  "title": "<Menu title>",`                  | Un șir arbitrar afișat drept titlu al meniului.                                  |
| `  "titleMsgId": "<Menu title msgID>",`       | O alternativă la `title`, un ID de mesaj, tradus prin fișierele de traducere.  |
| `  "entries": [<link_name>, ...]`             | Lista numelor de link-uri cu informații despre teme, vezi mai jos.                                        |
| `},`                                          |                                                                                  |
| `"backgroundLayers": [{,`                     | Opțional, lista de straturi de fundal disponibile.                                   |
| `  "name": "<Background layer name>",`        | Numele potrivirii cu `BackgroundLayerDefinition`, vezi mai jos.                         |
| `  "printLayer": "<QGis layer name>"\|[<list>],`| Opțional, numele stratului de utilizat ca strat de fundal potrivit la imprimare. Alternativ, o listă `[{"maxScale": <scale>, "name": "<QGis layer name>"}, ..., {"maxScale": null, "name": "<QGis layer name>"}]` pot fi furnizate, ordonate în ordine crescătoare de `maxScale`. Ultima intrare ar trebui să aibă `maxScale` `null`, ca strat folosit pentru toate scările rămase. Dacă este omis, fundalul nu este tipărit, atât timp cât stratul nu este de tip "wms" și `printExternalLayers` este `true` în configurarea pluginului Print. |
| `  "visibility": <boolean>`                   | Opțional, vizibilitatea inițială a stratului atunci când tema este încărcată.                 |
| `}],`                                         |                                                                                  |
| `"searchProviders": ["<ProviderId>"],`        | Opțional, lista ID-urilor furnizorilor de căutare. Un ID corespunde cheii exportului din `SearchProviders` aflat în `js/SearchProviders.js`. |
| `"minSearchScaleDenom": <number>,`                 | Opțional, scara minimă de aplicat atunci când măriți rezultatele căutării. Are prioritate față de valoarea din `config.json`. |
| `"featureReport": {`                          | Opțional, șabloane disponibile de rapoarte de caracteristici.                                   |
| `  "<LayerId>": "<TemplateID>"  `             |ID-ul substratului WMS și ID-ul șablonului asociat pentru a le transmite în `featureReportService`.|
| `},`                                          |                                                                                  |
| `"additionalMouseCrs": ["<EPSG code>"],`      | Opțional, lista de proiecții suplimentare pentru afișarea poziției mouse-ului. WGS84 și `mapCrs` sunt disponibile implicit. Trebuie de asemenea adăugate definiții suplimentare pentru proiecții în `js/appConfig.js` sau `config.json`.         |
| `"watermark": {`                              | Opțional, configurația ștampilei de adăugat la imaginile de export raster.            |
| `  "text": "<text>",`                         | Text arbitrar                                                                 |
| `  "texpadding": <number>,`                   | Opțional, spațiu între text și cadru.                             |
| `  "fontsize": <number>,`                     | Opțional, mărimea textului.                                                             |
| `  "fontfamily": "<Font family>",`            | Opțional, familia din care face parte textul.                                                           |
| `  "fontcolor": "#RRGGBB",`                   | Opțional, culoarea textului.                                                            |
| `  "backgroundcolor": "#RRGGBB",`             | Opțional, culoarea de fundal a cadruluii.                                                |
| `  "framecolor": "#RRGGBB",`                  | Opțional, culoarea cadruluii.                                                           |
| `  "framewidth": <number>,`                   | Opțional, lățimea cadruluii.                                                           |
| `},`                                          |                                                                                  |
| `"collapseLayerGroupsBelowLevel": <level>,`   | Opțional, stratul nivel al arborelui sub care să restrângeți inițial grupurile. În mod implicit, arborele este complet extins. |
| `"skipEmptyFeatureAttributes": <boolean>,`    | Opțional, dacă să omiteți atributele goale în rezultatele de identificare.Implicit `false`. |
| `"mapTips":  <boolean>\|null,`                | Opțional, setarea pentru fiecare temă dacă map-tips nu este disponibil (`null`), dezactivat implicit (`false`) sau activat implicit (`true`). |
| `"extraLegendParameters": "<&KEY=VALUE>",`    | Opțional, parametri suplimentari de interogare la care să se atașeze la WMS GetLegendGraphic.         |
| `"printLabelBlacklist":  ["<LabelId>", ...]`  | Opțional, lista de ID-uri cu etichetă pentru a nu se afișa în fereastra de imprimare. |
| `"editConfig": "<editConfig.json>"`           | Opțional, obiect sau calea către un nume de fișier care conține configurația de editare pentru temă, vezi [EditingInterface.js](#editing-interface). |
| `"config": {`                                 | Opțional, date de configurare pe temă care înlocuiesc datele introduse global în `config.json`.|
| `  "allowRemovingThemeLayers": <boolean>`     | Vezi [`config.json`](#config-json-overrideable) pentru setările care pot fi specificate aici. |
| `  ...`                                       |                                                                                  |
| `}`                                           |

**Straturi externe:**
Straturile externe pot fi folosite pentru a înlocui selectiv straturile într-un proiect QGIS, de exemplu, în cazul unui strat WMS încorporat într-un proiect QGIS, pentru a evita solicitările WMS în cascadă. Acestea sunt gestionate în mod transparent de QWC2 (sunt poziționați în arborele straturilor identic cu stratul intern pe care îl înlocuiesc), dar cererile `GetMap` și `GetFeatureInfo` sunt trimise direct către serviciul WMS specificat.

Formatul pentru definițiile straturilor externe este următorul:

| Date de intrare                                        | Descriere                                                                         |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"name": "<external_layer_name>",`                     | Numele stratului extern, așa cum este menționat în definițiile temei.             |
| `"type": "<layer_type>",`                              | Tipul stratului, „wms” sau „wmts”                                                 |
| `"url": "<wms_baseurl>",       `                       | URL-ul WMS sau URL-ul resursei WMTS pentru stratul extern.                        |

Pentru straturile WMS externe, se aplică următorii parametri suplimentari:

| Date de intrare                                        | Descriere                                                                         |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"params": {`                                          | Parametrii pentru cererea de tip GetMap.                                          |
| `  "LAYERS": "<wms_layername>,..."`,                   | Numele straturilor WMS.                                                           |
| `  "OPACITIES": "<0-255>,..."`                         | Opțional, dacă serverul WMS acceptă opacități.                                    |
| `},`                                                   |                                                                                   |
| `"featureInfoUrl": "<wms_featureinfo_baseurl>",`       | Opțional, URL-ul de bază pentru WMS GetFeatureInfo, dacă este diferit de `url`.   |
| `"legendUrl": "<wms_legendgraphic_baseurl>"   ,`       | Opțional, URL-ul de bază pentru WMS GetLegendGraphic, dacă este diferit de `url`. |
| `"queryLayers": ["<wms_featureinfo_layername>", ...],` | Opțional, lista de straturi de interogare GetFeatureInfo, dacă diferă de `params.LAYERS`. |
| `"infoFormats": ["<featureinfo_format>", ...]`         | Lista de formate de interogare GetFeatureInfo pe care le acceptă serviciul WMS.   |

Pentru straturile WMTS externe, se aplică următorii parametri suplimentari (puteți folosi script-ul `qwc2/scripts/wmts_config_generator.py` pentru a obține următoarele valori):

| Date de intrare                                        | Descriere                                                                         |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"tileMatrixSet": "<tile_matrix_set_name>",`           | Denumirea unui tile matrix set.                                                   |
| `"originX": <origin_x>,`                               | Oringinea X a unui tile matrix.                                                   |
| `"originY": <origin_y>,`                               | Oringinea Y a unui tile matrix.                                                   |
| `"projection": "EPSG:<code>",`                         | Proiecția stratului                                                               |
| `"resolutions": [<resolution>, ...],`                  | Lista rezoluțiilor WMTS.                                                          |
| `"tileSize": [<tile_width>, <tile_height>]`            | Lățimea și înălțimea tile-ului                                                    |

De asemenea, puteți seta „Data Url” pentru un strat în QGIS  (Layer Properties &rarr; QGIS Server &rarr; Data Url) la un șir de caractere din tabel

    wms:<service_url>#<layername>

(sore exemplu, `wms:http://wms.geo.admin.ch/?#ch.are.bauzonen`), iar un strat extern care indică serviciul WMS specificat va fi creat automat pentru stratul QGIS corespunzător.
Rețineți că acest lucru este implementat în prezent numai pentru straturi WMS.


**Link-uri cu informații despre teme:**
Puteți specifica link-uri de afișat într-un meniu de informații lângă titlul temei respective în theme switcher.

Formatul pentru link-urile de informații ale temei este următorul:

| Date de intrare                                        | Descriere                                                                         |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"name": "<link_name>",`                               | Numele linkului, așa cum este menționat în definițiile temei.                     |
| `"title": "<link_title>",`                             | Titlul linkului, așa cum este afișat în meniul de informații al temei în theme switcher. |
| `"url": "<link>",`                                     | O adresă URL a link-ului.                                                                       |
| `"target": "<link_target>"`                            | Ținta linkului, spre exemplu `_blank`.                                                   |


**Straturi de fundal:**
Straturile de fundal sunt menținute complet de partea clientului și nu apar în arborele de straturi.

Formatul definițiilor stratului de fundal este următorul:

| Date de intrare              | Descriere                                                                         |
|------------------------------|-----------------------------------------------------------------------------------|
| `"name": "<Name>",`          | Numele stratului de fundal, folosit în definițiile temei.                         |
| `"title": "<Title>",       ` | Titlul stratului de fundal, așa cum este afișat în background switcher.           |
| `"thumbnail": "<Filename>",` | Opțional, fișier tip imagine din `assets/img/mapthumbs`. Implicit la `default.jpg`.        |
| `"type": "<Type>",`          | Tipul stratului de fundal, spre exemplu `wms` sau `wmts`.                                  |
| `"group":  "<GroupId>",`     | Opțional, un șir de ID de grup. Straturile de fundal cu același ID de grup vor fi grupate împreună în background switcher. |
| `"minScale": <min_scale>,`   | Opțional, numitorul de scară minimă de la care se redă stratul.               |
| `"maxScale": <max_scale>,`   | Opțional, numitorul de scară maximă de la care să reda stratul.               |
| `<Layer params>`             | Parametri în funcție de tipul de strat specificat. Consultați [sample `themesConfig.json`](https://github.com/qgis/qwc2-demo-app/blob/master/themesConfig.json) pentru cateva exemple. |

*Mențiune*: Puteți utiliza scriptul python de ajutor localizat la `qwc2/scripts/wmts_config_generator.py` pentru a genera cu ușurință configurații ale stratului de fundal WMTS.

Alternativ, o definiție a stratului de fundal poate fi un grup de straturi, în format:

    {
      "name": "<Name>",
      "title": "<Title>",
      "type": "group",
      "items": [
        { <BackgroundLayerDefinition> },
        { <BackgroundLayerDefinition> },
        ...
      ]
    }

În loc să specificați o definiție completă a stratului de fundal într-un grup, puteți face referire la unul existent cu `"ref": "<bg_layer_name>"`, și anulează selectiv anumite proprietăți, cum ar fi `minScale` și `maxScale`:

    {
      ...
      "items": [
        {
          "ref": "<bg_layer_name>",
          "minScale": <min_scale>,
          "maxScale": <max_scale>
        },
        ...
      ]
    }

#### <a id="themes-json"></a>Generând `themes.json`

Pasul final este generarea `themes.json`, fișierul care este în cele din urmă citit de către QWC2. Acest fișier combină datele introduse `themesConfig.json` cu informațiile din capabilitățile serviciului WMS și este generat automat la pornirea aplicației de dezvoltare locală prin `yarn start`. Alternativ, poate fi generat manual prin:
    yarn run themesconfig

sau, dacă se lucrează într-un mediu fără nod, folosind comanda echivalentă

    python3 qwc2/scripts/themesConfig.py


Dacă doriți să gestionați mai multe fișiere `themesConfig.json`, puteți specifica care ar trebui procesat de scriptul de generare a temei prin intermediul `QWC2_THEMES_CONFIG`.

Mențiune: dacă sunteți în spatele unui server proxy și fișierul `themesConfig.json` se referă la resurse din afara rețelei locale, va trebui să specificați setările proxy în care să utilizați `themesConfig.json` prin adăugarea unui bloc de nivel superior al formularului:
    {
      "proxy": {
          "host": "<host>",
          "port": <port>,
          "auth": {
              "username": "<username>",
              "password": "<password>"
        }
      },
      "themes": {...},
      ...
    }

### Implementarea furnizorilor de căutare în `js/SearchProviders.js`

Furnizorii de căutare sunt de obicei specifici aplicației și, prin urmare, trebuie implementați în fișierul `js/SearchProviders.js`. [sample `js/SearchProviders.js`](https://github.com/qgis/qwc2-demo-app/blob/master/js/SearchProviders.js) documentează modul de implementare a furnizorilor de căutare și conține câteva exemple.

O caracteristică avansată este posibilitatea de a defini furnizorii de căutare parametrizați. Acest lucru permite, de exemplu, să implementați doar o interfață de furnizor de căutare generică în `js/SearchProviders.js`și mutați detaliile de implementare în serviciu. În acest scop, puteți adăuga, de asemenea, la `searchProviders` lista din `themesConfig.json` datele de intrare ale formularului:

    {key: <Key>, label: <Label>, ...}

Such entries are passed to the method `searchProviderFactory` in `js/SearchProviders.js`, which you can tweak to dynamically create a search provider definition based on the parameters specified in the entry. Refer to the [sample  `js/SearchProviders.js`](https://github.com/qgis/qwc2-demo-app/blob/master/js/SearchProviders.js) for an example.

### <a id="editing-interface"></a>Implementarea interfeței de editare în `js/EditingInterface.js`

Pluginul de editare QWC2 permite adăugarea, eliminarea și editarea caracteristicilor de pe hartă. Pentru ca acest lucru să funcționeze, trebuie să efectuați următorii pași:

- Pluginul de editare trebuie să fie activat în `config.json` și `appConfig.js`.
- Fișierul `editServiceUrl` trebuie să fie specificat în `config.json`.
- Serviciul de pe partea serverului trebuie configurat. Fișierul [interfață de editare implicită](https://github.com/qgis/qwc2/tree/master/utils/EditingInterface.js) este omologul pentru [Serviciu de date QWC](https://github.com/qwc-services/qwc-data-service). De asemenea, puteți utiliza un serviciu de editare personalizat, caz în care trebuie să implementați un serviciu personalizat EditingInterface și transmis către pluginul Editing în `appConfig.js`.
- Pentru fiecare temă pentru care este permisă editarea, o potrivire `editConfig.json` trebuie implementat și specificat în datele de intrare corespunzătoare din `themesConfig.json`.

Formatul `editConfig.json` este după cum urmează:

| Date de intrare                     | Descriere                                                     |
|-------------------------------------|---------------------------------------------------------------|
| `{`                                 |                                                               |
| `  <LayerId>: {`                    | Un ID de strat WMS. Ar trebui să fie un nume de strat WMS temă, pentru a vă asigura că WMS-ul este reîmprospătat corect. |
| `    "layerName": "<LayerName>",`   | Numele stratului de afișat în fereastra combinată de selecție. |
| `    "geomType": "<GeomType>",`     | Tipul de geometrie, fie `Point`, `LineString` sau `Polygon`. |
| `    "displayField":  "<FieldId">",`| ID-ul câmpului de utilizat în meniul de selecție a caracteristicilor.     |
| `    "fields": [{`                  | O listă de definiții de câmp, pentru fiecare atribut afișat.      |
| `      "id": "<FieldID>",`          | ID-ul câmpului.                                                |
| `      "name": "<FieldName>",`      | Numele câmpului, așa cum este afișat în formularul de editare.             |
| `      "type": "<FieldType>",`      | Un tip de câmp. Fie `bool`, `list` sau un simplu [elemente de intrare tip HTML](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input). |
| `      "constraints": {`            | Constrângeri pentru câmpul de intrare.                             |
| `        "values": [<Entries>],`    | Numai dacă `type` este `list`: o matrice de șiruri de caractere arbitrare.     |
| `        ...`                       | Pentru tipurile obișnuite de introducere HTML, numele ReactJS API al oricărui aplicabil [Constrângere de intrare de tip HTML](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input), i.e. `maxLength` or `readOnly`. |
| `      }`                           |                                                               |
| `    }],`                           |                                                               |
| `    "form": "<PathToUiFile>",`     | Ca alternativă la `fields`, un fișier QtDesigner UI. Vezi mai jos.  |
| `  }`                               |                                                               |
| `}`                                 |                                                               |

* Dacă specificați `fields`, un formular simplu este generat automat pe baza definițiilor câmpurilor.
* Dacă specificați `form`, puteți specifica adresa URL a unui formular Qt Designer UI (utilizați `:/<path>` pentru a numi calea catre fișierul `assets`). Cele mai multe elemente de intrare de bază furnizate de QtDesigner sunt acceptate, vezi [acest exemplu de formular](https://github.com/qgis/qwc2-demo-app/blob/master/assets/forms/form.ui). Numele widgetului trebuie să fie setat la fel ca numele atributelor.

Vezi [exemplul `editConfig.json`](https://github.com/qgis/qwc2-demo-app/blob/master/test2056_edit.json). Vezi, de asemenea, [Serviciu de date QWC README](https://github.com/qwc-services/qwc-data-service/blob/master/README.md).

### <a id="translations"></a>Gestionarea traducerilor

Traducerile sunt gestionate pe două niveluri:

- La nivelul componentelor QWC2, în `qwc2/translations`.
- La nivel de aplicație, în `translations`.

Un script se va ocupa de îmbinarea traducerilor componente în traducerile aplicației. În acest fel, la actualizarea submodulului QWC2, se obțin automat noi traduceri. Acest script este invocat automat `yarn start`, dar poate fi, de asemenea, chemat manual folosind:

    yarn run tsupdate

Traducerile sunt stocate în folder-ul `translations` ca fișiere JSON obișnuite cu text simplu, denumit `<locale>.json` și poate fi editat cu orice editor de text.

Fisierele `tsconfig.json` stocate în interiorul folder-ului `translations` care conține lista de limbi pentru care ar trebui să fie generate traduceri și o listă de ID-uri de mesaje de inclus în traducere. Script-ul `tsupdate` va verifica automat ID-urile mesajelor (căutând șiruri statice transmise către `LocaleUtils.tr` și `LocaleUtils.trmsg`), stocate în `tsconfig.json` și creează automat sau actualizează fișierele de traducere.

În unele cazuri `tsconfig.json` nu va prelua un ID de mesaj (de exemplu, dacă este calculat în timpul execuției). În aceste cazuri, ID-urile mesajelor pot fi adăugate manual în `extra_strings` secțiunea `tsconfig.json`.

De asemenea, se poate suprascrierea o traducere venită din componentele QWC2 la nivel de aplicație. Pentru a preveni `tsupdate` de la revenirea continuă a traducerii suprascrise, ID-urile mesajelor respective poate fi adăugat la secțiunea `overrides` în fișierul `tsconfig.json`.

Fluxul de lucru tipic pentru gestionarea traducerilor aplicațiilor este:

- Declarați toate locațiile care ar trebui să fie luate în considerare, în`js/appConfig.js`.
- Asigurați-vă că aceste locații sunt listate în `translations/tsconfig.json`.
- Scrieți șirurile de caractere traduse în fișierul `<locale>.json`.
- Verificați traducerea, specificând `locale=<locale>` în adresa URL a aplicației QWC2.

Dacă lipsesc traduceri ale componentelor QWC2 pentru o limbă dorită, vă rugăm să creați sau actualizați traducerile fișierelor respective din `qwc2/translations` și contribuiți cu ele trimițând un pull request la [upstream qwc2 repository](https://github.com/qgis/qwc2).

### <a id="appearance-customization"></a>Personalizarea aspectului QWC2

Următoarele opțiuni sunt disponibile pentru personalizarea aspectului aplicației QWC2, păstrând în același timp compatibilitatea cu componentele de bază QWC2:

- Modificarea siglei în `assets/img/`.
- Modificarea pictogramelor aplicației în `icons`.
- Modificarea culorilor în `styleConfig.js`.
- Adăugarea declarațiilor de stil la fișierul de stilizare CSS principal `assets/css/qwc2.css`. Totuși, acest lucru este potențial fragil și ar trebui făcut doar ca ultimă variantă.
- Schimbarea titlului paginii browser în `index.html`, și eventual adăugarea unui favicon.
- Modificarea șablonului de imprimare a legendei în `assets/templates/legendprint.html`. Singura cerință pentru acest șablon este că trebuie să conțină elementul `<div id="legendcontainer"></div>`.
- Modificarea componentei de ajutor personalizat `js/Help.jsx`.

*Mențiune*: Pictogramele comune ale aplicațiilor sunt situate în `qwc2/icons`. Acestea pot fi suprascrise prin crearea unei pictograme cu același nume de fișier în aplicația specifică folder-ului `icons`.
*Mențiune*: Pictogramele din folder-ul `icons` sunt compilate într-un font pictogramă. În prezent, pictogramele trebuie să aibă conținut negru pe fundal transparent, iar toate desenele (inclusiv textele) trebuie convertite în căi pentru ca pictogramele să fie redate corect.


## Configurare pe partea serverului
### <a id="cross-origin-requests"></a>Solicitări încrucișate
Toate browserele moderne vor bloca o pagină să nu solicite resurse din altă parte (cu excepția imaginilor, fișierelor de stilizare, scripturilor, cadrelor iframe și videoclipurilor), cu excepția cazului în care răspunsul de la originea la distanță conține o potrivire a headeru-lui `Access-Control-Allow-Origin`. O origine este definită ca `<scheme>://<hostname>:<port>`.

Pentru fiecare serviciu cu care interacționează QWC2, în special cu serverul QGIS, trebuie să vă asigurați că această interacțiune nu este blocată de browser. Există următoarele opțiuni:

- Asigurați-vă că serviciul rulează pe aceeași origine ca și serverul web care deservește aplicația QWC2.
- Asigurați-vă că serviciul trimite un header `Access-Control-Allow-Origin` cu originea care se potrivește cu fiecare răspuns.
- În scopuri de dezvoltare, utilizați un plugin de browser care adaugă header-ele CORS, spre exemplu. [CORS peste tot](https://addons.mozilla.org/en-US/firefox/addon/cors-everywhere/).

### Nume de fișiere de imprimare, raster și export DXF
Răspunsul serverului QGIS pentru cererile de imprimare, raster și export DXF nu conține în mod implicit nici un header`Content-Disposition`, ceea ce înseamnă că browserele vor încerca să ghicească un nume de fișier, care este de obicei ultima parte a adresei URL, fără nicio extensie.

Pentru a vă asigura că browserele utilizează un nume de fișier adecvat, configurați serverul web care rulează QGIS Server pentru a adăuga un nume adecvat header-ului `Content-Disposition` pentru a răspunde. În cazul Apache, regula pentru ieșirea de imprimare ar putea arăta în felul următor:

    SetEnvIf Request_URI "^/wms.*/(.+)$" project_name=$1
    Header always setifempty Content-Disposition "attachment; filename=%{project_name}e.pdf" "expr=%{CONTENT_TYPE} = 'application/pdf'"

Această regulă va folosi ultima parte a adresei URL ca nume de bază și va adăuga extensia `.pdf`, și se va asigura, de asemenea, că tipul de conținut este setat la `application/pdf`. Rețineți că acest exemplu folosește `setenvif` și `headers` module specifice apache.

## <a id="url-parameters"></a>Parametri URL
Următorii parametri pot apărea în adresa URL a aplicației QWC2:

- `t`: Tema activă
- `l`: Straturile din hartă, vezi mai jos
- `bl`: Stratul de fundal vizibil
- `st`: Textul de căutare
- `sp`: Furnizor de căutare când se utilizează parametrul `st`
- `e`: Întinderea vizibilă
- `c`: Centrul întinderii vizibile
- `s`: Scara actuală
- `crs`: CRS-ul coordonatelor de întindere/ centru
- `hc`: Daca `c` este specificat și `hc` este `true` sau `1`, un marcator este setat cu `c` la pornirea aplicației. Mențiune: necesită plugin-ul `StartupMarkerPlugin` să fie activat.

Parametrul `l` listează toate straturile din hartă (sublinieri și de fundal) ca o listă CSV de date de intrare ale formularului:

    <layername>[<transparency>]!

unde
- `layername` este numele WMS al unui strat sau al unui grup de temă sau al unui șir al formatului

      <wms|wfs>:<service_url>#<layername>
   pentru straturi externe, de ex. `wms:https://wms.geo.admin.ch/?#ch.are.bauzonen`.
- `<transparency>` indică transparența stratului, între 0 și 100. Daca partea cu `[<transparency>]` este omisă, stratul devine opac în intregime.
- `!` indică faptul că stratul este invizibil. Dacă este omis, stratul este vizibil. Dacă stratul este vizibil într-un grup părinte invizibil, `~` este utilizat.

*Mențiune*: Dacă numele grupului este specificat în loc de numele stratului, QWC2 va rezolva automat acest lucru pentru toate numele straturilor conținute în acel grup, și va aplica setările de transparență și vizibilitate așa cum sunt specificate pentru grup.

Parametrul `urlPositionFormat` din `config.json` determină dacă în URL apare întinderea sau centrul și scara.

Parametrul `urlPositionCrs` din `config.json` determină proiecția de utilizat pentru măsura respectiv coordonatele centrului în URL. În mod implicit, este utilizată proiecția hărții temei curente. Dacă `urlPositionCrs` este egală cu proiecția hărții, parametrul `crs` este omis în adresa URL.

Dacă textul de căutare a trecut prin `st` rezultă într-un rezultat unic, vizualizatorul zoom-ează automat la acest rezultat la pornire. Dacă rezultatul căutării nu oferă o fereastră de delimitare, atunci `minScaleDenom` definit în `searchOptions` din `TopBar` configurat în `config.json` este utilizat.

## Starea inițială
În mod implicit, vizualizatorul se deschide mărind dimensiunea temei respective, așa cum este definit în `themes.json` (și cu posibilitatea de suprascriere în `themesConfig.json`).

Alternativ, există următoarele trei opțiuni pentru a influența starea inițială:

- Setați parametrii URL `c`, `s` sau `e` , așa cum este documentat în [parametri URL](#url-parameters).
- Transmite un text de căutare care are ca rezultat un rezultat unic(spre exemplu o coordonată) ca parametru URL, așa cum este documentat în [parametri URL](#url-parameters).
- Setați `startupMode` din `LocateSupport` opțiune aflată în `Map` configurând `config.json`. Valorile posibile sunt `DISABLED`, `ENABLED` sau `FOLLOWING`. Dacă un text de căutare este transmis prin parametrul URL`st` sau  `hc=1` este specificat în URL, `startupMode` este ignorat.

## <a id="layer-catalogs"></a>Cataloage de straturi
Funcționalitatea de import a stratului din arborele de straturi acceptă și încărcarea unui document de catalog de straturi dintr-o adresă URL. Sunt acceptate două formate de catalog:

* QGIS WMS/WFS conexiuni XML: acest fișier este creat prin exportarea conexiunilor WMS sau WFS configurate din fereastra QGIS Manager Data Source.
* JSON: un document cu conținut

      {
        "catalog": [
          {
            "title": "<Titel>",
            "resource": "<wms|wfs>:<service_url>#<layername>"
          },
          ...
        ]
      }

  unde `resource` este în același format ca identificatorii de strat stabilit într-un URL QWC2, vezi [parametri URL](#url-parameters).

Rețineți că serverul care deservește documentele de catalog trebuie să se asigure că setează head-urile CORS în mod corespunzător, dacă acestea sunt trimise dintr-o altă origine decât QWC2.

## Link-uri în identificarea rezultatelor

Link-urile din atributele răspunsurilor GetFeatureInfo sunt analizate și afișate ca link-uri pe care se poate face click în fereastra de identificare a rezultatelor. De asemenea, puteți avea linkuri deschise într-un dialog inline folosind următorul format:

    <a href="<url>" target=":iframedialog:<name>:<option1=value1>:<option2=value2>:...">Text</a>

Unde:

* `name` Denumierea ferestrei de dialog. Toate linkurile cu același nume se vor deschide în aceași fereastră de dialog (respectiv, reutilizați o fereastră de dialog deja vizibilă). Rețineți că titlul ferestrei va fi ID-ul mesajului de traducere `windows.<name>`, care ar trebui adăugat `extra_strings` în `tsconfig.json` și tradus în fișierele de limbă respective.
* Opțiunile acceptate includ:
  - `width=<integer>`: Lățimea inițială a ferestrei
  - `height=<integer>`: Înălțimea inițială a ferestrei
  - `print=<boolean>`: Dacă se afișează o pictogramă de tipărire în bara de titlu a ferestrei
  - `dockable=<boolean>`: Dacă fereastra este andoacabilă
  - `docked=<boolean>`: Dacă fereastra este inițial andocată


## Menținerea aplicației QWC2 la zi
După cum se menționează în capitolul de [inițializare](#quick-start), QWC2 este împărțit într-un depozit comun de componente și un depozit specific aplicației. Scopul acestei abordări este de a separa în mod curat configurația specifică utilizatorului și componentele care componente comune care servesc drept bază pentru toate aplicațiile QWC2, și pentru a face cât mai ușor posibil rebazarea aplicației pe cele mai recente componente QWC2 comune.

Fluxul de lucru recomandat este păstrarea folder-ului `qwc2` un submodul de referință pentru [upstream qwc2 repository](https://github.com/qgis/qwc2). Pentru a-l actualiza, verificați cel mai recent master (sau un anumit commit dacă doriți):

    cd qwc2
    git checkout master
    git pull
    cd ..
    yarn install

Este o practică bună să utilizezi `yarn install` după ce descărcați submodulul pentru a vă asigura că sunt instalate toate dependențele corecte.

[Mențiuni de upgrade](https://github.com/qgis/qwc2-demo-app/blob/master/UpgradeNotes.md) care documentează modificările majore și, în special, toate modificările incompatibile dintre versiuni care necesită modificări ale codului și/sau configurației specifice aplicației.

## API pentru aplicații externe
Pluginul API leagă multe acțiuni ale aplicației la `window.qwc2` și le face accesibile pentru aplicații externe. În prezent, sunt disponibile următoarele metode:

- Toate acțiunile de [afișăre](https://github.com/qgis/qwc2/blob/master/actions/display.js)
- Toate acțiunile [straturilor](https://github.com/qgis/qwc2/blob/master/actions/layers.js)
- Toate acțiunile de [localizare](https://github.com/qgis/qwc2/blob/master/actions/locate.js)
- Toate acțiunile [hărții](https://github.com/qgis/qwc2/blob/master/actions/map.js)
- Toate acțiunile legate de [sarcini](https://github.com/qgis/qwc2/blob/master/actions/task.js)
- Toate acțiunile legate de [teme](https://github.com/qgis/qwc2/blob/master/actions/theme.js)
- Toate acțiunile legate de [ferestre](https://github.com/qgis/qwc2/blob/master/actions/windows.js)
- Apelurări de tip API suplimentare:
  - `window.qwc2.addExternalLayer(resource, beforeLayerName=null)`
  - `window.qwc2.drawScratch(geomType, message, drawMultiple, callback, style = null)`

Vedeți documentele din [API.js](https://github.com/qgis/qwc2/blob/master/plugins/API.jsx) precum și funcțiile de acțiuni notate mai sus pentru mai multe informații.

Vedeți [api_examples.js](https://github.com/qgis/qwc2-demo-app/blob/master/api_examples.js) pentru câteva exemple concrete.


## Dezvoltare
QWC2 este scris în JavaScript folosind în special bibliotecile ReactJS, Redux și OpenLayers. Următoarele link-uri indică câteva resurse utile pentru a învăța elementele de bază:

**ECMAScript 2015**
- https://babeljs.io/docs/learn-es2015/

**ReactJS**

- https://facebook.github.io/react/docs/thinking-in-react.html
- https://medium.com/@diamondgfx/learning-react-with-create-react-app-part-1-a12e1833fdc
- https://medium.com/@diamondgfx/learning-react-with-create-react-app-part-2-3ad99f38b48d
- https://medium.com/@diamondgfx/learning-react-with-create-react-app-part-3-322447d14192

**Redux**

- http://redux.js.org/
- https://egghead.io/courses/getting-started-with-redux
- https://egghead.io/courses/building-react-applications-with-idiomatic-redux

În timpul dezvoltării, este util să folosiți `debug=true` la parametrii de interogare URL, care va permite înregistrarea tuturor modificărilor stării aplicației în consola browserului.
