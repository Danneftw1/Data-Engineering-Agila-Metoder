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

## **dashboard.py**

*Kodexempel nedan är inte koden i dess helhet, endast korta utdrag* 

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
# Använder 'argparse'-biblioteket för att läsa kommandordsargument.
# Retunerar en parserad lista med argument.

if __name__ == "__main__":
    args = parse_args()
    main(blog_name=args.blog_name)
# Kör parse_args() för hämta argument via terminalen
# Kör Main-funktionen med angivna bloggnamnet.
```