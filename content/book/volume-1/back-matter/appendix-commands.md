---
title: "Appendix: Commands and Configuration Reference"
weight: 100
draft: false
---

Every command, Docker Compose block, and config snippet from the book, gathered in one place by chapter, so you don't have to hunt back through the text to find one you remember reading.


## Chapter 4: Setting Up Lemonade

*Installing Lemonade Server*

```powershell
winget install --id AMD.LemonadeServer --exact
```

```powershell
lemonade-server status
```

*Starting the Server*

```powershell
lemonade-server serve --port 13305
```

*Keeping It Running: What Actually Starts Lemonade*

```powershell
Get-Process | Where-Object {$_.ProcessName -like "*lemonade*"} | Select-Object ProcessName, Path, Id
```

*Loading Local Models*

```powershell
lemonade-server pull <model-name>
```

*Testing the Server Responds*

```
http://localhost:13305/app
```

```powershell
curl http://localhost:13305/v1/models
```


## Chapter 5: Model Aliasing

*The Duplicate-and-Rename Approach*

```powershell
copy "C:\localllms\qwen3-coder-next-Q4_K_M.gguf" "C:\localllms\claude-3-5-sonnet.gguf"
```

```powershell
curl http://localhost:13305/v1/models
```

*Setting Context Size Per Tier*

```powershell
Invoke-RestMethod http://localhost:13305/v1/models | ConvertTo-Json -Depth 5
```


## Chapter 6: Wiring the Gateway Config

*What I Actually Run: Direct, No Middleman*

```powershell
setx ANTHROPIC_BASE_URL "http://localhost:13305"
setx ANTHROPIC_API_KEY "local-lemonade-no-key-needed"
```

*Verifying the Direct Connection Works*

```powershell
curl http://localhost:13305/v1/messages `
  -H "x-api-key: local-lemonade-no-key-needed" `
  -H "anthropic-version: 2023-06-01" `
  -H "content-type: application/json" `
  -d '{\"model\": \"claude-3-5-sonnet-20241022\", \"max_tokens\": 100, \"messages\": [{\"role\": \"user\", \"content\": \"Say hello in exactly five words.\"}]}'
```


## Chapter 8: Open WebUI + Vane/Perplexica Setup

*Installing Open WebUI*

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    environment:
      - OPENAI_API_BASE_URL=http://host.docker.internal:13305/v1
      - OPENAI_API_KEY=local-lemonade-no-key-needed
    volumes:
      - open-webui-data:/app/backend/data
    restart: unless-stopped

volumes:
  open-webui-data:
```

```powershell
docker compose up -d
```

*Adding Vane*

```yaml
  vane:
    image: vane/vane:slim
    container_name: vane
    ports:
      - "3030:3000"
    environment:
      - SEARXNG_API_URL=http://host.docker.internal:8080
    depends_on:
      - open-webui
    restart: unless-stopped
```

```powershell
docker compose ps
```


## Chapter 9: SearXNG as a Search Replacement

*Setting Up SearXNG*

```yaml
  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    ports:
      - "8080:8080"
    volumes:
      - ./searxng:/etc/searxng
    environment:
      - SEARXNG_BASE_URL=http://localhost:8080/
    restart: unless-stopped
```

```powershell
docker compose up -d
```

```
http://localhost:8080
```

*The Limiter: A Setting That Will Silently Break Everything*

```yaml
server:
  limiter: false
```

```powershell
docker compose restart searxng
```

*Wiring the Connection to Vane*

```powershell
docker compose restart vane
```


## Chapter 10: Pipelines

*Setting Up Pipelines*

```yaml
  pipelines:
    image: ghcr.io/open-webui/pipelines:main
    container_name: pipelines
    ports:
      - "9099:9099"
    volumes:
      - ./pipelines:/app/pipelines
    restart: unless-stopped
```

*A Worked Example: Redact, Don't Block*

```python
import re
from pydantic import BaseModel

class Pipeline:
    class Valves(BaseModel):
        pass

    def __init__(self):
        self.name = "Sensitive Data Redactor"
        self.email_pattern = re.compile(r'[\w\.-]+@[\w\.-]+\.\w+')
        # Extend with your own patterns or a name/company lookup list

    async def inlet(self, body: dict, user: dict) -> dict:
        last_message = body['messages'][-1]['content']
        redacted = self.email_pattern.sub('[REDACTED EMAIL]', last_message)
        # Apply additional redaction rules here — names, company identifiers, etc.
        body['messages'][-1]['content'] = redacted
        return body
