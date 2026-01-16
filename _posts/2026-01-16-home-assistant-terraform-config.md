---
layout: post
title: "Home Assistant als Code: Een Terraform-achtige Workflow"
date: 2026-01-16
---

Home Assistant is een krachtig platform voor domotica, maar de standaard workflow heeft zo zijn beperkingen. Aanpassingen maak je vaak via de UI, configuratie staat verspreid over YAML-bestanden en `.storage/` JSON files, en versiebeheer is lastig. Wat als je Home Assistant kon configureren met dezelfde principes als Infrastructure-as-Code tools zoals Terraform?

In deze post laat ik zien hoe ik een volledig git-gedreven workflow heb opgezet voor Home Assistant, compleet met validatie, deployment scripts en AI-assistentie. Alle configuratie - van automaties tot dashboards - zit in version control en wordt gedeployed met één commando.

## Het Probleem met Standaard Home Assistant

Home Assistant slaat configuratie op in twee verschillende systemen:

1. **YAML bestanden** - `automations.yaml`, `scripts.yaml`, `scenes.yaml`
2. **JSON storage** - `.storage/` directory met entity registry, dashboards, helpers

Het probleem? De UI schrijft naar beide locaties, en de `.storage/` files zijn officieel "niet bedoeld voor handmatige bewerking". Dit maakt version control een uitdaging.

## De Oplossing: Git-Driven Configuration

Mijn aanpak: behandel **alles** als code, inclusief de `.storage/` files. De workflow:

```bash
# Voordat je begint: sync wijzigingen van de Raspberry Pi
./deploy.sh sync

# Maak aanpassingen lokaal (via editor of AI)
# ...

# Valideer en deploy
./deploy.sh simple    # Voor normale wijzigingen
./deploy.sh full      # Voor wijzigingen die restart vereisen
```

### De Deploy Script Architectuur

Het hart van de setup is `deploy.sh`, een unified deployment script dat de sync tussen laptop en Raspberry Pi beheert:

```bash
#!/bin/bash
# Configuratie
RPI_HOST="root@homeassistant.t.lan"
RPI_CONFIG_DIR="/config"

# Deploy modes
case "$MODE" in
    sync)
        # Pull changes from RPi to local
        git_sync_rpi      # Commit + push on RPi
        git_pull_local    # Pull to laptop
        ;;
    simple)
        # Deploy with hot reload
        git_sync_rpi
        git_pull_local
        validate_config   # Validation layer
        git_push_local
        git_pull_rpi
        ha_reload_all     # Hot reload via Home Assistant API
        ;;
    full)
        # Deploy with restart
        ha_core_stop      # Stop Home Assistant
        git_sync_rpi
        git_pull_local
        validate_config
        git_push_local
        git_pull_rpi
        ha_core_start     # Start Home Assistant
        ;;
esac
```

**Belangrijke functies**:

- `git_sync_rpi()` - Commit en push onopgeslagen wijzigingen op de RPi
- `validate_config()` - Pre-deployment validatie (zie hieronder)
- `ha_reload_all()` - Hot reload via `hass-cli`

### De Validatie Laag

Een van de krachtigste features is de **pre-deployment validatie**. Voordat configuratie naar de RPi wordt gepusht, worden deze checks uitgevoerd:

```bash
validate_config() {
    # 1. Script references - voorkomen broken automations
    check_script_references
    
    # 2. Helper references - valideren input_boolean/datetime/number
    check_helper_references
    
    # 3. Dashboard entities (warnings only)
    check_dashboard_entities
    
    # 4. Ungrouped items (warnings only)
    check_ungrouped_items
}
```

**Voorbeeld output**:

```
❌ ERROR: Scripts called but not defined:
  script.old_script_name
     → automations.yaml:123: action: script.old_script_name

Continue deployment anyway? (y/N)
```

Dit voorkomt dat broken configuratie naar productie gaat!

### Script Reference Validatie

Automatisch detecteren van missende scripts:

```bash
check_script_references() {
    # Extract script calls: entity_id: script.xxx
    grep -ohE 'entity_id:\s*script\.[a-z0-9_]+' \
        automations.yaml scripts.yaml \
        | cut -d. -f2 > called_scripts.txt
    
    # Extract defined scripts
    grep -oE '^[a-z0-9_]+:' scripts.yaml \
        | sed 's/:$//' > defined_scripts.txt
    
    # Find missing scripts
    comm -23 called_scripts.txt defined_scripts.txt
}
```

### Helper Reference Validatie

Helpers (`input_boolean`, `input_datetime`, etc.) worden opgeslagen in `.storage/` files, maar de entity registry kan afwijken door hernoemen via UI:

