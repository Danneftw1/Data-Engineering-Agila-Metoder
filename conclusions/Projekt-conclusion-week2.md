# Sammanställning av Projekt
## Generellt

Vi har ett projekt som automatiserar upphämtningen, visualisering (dashboard) och sammanfattningen av artiklar från olika bloggar/universitet (RSS feed*).

Vi börjar med att ladda ner artiklarna i .xml format och detta inkluderar metadata för artiklarna. Vi använder oss utav två pydantic data modeller som strukturerar datan från artiklarna på det sättet som vi vill ha. Detta ser till att vi får consistency och gör det lättare att hantera data i andra delar av applikationen.

Vi extraherar och parsar sedan artiklarna, det här gör att vi kan plocka ut title, link, description, text och published date. Det här sparas sedan i en .json fil i mappen 'data/data_warehouse'. 

Efter det här steget så kan vi nu använda våran summarize.py fil som använder sig utav gpt-3.5 för att sammanfatta texterna i artiklarna. Den läser en .json artikel från vårat data warehouse, sammanfattar och sparar sedan sammanfattningen i en separat map i data warehouse.

När vi har våra sparade sammanfattningar så kör vi vår dashboard. Dashboarden inkluderar svenska och engelska översättningar. Vi har också en dropdown meny där användaren kan välja vilken blogg som ska visas. Varje resultat inkluderar titel, date published, artikellänk till blogg och vår AI-genererade sammanfattning av artikeln.

*: *RSS feed: Möjliggör att användaren kan prenumerera på webbflöden, man blir omedelbart kontaktad när något nytt publiceras.*


## Övriga Funktion (Generellt)

Utöver förklaringen ovan så har vårt projekt fler funktioner som är följande:

Vi kan skicka liknande sammanfattning som vi har på vår dashboard till Discord via en webhook (callback funktion som är en event-driven kommunikation mellan 2 API). 

Vi använder oss också utav en **CI/CD pipeline**. Det här betyder att kodändringar automatiskt testas och ändringar kan genomföras på ett korrekt sätt. Detta ökar hastigheten på utvecklingscykeln, minskar manuellt arbete och minimerar risken för fel. Detta görs genom Pre-commit, Makefil och pyproject.

**Pre-commit**: Varje gång vi försöker commita och pusha till Git så körs tester som kontrollerar att koden håller den kvalitén vi vill ha (speciferas i .pre-commit-config.yaml filen). Vi kan se några exempel på vad pre-commiten kollar efter:

```yaml
---
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.4.0
  hooks:
  - id: trailing-whitespace
  - id: check-yaml
  - id: check-toml
  - id: check-added-large-files
  - id: debug-statements
    language_version: python3
```

**Makefil**: Utför specifika uppgifter som underlättar utvecklingsprocessen. Skriver vi tex 'make install_dependencies' så körs alla kommandon som ligger under.

```Makefile
install_dependencies:
	python3.10 -m venv venv
	source venv/bin/activate && pip install -r         requirements.txt
	source venv/bin/activate && pre-commit install --hook-type pre-push --hook-type post-checkout --hook-type pre-commit
```
**Pyproject**: Anger inställningar för olika verktyg i vårt projekt. I vårt fall har vi konfigurationer för två verktyg: Black och Isort. Black formaterar koden på ett speciellt sätt (gör den lättare att läsa), och Isort automatiskt sorterar och organiserar imports för våra scripts. Utöver detta så konfigurerar vi också pytest för våra tester som ligger i 'tests'-mappen.

```toml
[tool.black]
line-length = 100

[tool.isort]
profile = "black"
multi_line_output = 3

[tool.pytest.ini_options]
log_cli = 1
addopts = "-rA -s --strict-markers -m 'local_test'"
markers = [
    "local_test: marks tests as local_test that will NOT run on CI/CD (deselect with '-m \"not local_test\"')",
]
```

**Docker**: Docker är en plattform för att utveckla och köra applikationer i containrar. En container innehåller kod och alla dess dependencies så att applikationen kör snabbt och tillförlitligt från en virtual enviroment till en annan. Docker används ofta för att förenkla driftsättning och skalning av applikationer.

