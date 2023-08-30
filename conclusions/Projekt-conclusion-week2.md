# Sammanställning av Projekt
## Generellt

Vi har ett projekt som automatiserar upphämtningen, visualisering (dashboard) och sammanfattningen av artiklar från olika bloggar/universitet (RSS feed*).

Vi börjar med att ladda ner artiklarna i .xml format och detta inkluderar metadata för artiklarna. Vi använder oss utav två pydantic data modeller som strukturerar datan från artiklarna på det sättet som vi vill ha. Detta ser till att vi får consistency och gör det lättare att hantera data i andra delar av applikationen.

Vi extraherar och parsar sedan artiklarna, det här gör att vi kan plocka ut title, link, description, text och published date. Det här sparas sedan i en .json fil i mappen 'data/data_warehouse'. 

Efter det här steget så kan vi nu använda våran summarize.py fil som använder sig utav gpt-3.5 för att sammanfatta texterna i artiklarna. Den läser en .json artikel från vårat data warehouse, sammanfattar och sparar sedan sammanfattningen i en separat map i data warehouse.

När vi har våra sparade sammanfattningar så kör vi vår dashboard. Dashboarden inkluderar en dropdown meny där användaren kan välja vilken blogg som ska visas. Varje resultat inkluderar titel, date published, artikellänk till blogg och vår AI-genererade sammanfattning av artikeln.

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

Den skapar först en Dash app och importerar interface-designen. Sedan letar den upp artiklarna och sammanfattningar där dem är sparade.

```py
app = dash.Dash
app.layout = layout

NEWS_ARTICLES_SUMMARY_SOURCES = {
    "mit": Path(__file__).parent.parent.parent / f"data/data_warehouse/mit/summaries"
    ...
NEWS_ARTICLES_ARTICLE_SOURCES = {
    "mit": Path(__file__).parent.parent.parent / f"data/data_warehouse/mit/articles"
    ...
```
### Funktioner:

```py
def read_json_files_to_df(folder_path):
# Läser JSON-filer från en given mapp
# Omvandlar dessa filer till en dataframe

def get_news_data(news_blog_source="all_blogs"):
# Samlar data från en specifik källa eller alla beroende på ditt argument
# Hämtar datan via read_json_files_to_df
```

### Dash Callbacks

```py
@app.callback(Output("blogs-df", "data"), [Input("data-type-dropdown", "value")])
def blogs_df(selected_data_type):
    if selected_data_type == "all_blogs":
        all_blogs = get_news_data("all_blogs")
        return all_blogs.to_dict("records")
    elif selected_data_type == "google_ai":
        google_ai = get_news_data("google_ai")
        return google_ai.to_dict("records")
    ...
# Denna callbacken utlöses varje gång dropdown-menyn interfacet ändras.
# Data hämtas baserat på "all_blogs", "mit", etc. Sedan omvandlas datan (från DF till dict) så den kan visas på webbsidan
```

```py
@app.callback(
    [Output("blog-heading", "children"), Output("content-container", "children")],
    [Input("dropdown-choice", "value")],
)
def display_blogs(choice):
# Utlöses baserat på ett annat dropdown-val.
# Visar titeln, utgivningsdatum, sammanfattningen och länken till artikeln från den källa som speciferas i 'choice'.
```
---

## **layout.py**

Använder ett webbapplikationsramverk, och Dash Bootstrap Components för att styla webbsidan. Koden definierar layouten av en webbsida, inklusive olika element som kort, rader och kolumner, för att visa olika typer av innehåll.

Det är lite svårt att sammanfatta vad som sker genom koden, så jag har valt att inte ha med kodexempel här.

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

---

## **utils.py**

Innehåller en funktion parse_args(), som använder argparse för att tolka ett kommandoradsargument ('--blog_name') och returnera det som ett 'Namespace'-objekt. Detta argument är avsett att specificera vilken bloggkälla som ska behandlas.

### Funktioner

```py
def parse_args():
# Skapar en 'argparse.ArgumentParzer'-instans för att tolka kommandoradsargument
# Lägger till ett argument '--blog_name' av typen 'str', som avsett att ta emot namnet på en specifik bloggkälla.
# Returnerar ett 'Namespace'-objekt som innehåller de argument som har tolkats från kommandoraden.
```