```bash
check_helper_references() {
    # Find referenced helpers in YAML
    grep -ohE 'entity_id:\s*input_(boolean|datetime|number)\.[a-z0-9_]+' \
        automations.yaml scripts.yaml > referenced_helpers.txt
    
    # Extract from entity registry
    grep -oE '"entity_id":"input_(boolean|datetime|number)\.[^"]+' \
        .storage/core.entity_registry \
        | cut -d'"' -f4 > registry_helpers.txt
    
    # Find missing
    comm -23 referenced_helpers.txt registry_helpers.txt
}
```

**Waarom dit belangrijk is**: Als je een helper hernoemt via de UI, wordt het ID in `.storage/input_boolean` niet aangepast, maar de mapping in `core.entity_registry` wel. Deze validatie voorkomt broken references.

## File Structure: Infrastructure-as-Code

De repository structure volgt Infrastructure-as-Code principes:

```
homeassistant-config/
├── configuration.yaml          # Main config met includes
├── automations.yaml            # Automation rules
├── scripts.yaml                # Reusable scripts
├── scenes.yaml                 # Lighting scenes
├── .storage/                   # Tracked in git!
│   ├── core.entity_registry    # Entity mappings
│   ├── core.area_registry      # Room definitions
│   ├── input_boolean           # Helper definitions
│   ├── input_datetime
│   ├── lovelace                # Main dashboard
│   └── lovelace.dashboard_*    # Custom dashboards
├── deploy.sh                   # Deployment orchestration
├── .validate_ungrouped.py      # Validation helper
└── AGENTS.md                   # AI agent instructions
```

**Belangrijk**: De `.storage/` directory is **wel** in git, in tegenstelling tot standaard aanbevelingen. Dit maakt volledige version control mogelijk.

## Configuration-as-Code: YAML Patterns

### Automation Structure

Automaties volgen een consistent patroon met categorisatie:

```yaml
- id: '1738183906765'              # Timestamp-based ID
  alias: 'Wekker: papa gordijnen'  # Category: Description
  description: ''
  triggers:
  - at: input_datetime.gordijnen_open_time
    trigger: time
  conditions:
  - condition: state
    entity_id: input_boolean.wekker_actief
    state: 'on'
  actions:
  - parallel:
    - action: cover.set_cover_position
      data:
        position: 90
      target:
        floor_id: zolder
      continue_on_error: true      # Error handling!
    - action: media_player.unjoin
      target:
        entity_id: media_player.sonos_slaapkamer
      continue_on_error: true
  mode: single
```

**Best practices**:

- `continue_on_error: true` voor non-critical actions
- `parallel:` voor onafhankelijke acties
- Dynamische tijden via `input_datetime` helpers
- Categorieën in alias: `Wekker:`, `Gordijn:`, `Alert:`, etc.

### Scripts: Reusable Building Blocks

Scripts fungeren als herbruikbare functies:

```yaml
sonos_group_all:
  sequence:
  - data:
      group_members:
      - media_player.sonos_emma
      - media_player.sonos_keuken
      - media_player.sonos_melle
    action: media_player.join
    target:
      device_id: 81d62bd216086081f4dc50b2e19ec67e
  - action: media_player.volume_set
    data:
      volume_level: 0.4
    target:
      device_id:
      - d6d9cbd37ee21e9b8fe20c4e48943481
      - 81d62bd216086081f4dc50b2e19ec67e
  alias: 'Sonos: groepeer alles'
  description: ''
```

Scripts worden aangeroepen vanuit automaties:

```yaml
actions:
- action: script.sonos_group_all
  continue_on_error: true
```

### Helpers: Dynamic Configuration

Helpers worden gedefinieerd in `.storage/input_boolean` (JSON):

```json
{
  "version": 1,
  "key": "input_boolean",
  "data": {
    "items": [
      {
        "id": "wekker_aan",
        "name": "Wekker: actief",
        "category": "01K9FCWM0025121PPKZS38XY90"
      },
      {
        "id": "gordijnen_open",
        "name": "Gordijn: ochtend",
        "category": "01K9FAV5XQW298AAT2B2ACBJQD"
      }
    ]
  }
}
```

**Categorieën** zorgen voor groepering in de UI. De validation script controleert op uncategorized items:

```python
def find_ungrouped_helpers(file_path, helper_type):
    with open(file_path, 'r') as f:
        data = json.load(f)
    
    ungrouped = []
    for item in data.get('data', {}).get('items', []):
        if item.get('category') is None:
            ungrouped.append({
                'id': item.get('id'),
                'name': item.get('name')
            })
    
    if ungrouped:
        print(f"⚠️  {helper_type} without category:")
        for item in ungrouped:
            print(f"  ⚠️  {helper_type}.{item['id']}")
```