```


## Chapter 11: nginx and Routing

*The Basic Reverse Proxy Config*

```yaml
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - open-webui
      - vane
      - searxng
    restart: unless-stopped
```

```nginx
events {}

http {
    server {
        listen 80;

        location / {
            proxy_pass http://open-webui:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        location /search/ {
            proxy_pass http://searxng:8080/;
            proxy_set_header Host $host;
        }
    }
}
```


## Chapter 12: Remote Access with Tailscale Serve

*Installing Tailscale*

```powershell
winget install tailscale.tailscale
tailscale up
```

```powershell
tailscale status
```

*Tailscale Serve, Specifically*

```powershell
tailscale serve --https=443 http://localhost:13305
```


## Chapter 13: Staying Updated

*Watchtower, and Why Not the Obvious Image*

```yaml
  watchtower:
    image: nickfedor/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_POLL_INTERVAL=86400
    restart: unless-stopped
```

*Update Hygiene: What Should Auto-Update, and What Shouldn't*

```yaml
  lemonade:
    labels:
      - "com.centurylinklabs.watchtower.enable=false"
```

*Rollback Strategy*

```powershell
docker compose pull <service> --quiet
docker compose up -d <service>
```


## Chapter 14: Backups with Kopia

*What's Actually Running*

```powershell
kopia repository create filesystem --path=D:\kopia-backups
kopia policy set --global --keep-latest=10
```

```powershell
kopia snapshot create C:\path\to\your\stack
```

*The Retention Window Problem*

```powershell
kopia policy set --global --keep-hourly=24 --keep-daily=14 --keep-weekly=8 --keep-monthly=6
```

*The Test I'm About to Go Run*

```powershell
kopia snapshot restore <snapshot-id> D:\restore-test
```


## Chapter 15b: The Ingestion Gap I Missed

*What Actually Runs*

```python
# pdf_to_vault.py — one format, one job
import sys
from pathlib import Path
import pdfplumber

def extract(source_path: Path) -> str:
    with pdfplumber.open(source_path) as pdf:
        return "\n\n".join(page.extract_text() or "" for page in pdf.pages)

def write_note(source_path: Path, text: str, vault_dir: Path):
    note_path = vault_dir / f"{source_path.stem}.md"
    note_path.write_text(
        f"# {source_path.stem}\n\nSource: {source_path.name}\n\n{text}",
        encoding="utf-8"
    )

if __name__ == "__main__":
    source = Path(sys.argv[1])
    vault_dir = Path(r"D:\corporate_brain\vault")  # or your own vault path
    text = extract(source)
    write_note(source, text, vault_dir)
```

```python
# xlsx_to_vault.py — Excel's extract() is a genuine exception, not a raw dump
import openpyxl

def extract(source_path: Path) -> str:
    wb = openpyxl.load_workbook(source_path, data_only=True)
    summary = []
    for sheet in wb.worksheets:
        headers = [cell.value for cell in sheet[1]]
        summary.append(
            f"## {sheet.title}\n"
            f"Rows: {sheet.max_row}, Columns: {sheet.max_column}\n"
            f"Headers: {headers}\n"
        )
    return "\n".join(summary)
```


## Chapter 17: The Vault That Feeds Itself

*A Worked Example: The ArXiv Script*

```powershell
$Keywords = @("quantization", "mixture of experts", "efficient inference")
$SeenFile = "C:\scripts\state\arxiv-seen.json"

$seen = @{}
if (Test-Path $SeenFile) {
    $seenData = Get-Content $SeenFile -Raw | ConvertFrom-Json
    foreach ($item in $seenData) { $seen[$item.Id] = $item.DateAdded }
}

$newPapers = @($allPapers | Where-Object { -not $seen.ContainsKey($_.AbsUrl) })

if ($newPapers.Count -eq 0) {
    Write-Log "Nothing new. Exiting."
    exit 0
}

# ... write $newPapers to the vault, then update $seen and save it back
```

*Task Scheduler as the Backbone*

```powershell
$action  = New-ScheduledTaskAction -Execute "powershell.exe" `
             -Argument "-NoProfile -ExecutionPolicy Bypass -File `"$scriptPath`""
$trigger = New-ScheduledTaskTrigger -Daily -At $time
$settings = New-ScheduledTaskSettingsSet -StartWhenAvailable `
              -RestartCount 2 -RestartInterval (New-TimeSpan -Minutes 5)

Register-ScheduledTask -TaskName $name -Action $action -Trigger $trigger `
    -Settings $settings -User $env:USERNAME
```
