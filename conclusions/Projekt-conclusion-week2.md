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
**Pyproject**: Anger inställningar för olika verktyg i vårt projekt. I vårt fall har vi konfigurationer för två verktyg: Black och Isort. Black formaterar koden på ett speciellt sätt (gör den lättare att läsa), och Isort automatiskt sorterar och organiserar imports för våra scripts.

```toml
[tool.black]
line-length = 100

[tool.isort]
profile = "black"
multi_line_output = 3
```

---
# Varje script förklarat

*Kodexempel nedan är inte koden i dess helhet, endast korta utdrag*

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

def save_metadata_info(xml_text, blog_name):
# Tar emot XML-data och bloggnamnet som argument.
# Sparar XML-data i en fil i en specifik katalog ("data/data_lake")

def main(blog_name):
# Hämtar data via get_metadata_info()
# Sparar den via save_metadata_info()

def parse_args():
# Använder 'argparse'-biblioteket för att läsa kommandordsargument (terminal).
# Retunerar en parserad lista med argument.

if __name__ == "__main__":
    args = parse_args()
    main(blog_name=args.blog_name)
# Kör parse_args() för hämta argument via terminalen
# Kör Main-funktionen med angivna bloggnamnet.
```

## **extract_articles.py**

Den här koden bearbetar bloggartiklar. Den läser in en XML-fil med metadata, extraherar informationen om varje artikel och sparar den i JSON-format.

### Funktioner

```py
def create_uuid_from_string(title):
# Skapar en unik identifierare baserad på  artikelns titel

def load_metadata(blog_name):
# Läser in XML-metadata från en fil
# Retunerar filen som ett BeautifulSoup-objekt för vidare bearbetning

def extract_articles_from_xml(parsed_xml):
# Tar in BeautifulSoup-objektet som argument (parsade XML-data) och extraherar artiklar.
# Skapar en lista med 'BlogInfo'-object

def save_articles(articles, blog_name):
# Sparar artiklarna som JSON-filer i specifik map.

def main(blog_name):
# Kör funktionerna ovan
def parse_args():
# Gör så att man kan skriva "--blogg_name" i terminalen
```

---

## **send_summaries_to_discord.py**

Hämtar, formaterar och skickar sammanfattningar till en Discord-webhook.

*Async: Tillåter ditt program att starta potentiellt långvariga uppgifter och fortfarande vara mottaglig till andra händelser medans den uppgiften körs, istället för att behöva vänta tills den uppgiften har slutförts*

### Funktioner

```py
async def get_articles_from_folder(folder_path):
# Asynkront läser in JSON-filer från en specifik mapp och returnerar en lista av artiklar

def format_summary_message(summary_item, group_name):
# Formaterar sammanfattningar av artiklar för att skickas till Discord

async def send_summary_to_discord(blog_name):
# Använder asynkron HTTP-session för att skicka formaterade artikelmeddelanden till en Discord-webhook

if __name__ == "__main__":
    args = utils.parse_args()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(send_summary_to_discord(blog_name=args.blog_name))
# kör parge_args()
# Initialiserar en asynkron loop och kör funktionen send_summary_to_discord tills den är klar.
```
---

## **summarize.py**

Den kod tar ett bloggnamn som kommandoradsargument, läser in dess artiklar, genererar sammanfattningar med hjälp av GPT-3.5 turbo och sparar dessa sammanfattningar i en mapp.

### Funktioner

```py
def get_articles_from_folder(blog_name):
# Läser in JSON-filer som representerar artiklar från data/data warehouse och returnerar en lista av artiklar.

load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")
# Laddar in vår GPT API key från .env filen
# Detta är inte en funktion, kan dock va bra att ha med

def summarize_text(blog_text):
# Använder GPT-3.5 turbo för att generera en sammanfattning av en given text.

def transform_to_summary(article: BlogInfo) -> BlogSummary:
# Tar en BlogInfo-instans, genererar en sammanfattning och returnerar en BlogSummary-instans (datatypes.py)

def save_blog_summaries(articles, blog_name):
# Sparar genererade sammanfattning i en särskild mapp i data warehouse.

if __name__ == "__main__":
    args = (utils.parse_args())  
    blog_name = args.blog_name
    articles = get_articles_from_folder(blog_name)
    save_blog_summaries(articles, blog_name)
# Använder parse_args() för terminal
# Anropar get_articles_from_folder och save_blog_summaries för att generera och spara sammanfattningar.
```

## translation_model.py

---

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