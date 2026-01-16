# Agent Guidelines for toebes.nl

This document provides coding guidelines and commands for AI coding agents working in this repository.

## Project Overview

**Type**: Jekyll blog (GitHub Pages)  
**Repository**: github.com/jantoebes/toebes.nl  
**Primary Language**: Markdown, HTML, CSS (SCSS)  
**Framework**: Jekyll 4.3+  
**Deployment**: Automatic deployment via GitHub Pages from main branch  
**Theme**: Agentic AI blog with brutalist dark mode design

## Build & Development Commands

### Jekyll Setup

This is a **Jekyll-based static site** that is automatically built by GitHub Pages.

**Technology Stack**:
- Jekyll 4.3+ (static site generator)
- Liquid (templating language)
- Kramdown (Markdown processor)
- Rouge (syntax highlighting)
- SCSS (CSS preprocessor)
- GitHub Pages (automatic deployment)

### Local Development (Optional)

```bash
# Install Ruby dependencies (first time only)
bundle install

# Run local development server
bundle exec jekyll serve
# Visit http://localhost:4000

# Run with live reload
bundle exec jekyll serve --livereload

# Build site for production (GitHub does this automatically)
bundle exec jekyll build

# Clean generated files
bundle exec jekyll clean
```

**Note**: Local development is optional. You can push directly to `main` and GitHub Pages will build automatically.

## Code Style Guidelines

### HTML

