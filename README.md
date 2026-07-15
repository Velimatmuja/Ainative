# WordPress Auto-Post me AI (n8n)

Workflow n8n që **automatizon publikimin e postimeve në WordPress** me artikuj të shkruar nga AI (Claude). Çdo ditë zgjedh një kategori tjetër (Ekonomi, Kredi, Banka, Investime, Financa Personale, Sigurime), gjeneron një artikull **maksimumi 500 fjalë** në shqip dhe e publikon automatikisht në kategorinë përkatëse.

## Si funksionon

```
Çdo ditë 09:00 → Konfigurimi → Zgjidh Kategorinë → Gjenero Artikullin (Claude)
   → Përpuno Përgjigjen → Kërko Kategorinë në WP → Ekziston Kategoria?
        ├── Po  → Posto në WordPress
        └── Jo  → Krijo Kategorinë → Posto në WordPress
```

- **Rrotullimi i kategorive:** çdo ditë përdoret kategoria e radhës nga lista; temat brenda kategorisë rrotullohen gjithashtu, që artikujt të mos përsëriten.
- **Kufiri 500 fjalë:** i kërkohet AI-së në prompt **dhe** verifikohet në kod — nëse artikulli del më i gjatë, shkurtohet automatikisht paragraf pas paragrafi.
- **Kategoritë në WordPress:** nëse kategoria nuk ekziston në sajt, krijohet automatikisht para publikimit.

## Instalimi

### 1. Importo workflow-in

Në n8n: **Workflows → Import from File** → zgjidh `workflows/wordpress-ai-auto-post.json`.

### 2. Krijo kredencialet

**a) Anthropic API Key** (për gjenerimin e artikujve)

1. Merr një API key nga [platform.claude.com](https://platform.claude.com)
2. Në n8n: **Credentials → New → Header Auth**
   - **Name (header):** `x-api-key`
   - **Value:** API key-i yt (`sk-ant-...`)
3. Lidhe kredencialin te nyja **"Gjenero Artikullin (Claude)"**

**b) WordPress Application Password** (për publikimin)

1. Në WordPress: **Users → Profile → Application Passwords** → krijo një fjalëkalim të ri aplikacioni (kërkon WordPress 5.6+ dhe HTTPS)
2. Në n8n: **Credentials → New → Basic Auth**
   - **User:** emri i përdoruesit në WordPress (duhet të ketë të drejta `editor` ose `administrator`)
   - **Password:** application password-i i krijuar (me hapësirat siç shfaqet)
3. Lidhe kredencialin te nyjat **"Kërko Kategorinë në WP"**, **"Krijo Kategorinë"** dhe **"Posto në WordPress"**

### 3. Konfiguro sajtin

Hap nyjen **"Konfigurimi"** dhe vendos:

| Fusha | Vlera | Shembull |
|---|---|---|
| `wpUrl` | URL e sajtit WordPress (pa `/` në fund) | `https://portali-im.com` |
| `postStatus` | `publish` (publikim direkt) ose `draft` (për rishikim para publikimit) | `publish` |

> 💡 **Këshillë:** fillo me `draft` derisa të jesh i kënaqur me cilësinë e artikujve, pastaj kalo në `publish`.

### 4. Personalizo kategoritë dhe temat (opsionale)

Hap nyjen **"Zgjidh Kategorinë"** — aty është lista `categories` me kategoritë dhe temat. Shto, hiq ose ndrysho sipas nevojës:

```js
{
  name: 'Kredi',
  topics: [
    'Si të merrni një kredi hipotekare: hapat kryesorë',
    'Kredi konsumatore: çfarë duhet të dini para se të aplikoni',
    // shto tema të tjera këtu...
  ]
}
```

### 5. Ndrysho orarin (opsionale)

Nyja **"Çdo ditë në 09:00"** përdor cron `0 9 * * *`. Shembuj të tjerë:

| Cron | Kuptimi |
|---|---|
| `0 9 * * *` | Çdo ditë në 09:00 |
| `0 9,15 * * *` | Dy herë në ditë (09:00 dhe 15:00) |
| `0 9 * * 1-5` | Vetëm ditët e punës në 09:00 |

### 6. Aktivizo workflow-in

Testoje fillimisht me **"Execute Workflow"** (ekzekutim manual), kontrollo artikullin e krijuar në WordPress, pastaj aktivizoje me çelësin **Active**.

## Detaje teknike

- **Modeli AI:** `claude-opus-4-8` (Anthropic Messages API). Përdor *structured outputs* (`output_config.format` me JSON Schema), kështu që përgjigja është gjithmonë JSON valid me `title`, `content_html` dhe `excerpt` — pa nevojë për parsing të brishtë.
- **Formati i artikullit:** HTML i pastër (`<h2>`, `<h3>`, `<p>`, `<ul>`, `<li>`, `<strong>`) — gati për editorin e WordPress.
- **Trajtimi i gabimeve:** nëse Claude e refuzon kërkesën ose përgjigja pritet nga `max_tokens`, workflow-i ndalon me mesazh të qartë gabimi në shqip.
- **Kostoja:** një artikull ~500 fjalë kushton pak centë për ekzekutim (varet nga çmimet aktuale të modelit).

## Zgjidhja e problemeve

| Problemi | Zgjidhja |
|---|---|
| `401 Unauthorized` nga Anthropic | Kontrollo që header-i i kredencialit është `x-api-key` dhe key-i është valid |
| `401` nga WordPress | Kontrollo application password-in dhe që sajti ka HTTPS |
| `rest_cannot_create` nga WordPress | Përdoruesi nuk ka të drejta të mjaftueshme — përdor llogari `editor`/`administrator` |
| Artikulli s'shfaqet | Kontrollo `postStatus` — me `draft` shfaqet vetëm te Posts → Drafts |
| Kategoria krijohet dyfish | Emri në listën e kategorive duhet të përputhet saktësisht me emrin në WordPress (pa hapësira shtesë) |
