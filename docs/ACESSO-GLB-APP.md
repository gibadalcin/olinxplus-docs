# 🔐 Como o App Mobile Acessa os GLBs

## TL;DR
**✅ SIM, o app consegue acessar GLBs em `{userId}/ra/models/`!**

O backend gera **Signed URLs** (URLs assinadas) que permitem acesso público temporário aos GLBs privados no GCS.

---

## 🔄 Fluxo Completo (Upload → App)

### 1. **AdminUI faz upload de imagem**
```http
POST /api/add-content_image
Content-Type: multipart/form-data

file: totem_header.jpg
```

### 2. **Backend processa** (automático)
```python
# 1. Salvar imagem
gs://olinxra-conteudo/TR77xSOJ.../totem_header.jpg

# 2. Gerar GLB
gs://olinxra-conteudo/TR77xSOJ.../ra/models/totem_header.glb

# 3. Gerar Signed URL (válida por 7 dias padrão, 365 dias para assets principais)
https://storage.googleapis.com/olinxra-conteudo/TR77xSOJ.../ra/models/totem_header.glb?
  X-Goog-Algorithm=GOOG4-RSA-SHA256&
  X-Goog-Credential=...&
  X-Goog-Date=20251210T...&
  X-Goog-Expires=604800&  ← 7 dias (padrão) ou 31536000 (1 ano para assets críticos)
  X-Goog-Signature=...
```

### 3. **Backend retorna**
```json
{
  "success": true,
  "bloco": {
    "tipo": "imagem",
    "url": "gs://olinxra-conteudo/TR77xSOJ.../totem_header.jpg",
    "glb_url": "gs://olinxra-conteudo/TR77xSOJ.../ra/models/totem_header.glb",
    "glb_signed_url": "https://storage.googleapis.com/...?X-Goog-Expires=31536000..."
  }
}
```

### 4. **AdminUI salva no MongoDB**
```javascript
db.conteudos.insertOne({
  owner_uid: "TR77xSOJ...",
  blocos: [
    {
      tipo: "imagem",
      url: "gs://...",
      glb_url: "gs://olinxra-conteudo/TR77xSOJ.../ra/models/totem_header.glb",
      glb_signed_url: "https://storage.googleapis.com/...?X-Goog-Expires=31536000..."
    }
  ]
})
```

### 5. **App Mobile busca payload**
```typescript
// App scaneia QR Code → obtém payload ID
const response = await fetch(`${API_URL}/api/payload/${payloadId}`);
const payload = await response.json();

// Extrai GLBs
const glbUrls = payload.blocos
  .filter(b => b.tipo === 'imagem')
  .map(b => b.glb_signed_url)
  .filter(Boolean);

console.log('GLBs encontrados:', glbUrls);
// ["https://storage.googleapis.com/...?X-Goog-Expires=31536000..."]
```

### 6. **App carrega GLB em AR**
```typescript
import { ARLauncher } from '@/components/ar';

// Usar signed URL para carregar modelo
const glbUrl = payload.blocos[0].glb_signed_url;

// Launcher AR nativo (Expo GL + React Three Fiber)
// Suporte a ARCore (Android) e ARKit (iOS)
<ARLauncher modelUrl={glbUrl} />
```