## AI-Assisted Configuration met AGENTS.md

Een cruciale component is `AGENTS.md`: een gedetailleerde instructieset voor AI coding agents. Dit maakt AI-assisted configuratie mogelijk:

```markdown
# AGENTS.md - Home Assistant Configuration Repository

## Automation Structure
- id: timestamp-based unique ID
- alias: 'Category: Description' (colon-prefixed)
- Use 'trigger' key, not 'platform'
- Use 'action' key, not 'service'
- Always add continue_on_error: true for non-critical actions

## Entity ID Naming
# Pattern: domain.location_device_descriptor
light.keuken_tafel_links
media_player.sonos_slaapkamer
cover.gordijnen_group_alles
```

Met deze instructies kan een AI agent:

- Nieuwe automaties toevoegen met correct formaat
- Scripts creëren volgens naming conventions
- Validatie draaien voor deployment
- Git workflow uitvoeren

**Voorbeeld interactie**:

```
User: "Voeg een automatisering toe die 's avonds om 22:00 
       alle lampen uitdoet als ik thuis ben"

AI Agent:
1. Leest AGENTS.md voor conventions
2. Genereert automation met juiste structure
3. Voegt toe aan automations.yaml
4. Draait ./deploy.sh validate
5. Bij succes: ./deploy.sh simple
```

## Dashboard Configuration-as-Code

Dashboards worden opgeslagen in `.storage/lovelace*` files. Convention: gebruik `auto-entities` voor dynamische lijsten:

```json
{
  "type": "custom:auto-entities",
  "card": {
    "type": "entities",
    "title": "Batterijen Laag",
    "state_color": true
  },
  "filter": {
    "include": [
      {
        "integration": "hue",
        "attributes": {
          "device_class": "battery"
        }
      }
    ]
  },
  "sort": {
    "method": "state",
    "numeric": true
  },
  "show_empty": false
}
```

**Voordelen**:

- Nieuwe devices worden automatisch toegevoegd
- Geen handmatige maintenance bij toevoegen/verwijderen
- Consistente filtering en sorting

## Entity Registry: De Waarheid

Een veelvoorkomend probleem: je hernoemt een entity via de UI, maar in de YAML staat nog de oude naam. Hoe kan dit?

De `core.entity_registry` bevat de mapping:

```json
{
  "entity_id": "input_boolean.wekker_actief",
  "unique_id": "wekker_aan",
  "original_name": "Wekker: actief",
  "name": null
}
```

- `unique_id` - De ID in `.storage/input_boolean`
- `entity_id` - De daadwerkelijke entity ID in Home Assistant
- `name` - UI override (null = gebruik original_name)

**Debugging tip**: Als een entity "niet bestaat", check eerst de entity registry:

```bash
grep -A10 '"unique_id":"wekker_aan"' .storage/core.entity_registry
```

## Pre-Commit Hooks: Validation op Steroïden

Voor extra veiligheid kun je git pre-commit hooks toevoegen:

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Validate YAML syntax
yamllint automations.yaml scripts.yaml scenes.yaml || exit 1

# Run configuration validation
./deploy.sh validate || exit 1

# Check for secrets
if git diff --cached | grep -i "password\|token\|api_key"; then
    echo "ERROR: Possible secret detected!"
    exit 1
fi
```

## Terraform-Style Apply Workflow

De volledige workflow lijkt op Terraform:

```bash
# 1. Plan: Sync remote state
./deploy.sh sync

# 2. Edit: Make changes locally
vim automations.yaml

# 3. Validate: Check configuration
./deploy.sh validate

# 4. Apply: Deploy to production
./deploy.sh simple
```

**Output tijdens deployment**:

```
==> Syncing changes from RPi...
==> RPi has no uncommitted changes
==> Pulling changes to local...
==> Local pull complete
==> Validating configuration...
==> Configuration validation passed
==> Pushing local changes...
==> Committing local changes...
==> Local changes pushed
==> Pulling changes on RPi...
==> RPi pull complete
==> Reloading Home Assistant configuration...
==> Home Assistant configuration reloaded
==> SIMPLE deploy complete!
```

## Conflict Handling: Merge Strategies

Omdat zowel lokaal als op de RPi wijzigingen gemaakt kunnen worden, zijn merge conflicts mogelijk:

```bash
check_conflicts() {
    # Check for unmerged paths
    if git ls-files -u | grep -q .; then
        return 1
    fi
    # Check for conflict markers
    if git diff --name-only | xargs grep -l "<<<<<<\|>>>>>>"; then
        return 1
    fi
    return 0
}
```

**Bij conflicts**:

```
ERROR: Merge conflicts detected!

