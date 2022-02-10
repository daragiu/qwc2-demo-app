# QGIS Web Client 2 Documentație

## Introducere

QGIS Web Client 2 (QWC2) este un client web modular de ultimă generație pentru QGIS Server, creat cu ReactJS și OpenLayers.

Este lansat sub termenii și condițiile [licenței BSD](https://github.com/qgis/qwc2-demo-app/blob/master/LICENSE).

## Cerințe

Minimul necesar este reprezentat de un mediu pentru serverul de QGIS. Proiectele care urmează să fie oferite drept QWC2 trebuie publicate ca WMS.

În plus, este necesar și un server web care va servi aplicația QWC2.


## <a name="quick-start"></a>Inițializare

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
- Diverse funcții tip cârlig, așa cum este documentat în exmeplu [exemplu `js/appConfig.js`](https://github.com/qgis/qwc2-demo-app/blob/master/js/appConfig.js).

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

*Setări globale, suprascrise pentru fiecare temă*:<a name="config-json-overridable"></a>

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
* `mapClickAction`: Opțional: pentru pluginurile care sunt asociate unei sarcini de vizualizare (și de obicei legate în `menuItems` sau `toolbarItems` ale `TopBar`, vezi mai jos), determină dacă un clic pe hartă va duce la apariția instrumentului de identificare, la anularea sarcinii sau dacă nu trebuie efectuată nicio acțiune anume (implicit). Mențiune: `"mapClickAction"` trebuie să fie `null` omis pentru pluginurile care gestionează acțiunile mouse-ului de pe hartă. Opțional, poate fi specificat și direct în `menuItems` sau `toolbarItems` datele introduse, vezi mai jos.

Puteți omite un plugin pentru al dezactiva în modul desktop și/sau mobil. Pentru a elimina complet un plugin din aplicația compilată, eliminațil direct din `js/appConfig.js`.

Un aspect deosebit de interesant este configurarea datelor introduse din meniul aplicației și bara de instrumente, spre exemplu datele introduse din `menuItems` și `toolbarItems` în configurarea `TopBar` . Cel mai comun format pentru conectarea unor date introduse la un plugin existent este:

    {"key": "<key>", "icon": "<icon>", "themeWhitelist": ["<themename>", ...], "mapClickAction": <"identify"|"unset"|null>, "mode": "<mode>", "requireAuth": <true|false>}

unde

* `key`: Numele pluginului de activat când se face clic pe date, spre exemplu `LayerTree`. De asemenea, folosit pentru a căuta eticheta pentru datele din traduceri, folosind `appmenu.items.<key>` message identifier (vezi <a href="#translations">Gestionarea traducerilor</a>).
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
* `url`: Adresa URL de accesat. Poate conține ca substituenți cheile enumerate în <a href="#url-parameters">URL parameters</a>, închise în `$` (ex. `$e$` pentru măsură). În plus, substituenții `$x$` și `$y$` pentru coordonatele individuale ale centrului hărții sunt de asemenea acceptate.



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

#### <a name="themesConfig-json"></a>Configurarea temei din `themesConfig.json`

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
| Date introduse                               | Descriere                                                                         |
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
| `"themeInfoLinks": {`                         | Opțional, linkuri personalizate către resurse suplimentare, afișate ca meniu în selectorul de teme din theme switcher. |
| `  "title": "<Menu title>",`                  | Un șir arbitrar afișat drept titlu al meniului.                                  |
| `  "titleMsgId": "<Menu title msgID>",`       | O alternativă la `title`, un ID de mesaj, tradus prin fișierele de traducere.  |
| `  "entries": [<link_name>, ...]`             | Lista numelor de linkuri cu informații despre teme, vezi mai jos.                                        |
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
External layers can be used to selectively replace layers in a QGIS project, for instance in the case of a WMS layer embedded in a QGIS project, to avoid cascading WMS requests. They are handled transparently by QWC2 (they are positioned in the layer tree identically to the internal layer they replace), but the `GetMap` and `GetFeatureInfo` requests are sent directly to the specified WMS Service.

The format for external layer definitions is as follows:

| Entry                                                  | Description                                                                       |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"name": "<external_layer_name>",`                     | The name of the external layer, as referenced in the theme definitions.           |
| `"type": "<layer_type>",`                              | Layer type, "wms" or "wmts"                                                       |
| `"url": "<wms_baseurl>",       `                       | The WMS URL or WMTS resource URL for the external layer.                          |

For external WMS layers, the following additional parameters apply:

| Entry                                                  | Description                                                                       |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"params": {`                                          | Parameters for the GetMap request.                                                |
| `  "LAYERS": "<wms_layername>,..."`,                   | WMS layer names.                                                                  |
| `  "OPACITIES": "<0-255>,..."`                         | Optional, if WMS server supports opacities.                                       |
| `},`                                                   |                                                                                   |
| `"featureInfoUrl": "<wms_featureinfo_baseurl>",`       | Optional, base URL for WMS GetFeatureInfo, if different from `url`.               |
| `"legendUrl": "<wms_legendgraphic_baseurl>"   ,`       | Optional, base URL for WMS GetLegendGraphic, if different from `url`.             |
| `"queryLayers": ["<wms_featureinfo_layername>", ...],` | Optional, list of GetFeatureInfo query layers, if different from `params.LAYERS`. |
| `"infoFormats": ["<featureinfo_format>", ...]`         | List of GetFeatureInfo query formats which the WMS service supports.              |

For external WMTS layers, the following additional parameters apply (you can use the `qwc2/scripts/wmts_config_generator.py` script to obtain these values):

| Entry                                                  | Description                                                                       |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"tileMatrixSet": "<tile_matrix_set_name>",`           | The name of the tile matrix set.                                                  |
| `"originX": <origin_x>,`                               | The X origin of the tile matrix.                                                  |
| `"originY": <origin_y>,`                               | The Y origin of the tile matrix.                                                  |
| `"projection": "EPSG:<code>",`                         | The layer projection.                                                             |
| `"resolutions": [<resolution>, ...],`                  | The list of WMTS resolutions.                                                     |
| `"tileSize": [<tile_width>, <tile_height>]`            | The tile width and height.                                                        |

You can also set the "Data Url" for a layer in QGIS (Layer Properties &rarr; QGIS Server &rarr; Data Url) to a string of the form

    wms:<service_url>#<layername>

(for instance, `wms:http://wms.geo.admin.ch/?#ch.are.bauzonen`), and an external layer pointing to the specified WMS service will automatically be created for the corresponding QGIS layer.
Note that this is currently only implemented for WMS layers.


**Theme info links:**
You can specify links to display in an info-menu next to the respective theme title in the theme switcher entries.

The format for the theme info links is as follows:

| Entry                                                  | Description                                                                       |
|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| `"name": "<link_name>",`                               | The name of the link, as referenced in the theme definitions.                     |
| `"title": "<link_title>",`                             | The title for the link, as displayed in the info menu of the theme entry in the theme switcher. |
| `"url": "<link>",`                                     | A link URL.                                                                       |
| `"target": "<link_target>"`                            | The link target, i.e. `_blank`.                                                   |


**Background layers:**
Background layers are handled completely client-side and do not appear in the layer tree.

The format of the background layer definitions is as follows:

| Entry                        | Description                                                                       |
|------------------------------|-----------------------------------------------------------------------------------|
| `"name": "<Name>",`          | The name of the background layer, used in the theme definitions.                  |
| `"title": "<Title>",       ` | The title of the background layer, as displayed in the background switcher.       |
| `"thumbnail": "<Filename>",` | Optional, image file in `assets/img/mapthumbs`. Defaults to `default.jpg`.        |
| `"type": "<Type>",`          | The background layer type, i.e. `wms` or `wmts`.                                  |
| `"group":  "<GroupId>",`     | Optional, a group ID string. Background layers with the same group ID will be grouped together in the background switcher. |
| `"minScale": <min_scale>,`   | Optional, minimum scale denominator from which to render the layer.               |
| `"maxScale": <max_scale>,`   | Optional, maximum scale denominator from which to render the layer.               |
| `<Layer params>`             | Parameters according to the specified layer type. Refer to the [sample `themesConfig.json`](https://github.com/qgis/qwc2-demo-app/blob/master/themesConfig.json) for some examples. |

*Note*: You can use the helper python script located at `qwc2/scripts/wmts_config_generator.py` to easily generate WMTS background layer configurations.

Alternatively, a background layer definition can be a group of layers, in the format

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

Instead of specifiying a full background layer definition in a group, you can also reference an existing one with `"ref": "<bg_layer_name>"`, and selectively override certain properties, such as `minScale` and `maxScale`:

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

#### <a name="themes-json"></a>Generating `themes.json`

The final step is to generate `themes.json`, the file which is ultimately read by QWC2. This file combines the input from `themesConfig.json` with the information from the WMS service capabilities and is automatically generated when starting the local development application via `yarn start`. Alternatively, it can be manually generated via

    yarn run themesconfig

or, if working in an environment without node, using the equivalent command

    python3 qwc2/scripts/themesConfig.py


If you want to manage multiple `themesConfig.json` files, you can specify which while should be processed by the theme generation script via the `QWC2_THEMES_CONFIG` environment variable.

Note: if you are behind a proxy server and your `themesConfig.json` refers to resources outside the local network, you'll need to specify the proxy settings to use in `themesConfig.json` by adding a toplevel block of the form

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

### Implementing search providers in `js/SearchProviders.js`

Search providers are typically application specific, and hence need to be implemented in the application specific `js/SearchProviders.js` file. The [sample `js/SearchProviders.js`](https://github.com/qgis/qwc2-demo-app/blob/master/js/SearchProviders.js) documents how to implement search providers and contains some examples.

An advanced feature is the possibility to define parametrized search providers. This allows for instance to implement just a generic search provider interface in `js/SearchProviders.js` and move the implementation details to the service. To this end, you can also add to the `searchProviders` list in `themesConfig.json` entries of the form

    {key: <Key>, label: <Label>, ...}

Such entries are passed to the method `searchProviderFactory` in `js/SearchProviders.js`, which you can tweak to dynamically create a search provider definition based on the parameters specified in the entry. Refer to the [sample  `js/SearchProviders.js`](https://github.com/qgis/qwc2-demo-app/blob/master/js/SearchProviders.js) for an example.

### <a name="editing-interface"></a>Implementing the editing interface in `js/EditingInterface.js`

The QWC2 Editing plugin allows to add, remove and edit features from the map. For this to work, the following steps need to be performed:

- The Editing plugin needs to be enabled in `config.json` and `appConfig.js`.
- The `editServiceUrl` needs to be specified in `config.json`.
- The server-side service needs to be set up. The [default editing interface](https://github.com/qgis/qwc2/tree/master/utils/EditingInterface.js) is the conterpart for the [QWC data service](https://github.com/qwc-services/qwc-data-service). You can also use a custom editing service, in which case you need to implement a custom EditingInterface and pass it to the Editing plugin in `appConfig.js`.
- For every theme for which editing is allowed, a matching `editConfig.json` needs to be implemented and specified in the corresponding entry in `themesConfig.json`.

The format of the `editConfig.json` is as follows:

| Entry                               | Description                                                   |
|-------------------------------------|---------------------------------------------------------------|
| `{`                                 |                                                               |
| `  <LayerId>: {`                    | A WMS layer ID. Should be a theme WMS layer name, to ensure the WMS is correctly refreshed. |
| `    "layerName": "<LayerName>",`   | The layer name to show in the selection combo box. |
| `    "geomType": "<GeomType>",`     | The geometry type, either `Point`, `LineString` or `Polygon`. |
| `    "displayField":  "<FieldId">",`| The ID of the field to use in the feature selection menu.     |
| `    "fields": [{`                  | A list of field definitions, for each exposed attribute.      |
| `      "id": "<FieldID>",`          | The field ID.                                                 |
| `      "name": "<FieldName>",`      | The field name, as displayed in the editing form.             |
| `      "type": "<FieldType>",`      | A field type. Either `bool`, `list` or a regular [HTML input element type](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input). |
| `      "constraints": {`            | Constraints for the input field.                              |
| `        "values": [<Entries>],`    | Only if `type` is `list`: an array of arbitrary strings.      |
| `        ...`                       | For regular HTML input types, the ReactJS API name of any applicable [HTML input constraint](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input), i.e. `maxLength` or `readOnly`. |
| `      }`                           |                                                               |
| `    }],`                           |                                                               |
| `    "form": "<PathToUiFile>",`     | As alternative to `fields`, a QtDesigner UI file. See below.  |
| `  }`                               |                                                               |
| `}`                                 |                                                               |

* If you specify `fields`, a simple form is autogenerated based on the field definitions.
* If you specify `form`, you can specify the URL to a Qt Designer UI form (use `:/<path>` to specify a path below the `assets` folder). Most basic input elements provided by QtDesigner are supported, see [this sample form](https://github.com/qgis/qwc2-demo-app/blob/master/assets/forms/form.ui). The widget names must be set equal to the attribute names.

See the [sample `editConfig.json`](https://github.com/qgis/qwc2-demo-app/blob/master/test2056_edit.json) for a full example. See also the [QWC data service README](https://github.com/qwc-services/qwc-data-service/blob/master/README.md).

### <a name="translations"></a>Managing translations

The translations are managed on two levels:

- At QWC2 components level, in `qwc2/translations`.
- At application level, in `translations`.

A script will take care of merging the component translations into the application translations. This way, when updating the QWC2 submodule, new translations are automatically obtained. This script is automatically invoked on `yarn start`, but can also be manually invoked using

    yarn run tsupdate

Translations are stored inside the respective `translations` folder as regular plain-text JSON files, named `<locale>.json` and can be freely edited with any text editor.

The `tsconfig.json` files stored inside the respective `translations` folder contains the list of languages for which translations should be generated and a list of message IDs to include in the translation. The `tsupdate` script will automatically scan for message IDs (looking for static strings passed to `LocaleUtils.tr` and `LocaleUtils.trmsg`), store these in `tsconfig.json` and automatically create resp. update the translation files.

In some cases `tsconfig.json` will not pick up a message ID (for instance, if it is computed at runtime). In these cases, the message IDs can be added manually to the `extra_strings` section of the `tsconfig.json`.

Also it may be desired to override a translation inherited from the QWC2 components at application level. To prevent `tsupdate` from continuously reverting the overridden translation, the respective message IDs can be added to the `overrides` section in the application `tsconfig.json` file.

The typical workflow for managing application translations is:

- Declare all locales which should be supported in `js/appConfig.js`.
- Ensure these locales are listed in `translations/tsconfig.json`.
- Write the translated strings to the respective `<locale>.json` files.
- Test the translation, specifying `locale=<locale>` in the QWC2 application URL.

If translations of the QWC2 components for a desired language are missing, please create resp. update the translations respective files in `qwc2/translations` and contribute them by submitting a pull request to the [upstream qwc2 repository](https://github.com/qgis/qwc2).

### <a name="appearance-customization"></a>Customizing the QWC2 appearance

The following options are available for customizing the appearance of the QWC2 application while preserving compatibility with the core QWC2 components:

- Modifying the logo in `assets/img/`.
- Modifying the application icons in `icons`.
- Tweaking the colors in `styleConfig.js`.
- Adding style declarations to the master CSS stylesheet `assets/css/qwc2.css`. This however is potentially fragile and should only be done as a last resort.
- Changing the browser page title in `index.html`, and potentially adding a favicon.
- Modifying the legend print template in `assets/templates/legendprint.html`. The only requirement for this template is that is must contain a `<div id="legendcontainer"></div>` element.
- Modifying the custom Help component `js/Help.jsx`.

*Note*: The common application icons are located in `qwc2/icons`. They can be overridden by creating an icon with the same filename in the application specific `icons` folder.
*Note*: The icons in the `icons` folder are compiled into an icon font. Currently, the icons need to be black content on transparent background, and all drawings (including texts) must be converted to paths for the icons to render correctly.


## Server-side configuration
### <a name="cross-origin-requests"></a>Cross-Origin requests
All modern browsers will block a page from requesting resources from another origin (except for images, stylesheets, scripts, iframes and videos), unless the response from the remote origin contains a matching `Access-Control-Allow-Origin` header. An origin is defined as `<scheme>://<hostname>:<port>`.

For each service QWC2 interacts with, in particular the QGIS Server, one has to ensure that this interaction isn't blocked by the browser. The following options exist:

- Ensure that the service runs on the same origin as the web server which serves the QWC2 application.
- Ensure that the service sends a `Access-Control-Allow-Origin` header with matching origin with each response.
- For development purposes, use a browser plugin which adds the CORS headers, i.e. [CORS Everywhere](https://addons.mozilla.org/en-US/firefox/addon/cors-everywhere/).

### Filenames of print and raster and DXF export
The QGIS server response for the print, raster and DXF export requests does by default not contain any `Content-Disposition` header, meaning that browsers will attempt to guess a filename, which typically is the last part of the URL, without any extension.

To ensure browsers use a proper filename, configure the web server running QGIS Server to add a suitable `Content-Disposition` header to the response. In the case of Apache, the rule for the print output might look as follows:

    SetEnvIf Request_URI "^/wms.*/(.+)$" project_name=$1
    Header always setifempty Content-Disposition "attachment; filename=%{project_name}e.pdf" "expr=%{CONTENT_TYPE} = 'application/pdf'"

This rule will use the last part of the URL as basename and add the `.pdf` extension, and will also ensure that the content-type is set to `application/pdf`. Note that this example uses the `setenvif` and `headers` apache modules.

## <a name="url-parameters"></a>URL parameters
The following parameters can appear in the URL of the QWC2 application:

- `t`: The active theme
- `l`: The layers in the map, see below.
- `bl`: The visible background layer
- `st`: The search text
- `sp`: Applied search provider when using `st` Parameter
- `e`: The visible extent
- `c`: The center of the visible extent
- `s`: The current scale
- `crs`: The CRS of extent/center coordinates
- `hc`: If `c` is specified and `hc` is `true` or `1`, a marker is set at `c` when starting the application. Note: requires the `StartupMarkerPlugin` plugin to be active.

The `l` parameter lists all layers in the map (redlining and background layers) as a comma separated list of entries of the form

    <layername>[<transparency>]!

where
- `layername` is the WMS name of a theme layer or group, or a string of the format

      <wms|wfs>:<service_url>#<layername>
   for external layers, i.e. `wms:https://wms.geo.admin.ch/?#ch.are.bauzonen`.
- `<transparency>` denotes the layer transparency, betwen 0 and 100. If the `[<transparency>]` portion is omitted, the layer is fully opaque.
- `!` denotes that the layer is invisible. If omitted, the layer is visible. If the layer is visible in an invisible parent group, `~` is used.

*Note*: If group name is specified instead of the layer name, QWC2 will automatically resolve this to all layer names contained in that group, and will apply transparency and visibility settings as specified for the group.

The `urlPositionFormat` parameter in `config.json` determines whether the extent or the center and scale appears in the URL.

The `urlPositionCrs` parameter in `config.json` determines the projection to use for the extent resp. center coordinates in the URL. By default the map projection of the current theme is used. If `urlPositionCrs` is equal to the map projection, the `crs` parameter is omitted in the URL.

If the search text passed via `st` results in a unique result, the viewer automatically zooms to this result on startup. If the search result does not provide a bounding box, the `minScaleDenom` defined in the `searchOptions` of the `TopBar` configuration in `config.json` is used.

## Startup position
By default, the viewer opens zooming on the respective theme extent, as defined in `themes.json` (and overrideable in `themesConfig.json`).

Alternatively, the following three options exist to influence the startup position:

- Pass appropriate `c`, `s` or `e` URL parameters, as documented in [URL parameters](#url-parameters).
- Pass a search text which results in a unique result (i.e. a coordinate string) as URL parameter, as documented in [URL parameters](#url-parameters).
- Set `startupMode` in the `LocateSupport` options of the `Map` configuration in `config.json`. Possible values are `DISABLED`, `ENABLED` or `FOLLOWING`. If a search text is passed via `st` URL parameter or `hc=1` is specified in the URL, the `startupMode` is ignored.

## <a name="layer-catalogs"></a>Layer catalogs
The import layer functionality in the layertree also supports loading a layer catalog document from an URL. Two catalog formats are supported:

* QGIS WMS/WFS connections XML: this file is produced by exporting the configured WMS or WFS connections from the QGIS data source manager dialog.
* JSON: a document with contents

      {
        "catalog": [
          {
            "title": "<Titel>",
            "resource": "<wms|wfs>:<service_url>#<layername>"
          },
          ...
        ]
      }

  where `resource` is in the same format as the serialized layer identifiers in a QWC2 URL, see [URL Parameters](#url-parameters).

Note that the server serving the catalog documents needs to ensure that it sets CORS headers appropriately, if they are served from a different origin than QWC2.

## Links in identify results

Links in attributes of GetFeatureInfo reponses are parsed and displayed as clickable links in the identify results window. You can also have links open in an inline dialog using the following format:

    <a href="<url>" target=":iframedialog:<name>:<option1=value1>:<option2=value2>:...">Text</a>

Where:

* `name` The name of the dialog. All links with the same name will open in the same dialog (resp. re-use an already visible dialog). Note that the title of the window will be the translation message ID `windows.<name>`, which should be added to `extra_strings` in `tsconfig.json` and translated in the respective language files.
* Supported options include:
  - `width=<integer>`: Initial window width
  - `height=<integer>`: Initial window height
  - `print=<boolean>`: Whether to display a print icon in the window title bar
  - `dockable=<boolean>`: Whether the window is dockable
  - `docked=<boolean>`: Whether the window is initially docked


## Keeping the QWC2 application up to date
As mentioned in the [quick start](#quick-start) chapter, QWC2 is split into a common components repository and an application specific repository. The goal of this approach is to cleanly separate user-specific configuration and components which common components which serve as a basis for all QWC2 applications, and to make it as easy as possible to rebase the application onto the latest common QWC2 components.

The recommended workflow is to keep the `qwc2` folder a submodule referencing the [upstream qwc2 repository](https://github.com/qgis/qwc2). To update it, just checkout the latest master (or a specific commit if desired):

    cd qwc2
    git checkout master
    git pull
    cd ..
    yarn install

It is good practice to run `yarn install` after pulling the submodule to ensure all the correct dependencies are installed.

The [upgrade notes](https://github.com/qgis/qwc2-demo-app/blob/master/UpgradeNotes.md) documents major changes, and in particular all incompatible changes between releases which require changes to the application specific code and/or configuration.

## API for external applications
The API plugin binds many application actions to the `window.qwc2` object and makes them accessible for external applications. Currently, the following methods are available:

- All [display](https://github.com/qgis/qwc2/blob/master/actions/display.js) actions
- All [layers](https://github.com/qgis/qwc2/blob/master/actions/layers.js) actions
- All [locate](https://github.com/qgis/qwc2/blob/master/actions/locate.js) actions
- All [map](https://github.com/qgis/qwc2/blob/master/actions/map.js) actions
- All [task](https://github.com/qgis/qwc2/blob/master/actions/task.js) actions
- All [theme](https://github.com/qgis/qwc2/blob/master/actions/theme.js) actions
- All [windows](https://github.com/qgis/qwc2/blob/master/actions/windows.js) actions
- Additional API calls:
  - `window.qwc2.addExternalLayer(resource, beforeLayerName=null)`
  - `window.qwc2.drawScratch(geomType, message, drawMultiple, callback, style = null)`

See the docstrings in [API.js](https://github.com/qgis/qwc2/blob/master/plugins/API.jsx) as well as the actions functions linked above for more information.

See [api_examples.js](https://github.com/qgis/qwc2-demo-app/blob/master/api_examples.js) for some concrete examples.


## Developing
QWC2 is written in JavaScript using in particular the ReactJS, Redux and OpenLayers libraries. The following links point to some useful resources to learn the basics:

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

When developing, it is useful to add `debug=true` to the URL query parameters, which will enable logging of all application state changes to the browser console.
