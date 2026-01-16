# Jan Toebes Blog

Een minimale, brutalist dark mode blog over **Agentic AI**.

🌐 **Live op**: [https://toebes.nl](https://toebes.nl)

## Over

Dit blog behandelt onderwerpen rondom Agentic AI - autonome AI-systemen die zelfstandig kunnen plannen, beslissen en taken uitvoeren. Van praktische implementaties tot theoretische overwegingen.

## Technologie

- **Jekyll** - Statische site generator
- **GitHub Pages** - Automatische deployment
- **Markdown** - Content formaat
- **Dark Mode** - Brutalist design aesthetic
- **Rouge** - Syntax highlighting voor code

## Nieuwe Blog Post Toevoegen

### Stap 1: Maak een nieuw bestand

Maak een nieuw bestand in de `_posts/` directory met deze naming convention:

```
_posts/YYYY-MM-DD-titel-slug.md
```

Bijvoorbeeld: `_posts/2026-01-17-multi-agent-systemen.md`

### Stap 2: Voeg front matter toe

Bovenaan het bestand, voeg de volgende YAML front matter toe:

```yaml
---
layout: post
title: "Je Artikel Titel"
date: YYYY-MM-DD
---
```

### Stap 3: Schrijf content in Markdown

Onder de front matter kun je gewoon Markdown gebruiken:

```markdown
---
layout: post
title: "Multi-Agent Systemen"
date: 2026-01-17
---

Dit is mijn artikel over multi-agent systemen...

## Subkop

Gewone tekst met **vet** en *cursief*.

### Code Voorbeelden

```javascript
console.log('Hello world!');
```

- Lijsten
- Werken
- Ook

En [links](https://example.com) natuurlijk.
```

### Stap 4: Commit en push

```bash
git add _posts/YYYY-MM-DD-titel-slug.md
git commit -m "feat: nieuwe post over [onderwerp]"
git push origin main
```

### Stap 5: Verifieer deployment

- Wacht 1-2 minuten voor GitHub Pages de site herbouwt
- Bezoek https://toebes.nl om je nieuwe post te zien
- De nieuwste post verschijnt automatisch op de homepage
- Alle posts verschijnen in de linker sidebar

## Lokaal Testen (Optioneel)

Als je de site lokaal wilt testen voor je push naar GitHub:

### Vereisten

- Ruby 2.7 of hoger
- Bundler (`gem install bundler`)

### Setup

```bash
# Installeer dependencies (eerste keer)
bundle install

# Start lokale development server
bundle exec jekyll serve

# Bezoek http://localhost:4000 in je browser
```

### Lokaal testen zonder Ruby

Als je geen Ruby lokaal hebt, kun je ook gewoon pushen naar GitHub en testen op de live site. GitHub Pages bouwt automatisch voor je.

## Markdown Features

De blog ondersteunt alle standaard Markdown features:

### Tekst Formatting

- **Vet** met `**tekst**`
- *Cursief* met `*tekst*`
- `Inline code` met backticks
- [Links](https://example.com) met `[tekst](url)`

### Kopjes

```markdown
# H1 Kop
## H2 Kop
### H3 Kop
```

### Lijsten

**Ongenummerd:**
```markdown
- Item 1
- Item 2
  - Sub-item
```

**Genummerd:**
```markdown
1. Eerste
2. Tweede
3. Derde
```

### Code Blocks

Voor code blocks met syntax highlighting, gebruik triple backticks met de taal:

````markdown
```javascript
function hello() {
  console.log('Hello!');
}
```

```python
def hello():
    print("Hello!")
```
````

Ondersteunde talen: javascript, python, ruby, bash, html, css, json, yaml, en meer.

### Blockquotes

```markdown
> Dit is een quote
> Over meerdere regels
```

### Afbeeldingen

```markdown
![Alt tekst](path/to/image.png)
```

Voor afbeeldingen: plaats ze in een `assets/images/` directory en reference ze relatief.

### Tabellen

```markdown
| Kolom 1 | Kolom 2 |
|---------|---------|
| Data 1  | Data 2  |
| Data 3  | Data 4  |
```

## Project Structuur

```
toebes.nl/
├── _config.yml              # Jekyll configuratie
├── _layouts/
│   ├── default.html        # Base layout met sidebar
│   └── post.html           # Blog post layout
├── _posts/                 # Blog artikelen (Markdown)
│   └── YYYY-MM-DD-slug.md  # Post files
├── _sass/
│   └── main.scss           # SCSS styling
├── css/
│   └── styles.scss         # Main stylesheet
├── index.html              # Homepage (toont laatste post)
├── 404.html                # Error pagina
├── favicon.svg             # Site icon (SVG)
├── favicon-*.png           # Site icons (PNG)
├── Gemfile                 # Ruby dependencies
├── .gitignore              # Git ignore rules
├── AGENTS.md              # Guidelines voor AI agents
└── README.md              # Deze file
```

## Design Principes

### Brutalist Aesthetic

- **Minimaal**: Alleen wat nodig is, niets meer
- **Functioneel**: Content first, design second
- **Hard edges**: Geen rounded corners, shadows of gradients
- **System fonts**: Geen web fonts, alleen system font stack
- **Dark mode**: Hoog contrast, oog-vriendelijk

### Kleuren

- Achtergrond: `#0a0a0a` (bijna zwart)
- Tekst: `#e0e0e0` (licht grijs)
- Sidebar: `#000000` (echt zwart)
- Borders: `#333333` (donker grijs)
- Accent: `#ffffff` (wit)

### Layout

- **Sidebar**: 280px breed, fixed positie, scrollable
- **Content**: Max 800px breed voor leesbaarheid
- **Mobile**: Stacked layout (sidebar boven content)

## Deployment

Deze site wordt automatisch gedeployed via **GitHub Pages** wanneer je push naar de `main` branch.

### Deployment Proces

1. Push wijzigingen naar `main` branch
2. GitHub Pages detecteert de push
3. Jekyll bouwt de statische site (1-2 minuten)
4. Site is live op https://toebes.nl

### Custom Domain

De site gebruikt het custom domain `toebes.nl` geconfigureerd in GitHub repository settings.

## Toekomstige Features

Mogelijke toevoegingen:

- **About pagina** - Over de auteur
- **Archive pagina** - Chronologische lijst van alle posts
- **Tags/Categorieën** - Posts organiseren per onderwerp
- **Zoekfunctionaliteit** - Posts doorzoeken
- **Related posts** - Gerelateerde artikelen onder posts
- **Comments** - Via GitHub Issues of externe service

## License

MIT

## Contact

Voor vragen of suggesties, open een issue op GitHub.