```py
FROM apache/airflow:latest-python3.10
COPY requirements.txt requirements.txt

RUN grep -v "^-e" requirements.txt > requirements_without_editable_install.txt
RUN pip install -r requirements_without_editable_install.txt
# I den här Docker filen skapas en Docker container baserad på en specifik version av Airflow och Python, 
# kopierar över en fil med Python-library-dependencies,
# rensar bort "editable" installartioner och installerar sedan de nödvändiga biblioteken. 
# Detta ger en container som är redo att köra vår applikation.
```

Utöver detta så har vi också en **Docker-compose.yml** fil. Den används för att definiera och köra flera Docker-containrar som en del av en applikation. Med hjälp av den här filen kan vi konfigurera tjänster, volymer och nätverk för vår applikation direkt i YAML-format. 'docker-compose' kommandot läser sedan denna fil och skapar de definierade tjänsterna och resurserna. Detta gör det enkelt att hantera komplexa applikationer som består av flera containrar.

---
# Varje script förklarat

*Kodexempel nedan är inte nödvändigtvis koden i dess helhet, endast korta utdrag*

## **dashboard.py** 

Skapar en webdashboard som visar sammanfattningar av artiklar från olika källor (MIT, Google & AI Blog). Använder sig utav Dash för att skapa en interface.

Den skapar först en Dash app och importerar interface-designen, samt pathen för alla våra artiklar, sammanfattningar och deras språk.

```py
from newsfeed.utils import (
    NEWS_ARTICLES_ARTICLE_SOURCES,
    NEWS_ARTICLES_SUMMARY_SOURCES,
    SWEDISH_NEWS_ARTICLES_SUMMARY_SOURCES,
    formated_source,
    source_dict,
)

app = dash.Dash(
    __name__,
    meta_tags=[dict(name="viewport", content="width=device-width, initial-scale=1.0")],
)
app.layout = layout

server = app.server
```
### Funktioner:

```py
def read_json_files_to_df(folder_path):
# Läser JSON-filer från en given mapp
# Omvandlar dessa filer till en dataframe

def get_news_data(news_blog_source="all_blogs"):
# Hämtar nyhetsdata antingen från alla källor eller en specifik källa, beroende på parametrarna.
# Stödjer också engelska och svenska.

def fetch_and_prepare_articles(language, df):
# Hämtar och förbereder artiklar för visning, 
# kompletterar dem med ytterligare data (ex: datum, länk)
```

### Dash Callbacks

```py
@app.callback(Output("blogs-df", "data"), [Input("data-type-dropdown", "value")])
def blogs_df(selected_data_type):
    news_data = get_news_data(selected_data_type)
    return news_data.to_dict("records")
# Hämtar nyhetsdata baserat på den valda typen och uppdaterar en komponent med ID "blogs-df" i Dash layouten.
```

```py
@app.callback(
    Output("language-store", "data"),
    [Input("btn-english", "n_clicks"), Input("btn-swedish", "n_clicks")],
    [State("language-store", "data")],
)
def update_language(n_clicks_english, n_clicks_swedish, data):
# Uppdaterar språkpreferensen baserat på vilken knapp (engelska eller svenska) som klickades.
```

```py
@app.callback(
    [Output("blog-heading", "children"), Output("content-container", "children")],
    [
        Input("dropdown-choice", "value"),
        Input("language-store", "data"),
        Input("search-btn", "n_clicks"),
    ],
    [State("blogs-df", "data"), State("search-input", "value")],
)
def display_blogs(choice, language_data, n_clicks, blogs_data, search_query):
# Hämtar och visar bloggar baserat på flera inputs som dropdown-val, språkpreferens och en search-query. Den sorterar artiklarna efter datum och kan också filtrera dem baserat på sökfrågan.
```
---

## **layout.py & article_item.py**

Layout.py ställe in "skalet" eller den övergripande strukturen i vår app. article_item.py fokuserar på att fylla det "skalet" med faktiska nyhetsartiklar och deras detaljer, anpassade till användarens språkpreferenser.

### Layout.py: 
Sätter upp applikationens rubrik, logotyp, rullgardinsmenyer för att välja nyhetstyp, ett sökfält och den primära innehållsbehållaren. där nyhetsartiklarna kommer att visas. Varje del av layouten är innesluten i funktioner. Nedan kan vi se ett exempel på logotypen på dashboarden:

