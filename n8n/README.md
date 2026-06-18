# Winndia WhatsApp Bot - n8n MVP

Bot que escucha el grupo de WhatsApp "REMITOS", extrae datos de remitos / pagos / cheques con Claude vision, y los carga en el Google Sheet de cuentas corrientes.

## Arquitectura

```
WhatsApp grupo REMITOS
        ↓ (webhook)
Evolution API
        ↓ (POST)
n8n Cloud (este workflow)
        ↓
   ├── Claude API (visión) → extrae JSON estructurado
   ├── Google Sheets API → escribe en hoja del cliente
   └── Evolution API (sendText) → confirma o pregunta al grupo
```

## Cómo importar el workflow

1. En n8n Cloud, abrí el panel de Workflows.
2. Botón `+ Add workflow` (arriba a la derecha) → menú `⋮` → `Import from File`.
3. Seleccioná `winndia-bot-workflow.json`.

## Credenciales que necesitás crear en n8n

### 1. Anthropic API (HTTP Header Auth)
- Name: `Anthropic API`
- Header name: `x-api-key`
- Header value: tu API key de [console.anthropic.com](https://console.anthropic.com).

### 2. Evolution API (HTTP Header Auth)
- Name: `Evolution API`
- Header name: `apikey`
- Header value: la API key de tu instancia de Evolution API.

### 3. Google Sheets OAuth2
- Buscá la credencial `Google Sheets OAuth2 API`, hacé el flow de OAuth con tu cuenta `nahuelsconfianza@gmail.com`.

## Configuración del workflow

Abrí el nodo **Config** (Code node) y editá:

- `groupJid`: JID del grupo REMITOS (formato `xxxxxxxxxxxxxxxx@g.us`). Se obtiene desde el panel de Evolution API → Groups, o mandando un mensaje al grupo y mirando el webhook que llega.
- `spreadsheetId`: ya viene precargado con tu sheet (`1dLd0fgqVrDLliq4sCdVxMtXdavWuREkEeZcODDk-t8Y`).
- `evolutionApiUrl` y `evolutionInstance`: ajustar a tu instancia.
- `clientAliases`: agregá todas las formas en que cada cliente puede aparecer escrito en remitos.

## Configuración en Evolution API

En el panel de tu instancia, configurá el webhook:

- URL: la URL del nodo Webhook de n8n (la copiás del nodo `Webhook (Evolution API)` después de activar el workflow). Va a ser algo como `https://<tu-n8n>.app.n8n.cloud/webhook/winndia-bot`.
- Eventos a escuchar: `MESSAGES_UPSERT` (suficiente para el MVP).

## Estructura del Google Sheet (referencia)

Cada hoja de cliente tiene dos tablas:

| Col | A | B | C | D | E | F | G | H | I | J | K | L | M |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Header | cta 2 | COMPROBANTE | NOTA | DEBE | HABER | SALDO | *(vacía)* | cta 1 | COMPROBANTE | NOTA | DEBE | HABER | SALDO |

- Datos arrancan en fila 2.
- El bot escribe hasta la columna E (cta 2) o L (cta 1), nunca toca SALDO.

## TODOs para la v2

- Branch para escribir también en la hoja `cheques` cuando es un cheque/echeq.
- Procesar respuestas de texto del grupo y correlacionarlas con preguntas previas del bot.
- Fuzzy matching mejor de clientes (`fuse.js` o similar).
- Detección de cuenta (1 vs 2) para pagos sin contexto: usar histórico del cliente.
- Anti-duplicados: chequear `numero_comprobante` antes de escribir.

## Testing

Antes de activar:

1. Probá el workflow en **Test mode** mandando una imagen al grupo.
2. Mirá la ejecución en el panel de n8n y verificá que cada nodo devuelva los datos esperados.
3. Si Claude devuelve mal el JSON, ajustá el prompt en el nodo `Claude Vision`.
4. Si no encuentra el cliente, agregá el alias en `Config`.

Una vez funcionando, **activá el workflow** con el switch arriba a la derecha.