**Structure**:
- Use semantic HTML5 elements (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`)
- Proper document structure with `<!DOCTYPE html>`, `<html lang="en">`, `<head>`, and `<body>`
- Include meta tags for charset, viewport, and description

**Formatting**:
- Use 2-space indentation
- Use lowercase for element names and attributes
- Always quote attribute values with double quotes
- Self-closing tags should not have a trailing slash (HTML5 style)
- One blank line between major sections

**Example**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
</head>
<body>
  <header>
    <h1>Site Title</h1>
  </header>
  
  <main>
    <article>
      <h2>Article Title</h2>
      <p>Content goes here.</p>
    </article>
  </main>
</body>
</html>
```

### CSS

**Organization**:
- Use CSS custom properties for theming and reusable values
- Organize styles by component or page section
- Mobile-first responsive design with `min-width` media queries

**Naming**:
- Use kebab-case for class names: `.main-header`, `.nav-item`
- Use BEM methodology for complex components: `.block__element--modifier`
- Prefix utility classes with `u-`: `.u-hidden`, `.u-text-center`

**Formatting**:
- Use 2-space indentation
- One selector per line for multi-selector rules
- One blank line between rule sets
- Properties in logical order: positioning, box model, typography, visual, misc

**Example**:
```css
:root {
  --color-primary: #0066cc;
  --spacing-unit: 8px;
}

.header {
  position: relative;
  padding: var(--spacing-unit);
  background-color: var(--color-primary);
}

.header__title {
  margin: 0;
  font-size: 2rem;
  color: white;
}
```

### JavaScript

**Style**:
- Use modern ES6+ syntax
- Use `const` by default, `let` when reassignment is needed, never `var`
- Use arrow functions for callbacks and short functions
- Use template literals for string interpolation
- Prefer destructuring for objects and arrays

**Naming Conventions**:
- camelCase for variables and functions: `getUserData`, `isActive`
- PascalCase for classes and constructors: `UserProfile`, `ApiClient`
- UPPER_SNAKE_CASE for constants: `MAX_RETRIES`, `API_URL`
- Prefix private properties with underscore: `_privateMethod`

**Formatting**:
- Use 2-space indentation
- Use semicolons
- Single quotes for strings (unless interpolating)
- Trailing commas in multiline objects/arrays

**Functions**:
- Keep functions small and focused (single responsibility)
- Use descriptive names that indicate what the function does
- Document complex functions with JSDoc comments

**Example**:
```javascript
const API_BASE_URL = 'https://api.example.com';

/**
 * Fetches user data from the API
 * @param {string} userId - The user's unique identifier
 * @returns {Promise<Object>} User data object
 */
async function getUserData(userId) {
  const response = await fetch(`${API_BASE_URL}/users/${userId}`);
  
  if (!response.ok) {
    throw new Error(`Failed to fetch user: ${response.statusText}`);
  }
  
  return response.json();
}
```

### Error Handling

**JavaScript**:
- Always handle promise rejections with `.catch()` or `try/catch`
- Throw descriptive errors with context
- Log errors appropriately (don't silence them)
- Validate user input before processing

**Example**:
```javascript
try {
  const data = await fetchData();
  processData(data);
} catch (error) {
  console.error('Failed to process data:', error);
  showUserError('Unable to load content. Please try again.');
}
```

## File Organization

### Recommended Structure
```
/
├── index.html              # Main entry point
├── about.html              # Additional pages
├── css/
│   ├── main.css           # Main stylesheet
│   ├── components.css     # Reusable components
│   └── utilities.css      # Utility classes
├── js/
│   ├── main.js            # Main application logic
│   └── utils.js           # Helper functions
├── assets/
│   ├── images/            # Image files
│   ├── fonts/             # Custom fonts
│   └── icons/             # Icon files
└── tests/                 # Test files (if applicable)
```

## Git Workflow

### Commit Messages
- Use conventional commit format: `type(scope): description`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Keep subject line under 50 characters
- Use imperative mood: "add feature" not "added feature"

**Examples**:
```
feat: add contact form to homepage
fix: correct mobile navigation overflow
docs: update installation instructions
style: format CSS with prettier
```

### Branch Naming
- Feature branches: `feature/descriptive-name`
- Bug fixes: `fix/issue-description`
- Hotfixes: `hotfix/critical-bug`

## Deployment (GitHub Pages)

This site is automatically deployed via **GitHub Pages** from the `main` branch.

### Deployment Workflow

**IMPORTANT**: After completing changes, always deploy to production by pushing to the main branch.

```bash
# Standard workflow after making changes:
# 1. Test changes locally (see "Build & Development Commands")
# 2. Commit changes with descriptive message
git add .
git commit -m "feat: description of changes"

# 3. Push to main branch to trigger automatic deployment
git push origin main
```

### Deployment Guidelines

- **All commits to `main` branch are automatically deployed** to GitHub Pages
- Changes typically appear live within 1-2 minutes after push
- Always test locally before pushing to main
- For major changes, consider using a feature branch first, then merge to main
- Verify deployment at: https://toebes.nl (or configured custom domain)

### GitHub Pages Configuration

- **Source branch**: `main`
- **Source folder**: `/` (root)
- **Custom domain**: toebes.nl (if configured in repository settings)
- **Build process**: Jekyll automatically builds the site on each push

### Deployment Checklist

Before pushing to main:
- [ ] Test all pages locally
- [ ] Validate HTML at https://validator.w3.org/
- [ ] Check responsive design on multiple screen sizes
- [ ] Verify all links work correctly
- [ ] Check browser console for errors
- [ ] Ensure images load properly
- [ ] Test on different browsers (Chrome, Firefox, Safari)

After pushing to main:
- [ ] Wait 1-2 minutes for deployment
- [ ] Visit live site to verify changes
- [ ] Check browser console on production
- [ ] Test critical user flows

## Testing

### Manual Testing Checklist
- [ ] Test in latest Chrome, Firefox, Safari
- [ ] Test on mobile devices (responsive design)
- [ ] Verify accessibility (keyboard navigation, screen readers)
- [ ] Check all links work correctly
- [ ] Validate HTML: https://validator.w3.org/
- [ ] Test page load performance

### Automated Testing
If testing framework is added, follow these practices:
- Write tests for complex JavaScript functions
- Test edge cases and error conditions
- Aim for descriptive test names: `it('should return null when user is not found')`

## Accessibility

- Use semantic HTML elements
- Include `alt` text for all images
- Ensure sufficient color contrast (WCAG AA minimum: 4.5:1)
- Make interactive elements keyboard accessible
- Use ARIA labels when semantic HTML is insufficient
- Test with screen readers

## Performance

- Optimize images (use modern formats: WebP, AVIF)
- Minify CSS and JavaScript for production
- Use lazy loading for images below the fold
- Minimize HTTP requests
- Use CDN for third-party libraries

## Security

- Sanitize user input to prevent XSS attacks
- Use HTTPS for all external resources
- Implement Content Security Policy headers
- Keep dependencies up to date

## Jekyll Blog Guidelines

### File Structure

```
toebes.nl/
├── _config.yml              # Jekyll configuration
├── _layouts/
│   ├── default.html        # Base layout (sidebar + grid)
│   └── post.html           # Blog post layout
├── _posts/                 # Blog posts (Markdown files)
│   └── YYYY-MM-DD-slug.md  # Post naming convention
├── _sass/
│   └── main.scss           # SCSS source files
├── css/
│   └── styles.scss         # Main stylesheet (imports _sass)
├── index.html              # Homepage (shows latest post)
├── 404.html                # Error page
├── favicon.svg             # Site icon (SVG)
├── favicon-*.png           # Site icons (PNG)
├── Gemfile                 # Ruby dependencies
└── .gitignore              # Ignore _site/, .jekyll-cache/
```

### Creating Blog Posts

**Naming Convention**:
- Files must be named: `YYYY-MM-DD-title-slug.md`
- Example: `2026-01-16-welkom-agentic-ai.md`
- Date in filename determines post date (unless overridden in front matter)

**Front Matter** (required at top of every post):
```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD
---
```

**Complete Example**:
```markdown
---
layout: post
title: "Multi-Agent Systemen"
date: 2026-01-17
---

Dit is de content van mijn blog post...

## Subkop

Gebruik gewoon Markdown syntax:

- Lijsten
- **Vet** en *cursief*
- [Links](https://example.com)

```javascript
// Code blocks met syntax highlighting
console.log('Hello world!');
```
```

### Workflow for New Blog Posts

```bash
# 1. Create new post file in _posts/
touch _posts/2026-01-17-my-new-post.md

# 2. Add front matter and write content in Markdown
# (edit in your editor)

# 3. Optional: Test locally
bundle exec jekyll serve
# Visit http://localhost:4000

# 4. Commit and push to main
git add _posts/2026-01-17-my-new-post.md
git commit -m "feat: nieuwe post over [onderwerp]"
git push origin main

# 5. GitHub Pages automatically builds and deploys (1-2 min)
# 6. Verify at https://toebes.nl
```

### Liquid Template Syntax

Jekyll uses **Liquid** for templating:

**Variables**:
```liquid
{{ site.title }}          # Site config from _config.yml
{{ page.title }}          # Current page title
{{ page.date }}           # Current page date
{{ content }}             # Page/post content
```

**Loops**:
```liquid
{% for post in site.posts %}
  <h2>{{ post.title }}</h2>
  <p>{{ post.date | date: "%d %B %Y" }}</p>
{% endfor %}
```

**Conditionals**:
```liquid
{% if page.title %}
  <h1>{{ page.title }}</h1>
{% endif %}
```

**Filters**:
```liquid
{{ page.date | date: "%d %B %Y" }}        # Format date
{{ post.url | relative_url }}             # Make URL relative
{{ page.title | slugify }}                # Convert to slug
{{ content | strip_html }}                # Remove HTML tags
```

**Dutch Date Formatting**:
```liquid
{% assign m = page.date | date: "%-m" %}
{{ page.date | date: "%-d" }}
{% case m %}
  {% when '1' %}januari
  {% when '2' %}februari
  {% when '3' %}maart
  # ... etc
{% endcase %}
{{ page.date | date: "%Y" }}
```

### SCSS/CSS Guidelines

**CSS Variables** (defined in `_sass/main.scss`):
```scss
:root {
  --bg: #0a0a0a;              # Background (almost black)
  --fg: #e0e0e0;              # Foreground text (light grey)
  --accent: #ffffff;          # Accent color (white)
  --border: #333333;          # Borders (dark grey)
  --sidebar-bg: #000000;      # Sidebar (true black)
  --code-bg: #1a1a1a;         # Code block background
  --code-keyword: #cc99cc;    # Syntax: keywords
  --code-string: #99cc99;     # Syntax: strings
  --code-function: #6699cc;   # Syntax: functions
}
```

**Design Principles**:
- **Brutalist aesthetic**: No rounded corners, shadows, or gradients
- **Dark mode**: High contrast for readability
- **System fonts only**: No web fonts loaded
- **Mobile-first**: Responsive design with stacked layout on mobile
- **Minimal**: Only essential styling, nothing decorative

**SCSS Structure**:
- Main styles in `_sass/main.scss`
- Imported via `css/styles.scss` (which has YAML front matter)
- Jekyll compiles SCSS to CSS automatically
- Use nesting, variables, and mixins where appropriate

### Syntax Highlighting

Jekyll uses **Rouge** for syntax highlighting with **base16.dark** color scheme.

**Supported languages**:
- JavaScript, Python, Ruby, Java, C++, C#
- HTML, CSS, SCSS, JSON, YAML, XML
- Bash, Shell, SQL
- And many more

**Usage in posts**:
````markdown
```javascript
function example() {
  return "Hello";
}
```

```python
def example():
    return "Hello"
```
````

### Content Guidelines

**Blog Topic**: Agentic AI - autonomous AI systems that plan, decide, and execute tasks

**Tone**:
- Professional but accessible
- Technical depth where appropriate
- Practical examples and code
- Clear explanations of complex concepts

**Post Structure**:
- Clear title that describes the content
- Introduction explaining the topic
- Logical sections with h2/h3 headings
- Code examples where relevant
- Conclusion or next steps

**Markdown Best Practices**:
- Use h2 (`##`) for main sections, h3 (`###`) for subsections
- Keep paragraphs short (3-5 lines)
- Use lists for multiple related points
- Add code examples to illustrate concepts
- Link to external resources when helpful

### Jekyll Configuration

Key settings in `_config.yml`:

```yaml
title: Jan Toebes Blog
description: Agentic AI - Gedachten en inzichten over autonome AI agents
url: "https://toebes.nl"
baseurl: ""

markdown: kramdown
highlighter: rouge
permalink: /blog/:year/:month/:title/
timezone: Europe/Amsterdam

kramdown:
  syntax_highlighter: rouge
  syntax_highlighter_opts:
    default_lang: javascript
    css_class: 'highlight'
```

**Permalink Structure**:
- Posts are accessible at: `/blog/YYYY/MM/title-slug/`
- Homepage (`/`) shows the most recent post
- Each post has its own page at the permalink URL

### Important Notes

- **No plugins** except GitHub Pages whitelist (jekyll-seo-tag)
- **Generated files** (`_site/`, `.jekyll-cache/`) are git-ignored
- **Dutch date formatting** uses Liquid case/when (no plugin needed)
- **Direct to production** workflow: push to main = live (optional local testing)
- **RSS feed** plugin removed (not required)
- **Favicon** displayed in browser tab and sidebar (48x48px in sidebar)

## Additional Notes

- Always test changes locally before committing (optional but recommended)
- Keep the codebase simple and maintainable
- Comment complex logic, but prefer self-documenting code
- Regularly review and refactor code for improvements
- Consider backwards compatibility for older browsers if required