```py
def create_logo():
    """Creates the logo section of the layout"""
    return dbc.Col(
        dbc.Card(
            dbc.CardBody(
                [
                    html.Img(
                        id="midjourney-logo",
                        src="assets/midjourney-logo.png",
                        style={
                            "position": "absolute",
                            "top": "-2%",
                            "left": "-2%",
                            "width": "300px",
                        },
                    )
                ]
            ),
        )
    )
# Utöver den här funktionen har vi också create_blog_heading som gör samma sak som create_logo
```

### Article_item.py
Detta script är mer inriktat på representationen av enskilda nyhetsartiklar. Det definierar hur varje nyhetsartikel kommer att visas inom innehållsbehållaren som vi definierade i layout.py

```py
def news_artcle_div(
    title, published_date, technical_summary, non_technical_summary, link, language, article_source
):
# Tar emot detaljer om en nyhetsartikel som dess titel, datum, sammanfattningar, länk och källa och returnerar en dictionary som innehåller ett "datum" och en "div"
# Div:en innehåller HTML-strukturen för att visa den nyhetsartikeln, formaterad baserat på det valda språket.
```

```py
def title_heading_for_dashboard(heading: str):
# Tar en sträng "heading" och returnerar en dash HTML Div med den angivna rubriken stiliserad enligt specifikationerna.
```

```py
def dashboard_content_container(children):
# Tar children-element och returnerar en Dash HTML Div som innehåller dem.
```

---
## **datatypes.py**

Använder två Pydantic-modeller 'Bloginfo' och 'BlogSummary' för att strukturera data kring artiklar och deras sammanfattningar. Varje modell har en metod för att generera ett unikt filnamn för lagring.

### Klasser & Modeller

**BlogInfo**: Inkluderar fält med krav på strukturen vilket MÅSTE finnas med (om man inte skriver med 'optional' efteråt). 

```py
class BlogInfo(pydantic.BaseModel):
    unique_id: str
    title: str
    description: Optional[str] = None
```

**BlogSummary**: Samma som BlogInfo fast annorlunda krav.

Sedan har båda dessa klasser med en funktion som heter get_filename. Den retunerar ett filnamn baserat på artikelns titel, där mellanslag är ersatta med understreck och filen har JSON formatet.

---
## **download_blogs_from_rss.py**

Hämtar och lagrar metadata från artiklar i .xml-format från olika bloggkällor.

### Funktioner

```py
def get_metadata_info(blog_name):
# Tar emot namn på en blogg som argument
# Hämtar den XML-data som finns på bloggens RSS-flöde
# Returnerar XML-data som en textsträng.
```
```py
def save_metadata_info(xml_text, blog_name):
# Tar emot XML-data och bloggnamnet som argument.
# Sparar XML-data i en fil i en specifik mapp ("data/data_lake")
```

Det här scriptet innehåller både main och parge-args.

## **extract_articles.py**

Den här koden bearbetar bloggartiklar. Den läser in en XML-fil med metadata, extraherar informationen om varje artikel och sparar den i JSON-format.

### Funktioner