Conflicting files:
  .storage/core.entity_registry

To resolve:
  1. Edit the conflicting files
  2. Run: git add <resolved-files>
  3. Re-run: ./deploy.sh simple
```

## Testing: Template Editor & Developer Tools

Home Assistant heeft ingebouwde test tools:

**Template Editor** (Developer Tools > Template):

```jinja
# Test Jinja2 templates
{{ states('input_boolean.wekker_actief') }}
{{ states.light | selectattr('state', 'eq', 'on') | list | count }}
```

**Service Calls** (Developer Tools > Actions):

```yaml
action: script.sonos_group_all
data: {}
```

Deze kunnen geïntegreerd worden in test scripts:

```bash
# Test automation triggers
hass-cli event fire test_event --data '{"entity_id": "light.test"}'

# Check if entity exists
hass-cli state get input_boolean.wekker_actief
```

## Remote Sync: Cron op de RPi

Op de Raspberry Pi draait een cron job die automatisch sync:

```bash
# /config/scripts/git_sync.sh (op RPi)
#!/bin/bash
cd /config
git add .
git commit -m "Auto-sync from RPi: $(date)"
git push origin main
```

**Cron entry**:

```cron
# Sync every hour
0 * * * * /config/scripts/git_sync.sh
```

Dit zorgt ervoor dat UI wijzigingen op de RPi automatisch naar git gaan.

## Security: Secrets Management

Gevoelige data gaat **nooit** in git:

```yaml
# .gitignore
secrets.yaml
.env.secrets
*.key
*.pem
```

**Secrets bestand** (`.env.secrets`):

```bash
export HASS_SERVER=http://homeassistant.t.lan:8123
export HASS_TOKEN=eyJ0eXAiOiJKV1QiLCJ...
```

**Usage in deploy script**:

```bash
load_secrets() {
    if [[ -f "$SECRETS_FILE" ]]; then
        source "$SECRETS_FILE"
        return 0
    fi
}

ha_reload_all() {
    check_secrets
    hass-cli service call homeassistant.reload_all
}
```

## Voordelen van deze Aanpak

**Version Control**:
- Volledige history van alle wijzigingen
- Rollback naar eerdere versies
- Branching voor experimenten

**Validatie**:
- Pre-deployment checks voorkomen errors
- Broken references worden gevangen
- Consistent formatting

**Reproduceerbaar**:
- Configuratie is portable
- Disaster recovery: clone repo en deploy
- Documentatie in code zelf

**AI-Assisted**:
- Agents kunnen veilig wijzigingen maken
- Validation voorkomt fouten
- Snellere iteratie

**Testing**:
- Test lokaal voor deployment
- Feature branches voor grote wijzigingen
- Automated validation

## Nadelen en Trade-offs

**Complexity**:
- Meer setup dan standaard workflow
- Git kennis vereist
- Deploy script onderhoud

**Storage Files**:
- Officieel "not for editing"
- Mogelijk breaking changes in HA updates
- Merge conflicts kunnen complex zijn

**Sync Delays**:
- UI wijzigingen niet instant in git
- Cron interval (1 uur in mijn setup)
- Conflicts bij gelijktijdige edits

## Conclusie

Door Home Assistant te behandelen als Infrastructure-as-Code krijg je:

- **Version control** voor alle configuratie
- **Pre-deployment validatie** die errors voorkomt  
- **Git-driven workflow** met sync tussen devices
- **AI-assistentie** via gestructureerde instructies
- **Reproduceerbare** configuratie

De setup vereist initiële investering, maar betaalt zich terug in:
- Minder debugging tijd
- Snellere iteratie met AI
- Veiligere deployments
- Betere documentatie

Voor teams of mensen die Infrastructure-as-Code gewend zijn, voelt deze workflow natuurlijk. En met AI agents die de AGENTS.md kunnen lezen, wordt configuratie wijzigen zo simpel als: "voeg een automatisering toe voor X" - de agent regelt formatting, validatie en deployment.

**Volgende stappen**:
- CI/CD pipeline met GitHub Actions
- Automated testing met HA test fixtures
- Multi-environment setup (test/prod)
- Metrics en monitoring van deployments

De volledige code is beschikbaar als referentie voor wie een vergelijkbare setup wil bouwen. Happy automating!