**Repositórios:**
- Mobile: [github.com/gibadalcin/olinxplus](https://github.com/gibadalcin/olinxplus)
- Backend: [github.com/gibadalcin/olinxplus-backend](https://github.com/gibadalcin/olinxplus-backend)
- Admin: [github.com/gibadalcin/olinxplus-adminui](https://github.com/gibadalcin/olinxplus-adminui)
```

---

## 🔐 Como Funcionam Signed URLs

### O que é uma Signed URL?

Uma **URL temporária** que permite acesso público a um arquivo **privado** no GCS, **sem precisar de autenticação**.

**Analogia:**
- GCS = cofre trancado
- Arquivo GLB = documento dentro do cofre
- Signed URL = chave temporária que abre o cofre (expira depois de X dias)

### Componentes da Signed URL
```
https://storage.googleapis.com/olinxra-conteudo/TR77xSOJ.../ra/models/totem_header.glb?
  ├─ X-Goog-Algorithm=GOOG4-RSA-SHA256   ← Algoritmo de assinatura
  ├─ X-Goog-Credential=service@...       ← Credencial do backend
  ├─ X-Goog-Date=20251103T120000Z        ← Data de criação
  ├─ X-Goog-Expires=31536000             ← Tempo de validade (segundos)
  └─ X-Goog-Signature=abc123...          ← Assinatura criptográfica
```

**Benefícios:**
- ✅ App **não precisa** de credenciais GCS
- ✅ App **não precisa** de token Firebase
- ✅ URL funciona em **qualquer dispositivo**
- ✅ Pode ser compartilhada (WhatsApp, email)
- ✅ Expira automaticamente (segurança)

---

## ⏰ Expiração das Signed URLs

### Tempos de Expiração

| Tipo | Expiração | Motivo |
|------|-----------|---------|
| **Imagens** | 1 hora (3600s) | Preview rápido no AdminUI |
| **GLBs** | **365 dias** (31536000s) | App mobile usa por muito tempo |

### Por quê 365 dias para GLBs?

**Problema com expiração curta (7 dias):**
```
Dia 1: Admin cria conteúdo → GLB gerado → signed URL válida
Dia 8: App tenta carregar GLB → signed URL expirada → ❌ 403 Forbidden
```

**Solução com 365 dias:**
```
Dia 1: Admin cria conteúdo → GLB gerado → signed URL válida por 1 ano
Dia 8, 30, 100, 300: App carrega GLB → ✅ Funciona normalmente
Dia 366: Signed URL expira → App regenera automaticamente
```

**Vantagens:**
- ✅ App não precisa pedir nova signed URL toda hora
- ✅ GLBs são arquivos **estáticos** (raramente mudam)
- ✅ Performance melhor (menos requests)
- ✅ Funciona offline (após 1º download)

---

## 🔄 Renovação Automática (Futuro)

### Cenário: Signed URL expirou

**Opção A - Regenerar on-demand (App):**
```typescript
// App detecta que signed URL expirou (403)
async function loadGLB(glbUrl: string) {
  try {
    const response = await fetch(glbUrl);
    if (response.status === 403) {
      // Signed URL expirou, pedir nova
      const freshUrl = await regenerateSignedUrl(glbUrl);
      return freshUrl;
    }
    return glbUrl;
  } catch (e) {
    console.error('Erro ao carregar GLB:', e);
  }
}

async function regenerateSignedUrl(gsUrl: string) {
  // Pedir nova signed URL ao backend
  const response = await fetch(
    `${API_URL}/api/conteudo-signed-url?gs_url=${encodeURIComponent(gsUrl)}`
  );
  const { signed_url } = await response.json();
  return signed_url;
}
```

**Opção B - Cron job (Backend):**
```python
# Rodar diariamente (cron)
@app.post("/api/admin/refresh-expired-signed-urls")
async def refresh_expired_signed_urls():
    """
    Atualiza signed URLs que expiram em < 7 dias.
    """
    # Buscar documentos com GLBs
    docs = await db.conteudos.find({
        "blocos.glb_signed_url": {"$exists": True}
    }).to_list(None)
    
    updated = 0
    for doc in docs:
        for bloco in doc.get("blocos", []):
            if bloco.get("glb_url"):
                # Gerar nova signed URL (365 dias)
                new_signed_url = gerar_signed_url_conteudo(
                    bloco["glb_url"],
                    expiration=365*24*60*60
                )
                
                # Atualizar MongoDB
                await db.conteudos.update_one(
                    {"_id": doc["_id"]},
                    {"$set": {"blocos.$.glb_signed_url": new_signed_url}}
                )
                updated += 1
    
    return {"updated": updated}
```

---

## 🛡️ Segurança

### Isolamento de GLBs

**Cada admin só acessa seus GLBs:**
```
Admin A (TR77xSOJ...):
  gs://olinxra-conteudo/TR77xSOJ.../ra/models/totem.glb
  └─ Signed URL válida para esse arquivo

Admin B (yiF2ZJyB...):
  gs://olinxra-conteudo/yiF2ZJyB.../ra/models/banner.glb
  └─ Signed URL DIFERENTE (não acessa GLB do Admin A)
```

**Por quê é seguro?**
- ✅ Signed URL contém **assinatura criptográfica** única
- ✅ Assinatura valida **exatamente** o path do arquivo
- ✅ Modificar URL = assinatura inválida = 403 Forbidden
- ✅ Expiração automática (1 ano)

**Exemplo de tentativa de hack:**
```bash
# URL original (válida)
https://storage.googleapis.com/.../TR77xSOJ.../ra/models/totem.glb?X-Goog-Signature=abc123

# Tentativa de acessar GLB de outro admin (FALHA)
https://storage.googleapis.com/.../yiF2ZJyB.../ra/models/banner.glb?X-Goog-Signature=abc123
                                 └─ Path diferente ─┘
                                                      └─ Assinatura INVÁLIDA ❌

# Resultado: 403 Forbidden
```

---

## 📊 Testes

### Teste 1: Verificar signed URL válida

**Script Python:**
```python
import urllib.parse
import requests

# URL do MongoDB
glb_signed_url = "https://storage.googleapis.com/..."

# Parsear query params
parsed = urllib.parse.urlparse(glb_signed_url)
params = urllib.parse.parse_qs(parsed.query)

# Ver expiração
expires = int(params['X-Goog-Expires'][0])
print(f"Expiração: {expires}s = {expires/(24*60*60):.1f} dias")

# Testar acesso
response = requests.head(glb_signed_url)
print(f"Status: {response.status_code}")
print(f"Content-Type: {response.headers.get('Content-Type')}")
print(f"Content-Length: {response.headers.get('Content-Length')} bytes")
```

**Resultado esperado:**
```
Expiração: 31536000s = 365.0 dias
Status: 200
Content-Type: model/gltf-binary
Content-Length: 157824 bytes
```

### Teste 2: Carregar GLB no app mobile

**React Native (Expo):**
```typescript
import * as FileSystem from 'expo-file-system';

async function testGLBAccess(glbSignedUrl: string) {
  console.log('🔍 Testando acesso ao GLB...');
  
  try {
    // Download GLB
    const downloadResult = await FileSystem.downloadAsync(
      glbSignedUrl,
      FileSystem.cacheDirectory + 'test_model.glb'
    );
    
    console.log('✅ GLB baixado com sucesso!');
    console.log('Status:', downloadResult.status);
    console.log('URI local:', downloadResult.uri);
    
    // Verificar tamanho
    const info = await FileSystem.getInfoAsync(downloadResult.uri);
    console.log('Tamanho:', (info.size! / 1024).toFixed(2), 'KB');
    
    return downloadResult.uri;
  } catch (e) {
    console.error('❌ Erro ao baixar GLB:', e);
  }
}

// Usar
const localGLB = await testGLBAccess(payload.blocos[0].glb_signed_url);
```

### Teste 3: Verificar expiração

**JavaScript (Browser/Node):**
```javascript
function checkSignedUrlExpiration(signedUrl) {
  const url = new URL(signedUrl);
  const params = new URLSearchParams(url.search);
  
  const expiresSeconds = parseInt(params.get('X-Goog-Expires') || '0');
  const dateString = params.get('X-Goog-Date');
  
  if (!dateString) {
    console.error('❌ Sem X-Goog-Date na URL');
    return;
  }
  
  // Parsear data: 20251103T120000Z → Date
  const year = parseInt(dateString.slice(0, 4));
  const month = parseInt(dateString.slice(4, 6)) - 1;
  const day = parseInt(dateString.slice(6, 8));
  const hour = parseInt(dateString.slice(9, 11));
  const minute = parseInt(dateString.slice(11, 13));
  const second = parseInt(dateString.slice(13, 15));
  
  const createdAt = new Date(Date.UTC(year, month, day, hour, minute, second));
  const expiresAt = new Date(createdAt.getTime() + expiresSeconds * 1000);
  const now = new Date();
  
  console.log('📅 Criada em:', createdAt.toISOString());
  console.log('⏰ Expira em:', expiresAt.toISOString());
  console.log('🕒 Agora:', now.toISOString());
  
  const daysRemaining = (expiresAt.getTime() - now.getTime()) / (1000 * 60 * 60 * 24);
  console.log(`⏳ Dias restantes: ${daysRemaining.toFixed(1)} dias`);
  
  if (daysRemaining < 0) {
    console.warn('❌ SIGNED URL EXPIRADA!');
  } else if (daysRemaining < 30) {
    console.warn('⚠️ Signed URL expira em breve!');
  } else {
    console.log('✅ Signed URL válida');
  }
}

// Usar
checkSignedUrlExpiration(payload.blocos[0].glb_signed_url);
```

---

## 🎯 Resumo

✅ **App consegue acessar GLBs em `{userId}/ra/models/`**

**Como:**
1. Backend gera **Signed URLs** (válidas por 365 dias)
2. App usa signed URLs para carregar GLBs
3. GCS valida assinatura e permite acesso
4. GLB é baixado e renderizado em AR

**Segurança:**
- ✅ Cada signed URL é única (assinatura criptográfica)
- ✅ Expiração automática (365 dias)
- ✅ Isolamento por `{userId}` (sem cross-access)

**Performance:**
- ✅ App não precisa autenticar
- ✅ Pode cachear GLB localmente
- ✅ Válida por 1 ano (sem regenerar constantemente)

---

**Última atualização:** 03/11/2025  
**Status:** ✅ Acesso validado e documentado