```py
def create_uuid_from_string(title):
# Skapar en unik identifierare baserad på  artikelns titel
```py
def load_metadata(blog_name):
# Läser in XML-metadata från en fil
# Retunerar filen som ett BeautifulSoup-objekt för vidare bearbetning
```
```py
def extract_openai_articles(soup):
# Med det tolkade XML/HTML-innehållet skrapar denna funktion OpenAIs blogg och samlar olika detaljer om varje blogginlägg, inklusive titel, beskrivning, URL och det faktiska blogginnehållet.
# Den returnerar en lista med BlogInfo-objekt.
```
```py
def extract_articles_from_xml(parsed_xml, blog_name):
# Allmän funktion som extraherar artiklar från det angivna XML-innehållet baserat på blog_name. 
# Om bloggnamnet är "open_ai" används "extract_openai_articles"; annars tolkas XML för att hitta andra typer av artiklar.
```
```py
def save_articles(articles, blog_name):
# Sparar artiklarna som JSON-filer i specifik map.
# Felmeddelandet skickas ifall något går fel här.
```
Det här scriptet innehåller också en main och parge-args funktion.

## **Hugging_face_model.py**

Här använder vi Hugging Face Transformers-biblioteket för att sammanfatta text med hjälp av en sekvens-till-sekvens maskininlärningsmodell. Vi använde oss utav den här modellen för att kunna köra det lokalt och inte behöva slå i penga-taket konstant som vi gjorde med gpt-3.5. Den är dock inte i närheten lika bra, men fungerar någolunda okej.

### Funktioner

```py
def summarize_text_with_hugging_face(
    text, model_name="facebook/bart-large-cnn", max_length=250, min_length=25
):
# Tar emot en text att sammanfatta, tillsammans med flera andra valfria parametrar som modellnamn, maximal sammanfattningens längd och minimal sammanfattnignens längd.
```

---

## **send_summaries_to_discord.py**

Hämtar, formaterar och skickar sammanfattningar till en Discord-webhook.

*Async: Tillåter ditt program att starta potentiellt långvariga uppgifter och fortfarande vara mottaglig till andra händelser medans den uppgiften körs, istället för att behöva vänta tills den uppgiften har slutförts*

### Funktioner

```py
async def get_articles_from_folder(folder_path):
# Asynkront läser in JSON-filer från en specifik mapp och returnerar en lista av artiklar
```
```py
def format_summary_message(summary_item, group_name, language="en"):
# Tar en artikel, ett gruppnamn och ett språk som argument. Den formaterar artikeln till ett meddelande som kan skicka till Discord. 
# Funktionen tar också hänsyn till språket.
```
```py
def truncate_string(input_str, max_len):
# Denna funktion tar en sträng och en maximal längd som argument och returnerar en trunkerad version av strängen.
```
```py
def hash_summary(summary):
# Denna funktion genererar en unik hash för varje sammanfattning. Detta används senare för att kontrollera om sammanfattningen redan har skickat eller inte.
```
```py
def read_sent_log():
def write_sent_log(sent_log):
# Dessa funktioner hanterar en loggfil som sparar hasharna för de sammanfattningar som redan har skickats.
```
```py
async def send_summary_to_discord(blog_name, language="en"):
# Tar ett bloggnamn och ett språk som argument.
# Den läser alla sammanfattningar från en given mapp, skickar de som ännu inte har skickats till Discord, och uppdaterar sedan loggfilen.
```
Detta script innehåller också en main och parse-args funktion

---

## **summarize.py**

Den kod tar ett bloggnamn som kommandoradsargument, läser in dess artiklar, genererar sammanfattningar med hjälp av GPT-3.5 turbo och sparar dessa sammanfattningar i en mapp.

### Funktioner

```py
def get_articles_from_folder(blog_name):
# Läser bloggartiklar från en mapp och omvandlar dem till en lista av BlogInfo-objekt
```
```py
def get_summaries_from_folder(blog_name):
# Läser befintliga sammanfattningar från en mapp och omvandlar dem till en lista av BlogSummary-objekt
```
```py
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")
# Laddar in vår GPT API key från .env filen
# Detta är inte en funktion, kan dock va bra att ha med
```
```py
def summarize_text(blog_text):
# Sammanfattar given text baserad på en specifik promptmall med hjälp av OpenAI GPT-3.5 turbo-modellen.
```
```py
def transform_to_summary(article: BlogInfo, model_type) -> BlogSummary:
# Tar en artikel representerad av ett BlogInfo-objekt, genererar tekniska och icke-tekniska sammanfattningar, och returnerar sedan dem som ett BlogSummary-objekt.
```
```py
def save_blog_summaries(articles, blog_name, model_type):
# Sparar genererade sammanfattning som en JSON-fil i en särskild mapp i data warehouse.
```
```py
def main(blog_name, model_type):
# Huvudfunktionen som gör följande:
# Kontrollerar om sammanfattningarna redan existerar.
# Om ja, identifierar dem artiklar som ännu inte har sammanfattats.
# Kallar på 'save_blog_summaries' för att generera och spara sammanfattningar.
```
### Vad scriptet gör

1. Skriptet börjar med att tolka kommandoordsargument för att identifiera vilken blogg man ska fokusera på och vilken modell man ska använda för sammanfattning.
2. Därefter anropas main()-funktionen som kontrollerar om en sammanfattningsmapp redan finns för den angivna bloggen.
3. Om mappen existerar, hämtar dem artiklar och befintliga sammanfattningar för att identifiera vilka artiklar som behöver sammanfattas.
4. Den kallar sedan på 'save_blog_summaries' för att spara sammanfattningarna.
---

## **translation_model.py**

Använder MarianMT-modellen från Helsinki-NLP för översättning, som är en del av "transformers"-biblioteket.

### Funktioner

```py
def translate_initialised(text, model, tokenizer):
# Tar in en textsträng, en förtränad modell och en tokenizer, och returnerar den översatta texten.
```
```py
def translate_summaries(blog_name):
# Går igenom alla JSON-filer i en viss bloggs sammanfattningsmapp och översätter dem.
# Den sparar sedan de översatta sammanfattningarna i en separat mapp.
```
Dessutom finns main(blog_name), och parse_args som fungerar likadant i andra scripts.
## **utils.py**

Innehåller en funktion parse_args(), som använder argparse för att tolka ett kommandoradsargument ('--blog_name') och returnera det som ett 'Namespace'-objekt. Detta argument är avsett att specificera vilken bloggkälla som ska behandlas.

Utöver detta så har vi paths till olika mappar som vi behöva komma åt i andra scripts, men eftersom dem tar upp så mycket plats så är det bättre att bara importera om man behöver dem i en annan fil.

### Funktioner

```py
def parse_args():
# Skapar en 'argparse.ArgumentParzer'-instans för att tolka kommandoradsargument
# Lägger till ett argument '--blog_name' av typen 'str', som avsett att ta emot namnet på en specifik bloggkälla.
# Returnerar ett 'Namespace'-objekt som innehåller de argument som har tolkats från kommandoraden.
```
---

# **Tester**

Vi har tre test-scripts som utför olika typer av tester för att verifiera olika aspekter av ett större system.

Sammantaget är dessa testskript utformade för att säkerställa att nyhetsflödesapplikationen fungerar som förväntat när det häller att samla in och hantera nyhetsdata, att alla dependencies är installerade och att rätt Python-version används.

## test_dashboard.py

Använder pytest för att genomföra fyra tester

### Funktioner

```py
@pytest.mark.local_test("Run with 'pytest -m local_test tests/'")
# decoratorn här testar funktionen som kommer direkt efter
def test_get_news_data_source_mit():
    df = get_news_data(news_blog_source="mit")
    assert "source" in df.columns
    assert all(df["source"] == "mit")
# Fungerar korrekt när nyhetskällan är "mit". Det säkerställer att "source"-kolumnen i den returnerade dataframe innehåller endast "mit"
```
Utöver denna finns det två till som gör exakt samma sak, fast för andra nyhetskällor (google_ai & ai_blog)

```py
@pytest.mark.local_test("Run with 'pytest -m local_test tests/'")
def test_get_news_data_source_all_blogs():
    df = get_news_data(news_blog_source="all_blogs")
    assert "source" in df.columns
    assert set(df["source"].unique()) == {"mit", "google_ai", "ai_blog"}
# Kan hantera flera nyhetskällor samtidigt och returnerar en dataframe med rätt "source"-värden.
```
---

## test_download_blogs_from_rss.py

Utför två tester: 

### Funktioner

```py
def test_import_project() -> None:
    print("Running test_import_project...")
    download_blogs_from_rss.main(blog_name="mit")
    print("Test completed.")
# Testar huvudfunktionen 'main()' i 'download_blogs_from_rss' modulen fungerar korrekt med "mit" som nyhetskälla.
```

```py
def test_imports():
    essential_modules = [
        "pydantic",
        "argparse",
        "requests",
        "newsfeed.download_blogs_from_rss",
        "dash",
        "dash_bootstrap_components",
        "pandas",
        "dash.dependencies",
        "bs4",
        "aiohttp",
        "discord",
        "openai",
        "tiktoken",
        "dotenv",
        "langchain",
    ]

    for module in essential_modules:
        try:
            __import__(module)
        except ImportError:
            assert False, f"Failed to import {module}"
# Säkerställer att alla nödvändiga moduler och paket kan importeras korrekt, vilket indikerar att alla dependencies är uppfyllda.
```
---

## test_python_version_is_10.py

Utför ett test: 

### Funktion

```py
def test_python_version():
    minor_version = sys.version_info[1]
    major_version = sys.version_info[0]
    assert (
        major_version == 3 and minor_version == 10
    ), f"The expected and required Python version for this project is 3.10.*, but got {major_version}.{minor_version}.*"
# Kontrollerar att Python-versionen som används för att köra projektet är 3.10. Om det inte är fallet, genereras ett fel.
```