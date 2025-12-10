# 📁 Arquitetura de Armazenamento - Olinx Plus

## Estrutura de Pastas no GCS (Google Cloud Storage)

### Bucket: `olinxra-conteudo`

```
olinxra-conteudo/
│
├── {userId_1}/                    ← Pasta isolada do Admin 1
│   ├── totem_header.jpg           ← Imagens originais
│   ├── totem_logo.png
│   ├── produto_1.jpg
│   ├── produto_2.jpg
│   │
│   └── ra/                        ← Subpasta de conteúdo AR
│       └── models/                ← GLBs gerados automaticamente
│           ├── totem_header.glb   ← GLB da imagem totem_header.jpg
│           ├── totem_logo.glb
│           ├── produto_1.glb
│           └── produto_2.glb
│
├── {userId_2}/                    ← Pasta isolada do Admin 2
│   ├── banner.jpg
│   ├── carousel_1.jpg
│   ├── carousel_2.jpg
│   │
│   └── ra/
│       └── models/
│           ├── banner.glb
│           ├── carousel_1.glb
│           └── carousel_2.glb
│
└── anonymous/                     ← Fallback (sem autenticação)
    └── ra/
        └── models/
            └── temp_*.glb         ← GLBs temporários
```

---

## 🎯 Regras de Organização

### 1. **Imagens Originais**
**Path:** `{userId}/nome_arquivo.jpg`

**Exemplos:**
```
gs://olinxra-conteudo/TR77xSOJ.../totem_header.jpg
gs://olinxra-conteudo/TR77xSOJ.../produto_destaque.png
gs://olinxra-conteudo/yiF2ZJyB.../banner_promocional.jpg
```

**Quando criado:**
- Upload via AdminUI (`/api/add-content-image`)
- Upload de carousel (múltiplas imagens)

---

### 2. **GLBs Gerados**
**Path:** `{userId}/ra/models/nome_arquivo.glb`

**Exemplos:**
```
gs://olinxra-conteudo/TR77xSOJ.../ra/models/totem_header.glb
gs://olinxra-conteudo/TR77xSOJ.../ra/models/produto_destaque.glb
gs://olinxra-conteudo/yiF2ZJyB.../ra/models/banner_promocional.glb
```

**Quando criado:**
- ✅ **Automático**: ao fazer upload de imagem (endpoint `/api/add-content-image`)
- ✅ **Manual**: via script `tools/generate_glbs_from_existing_images.py`
- ✅ **On-demand**: via endpoint `/api/generate-glb-from-image`

**Metadados:**
```json
{
  "generated_from_image": "gs://olinxra-conteudo/{userId}/totem_header.jpg",
  "base_height": "0.0",
  "auto_generated": "true"
}
```

---

## ✅ Vantagens desta Arquitetura

### 1. **Isolamento perfeito**
- ✅ Admin A não vê conteúdo do Admin B
- ✅ GLBs ficam na mesma pasta do usuário que fez upload
- ✅ Fácil aplicar permissões por pasta (ACL)

### 2. **Gerenciamento simplificado**
- ✅ Deletar `{userId}/` remove TUDO (imagens + GLBs)
- ✅ Backup/restore por usuário
- ✅ Fácil calcular quota por admin

### 3. **Lógica consistente**
```
Imagem:  {userId}/image.jpg
GLB:     {userId}/ra/models/image.glb
                  └─ sempre no mesmo prefixo {userId}
```

### 4. **Escalabilidade**
- ✅ Suporta milhares de admins sem conflito de nomes
- ✅ Path previsível (fácil debugar)
- ✅ CDN pode cachear por prefixo

---

## 🔄 Fluxo de Upload e Geração

### Fluxo Atual (com pré-geração automática)

```mermaid
graph TD
    A[Admin faz upload de imagem] --> B[POST /api/add-content-image]
    B --> C[Salvar imagem em {userId}/image.jpg]
    C --> D[Gerar GLB automaticamente]
    D --> E[Salvar GLB em {userId}/ra/models/image.glb]
    E --> F[Retornar response com glb_url e glb_signed_url]
    F --> G[AdminUI salva conteúdo no MongoDB]
    G --> H[App Mobile carrega GLB via glb_signed_url]
```

**Tempo total:** ~3-5s por imagem

---

## 📊 Exemplos Reais

### Exemplo 1: Upload de Header

**Request:**
```http
POST /api/add-content-image
Content-Type: multipart/form-data

file: totem_header.jpg (2MB)
name: "totem_header.jpg"
tipo_bloco: "imagem"
subtipo: "header"
```

**GCS após upload:**
```
olinxra-conteudo/
└── TR77xSOJ.../
    ├── totem_header.jpg              ← 2MB
    └── ra/
        └── models/
            └── totem_header.glb      ← 150KB
```

**Response:**
```json
{
  "success": true,
  "url": "gs://olinxra-conteudo/TR77xSOJ.../totem_header.jpg",
  "bloco": {
    "tipo": "imagem",
    "url": "gs://olinxra-conteudo/TR77xSOJ.../totem_header.jpg",
    "glb_url": "gs://olinxra-conteudo/TR77xSOJ.../ra/models/totem_header.glb",
    "glb_signed_url": "https://storage.googleapis.com/..."
  }
}
```

---

### Exemplo 2: Carousel com 3 imagens

**Request:**
```http
POST /api/add-content-image (3x)
```

**GCS após upload:**
```
olinxra-conteudo/
└── TR77xSOJ.../
    ├── produto_1.jpg
    ├── produto_2.jpg
    ├── produto_3.jpg
    └── ra/
        └── models/
            ├── produto_1.glb
            ├── produto_2.glb
            └── produto_3.glb
```

**MongoDB (documento de conteúdo):**
```json
{
  "_id": "...",
  "owner_uid": "TR77xSOJ...",
  "blocos": [
    {
      "tipo": "carousel",
      "imagens": [
        {
          "url": "gs://.../produto_1.jpg",
          "glb_url": "gs://.../ra/models/produto_1.glb",
          "glb_signed_url": "https://..."
        },
        {
          "url": "gs://.../produto_2.jpg",
          "glb_url": "gs://.../ra/models/produto_2.glb",
          "glb_signed_url": "https://..."
        },
        {
          "url": "gs://.../produto_3.jpg",
          "glb_url": "gs://.../ra/models/produto_3.glb",
          "glb_signed_url": "https://..."
        }
      ]
    }
  ]
}
```

**App Mobile (ar-view.tsx):**
```typescript
const glbModels = [
  "https://storage.googleapis.com/.../produto_1.glb",
  "https://storage.googleapis.com/.../produto_2.glb",
  "https://storage.googleapis.com/.../produto_3.glb"
];

setGlbModels(glbModels);
// Usuário navega: ◀ 1/3 ▶ ◀ 2/3 ▶ ◀ 3/3 ▶
```

---

## 🔧 Implementação Backend

### Endpoint: `/api/add-content-image` (linha 2036)

```python
# Gerar GLB a partir da imagem recém-uploadada
glb_filename = f"{token['uid']}/ra/models/{name_base}.glb"
#                └─ userId ─┘ └─ subpasta AR ─┘ └─ nome.glb ─┘
```

**Resultado:**
- Imagem: `gs://olinxra-conteudo/{userId}/totem_header.jpg`
- GLB:    `gs://olinxra-conteudo/{userId}/ra/models/totem_header.glb`

---

### Endpoint: `/api/generate-glb-from-image` (linha 682-690)

```python
# ANTES (tinha fallback público - REMOVIDO)
if owner_uid:
    filename = f"{owner_uid}/ra/models/{base_filename}"
else:
    filename = f"public/ra/models/{base_filename}"  # ❌ PÚBLICO

# DEPOIS (sempre isolado por usuário)
if not owner_uid:
    logging.warning("owner_uid não fornecido, usando 'anonymous'")
    owner_uid = 'anonymous'  # fallback seguro

filename = f"{owner_uid}/ra/models/{base_filename}"  # ✅ ISOLADO
```

---

## 🛡️ Segurança

### Permissões GCS (ACL)

**Recomendado:**
```bash
# Cada admin só pode acessar sua pasta
gsutil iam ch user:{admin_email}:objectViewer gs://olinxra-conteudo/{userId}/

# Backend tem acesso total
gsutil iam ch serviceAccount:{backend_sa}:objectAdmin gs://olinxra-conteudo/
```

### Signed URLs

**Imagens:** Expiração 1h (padrão)
```
https://storage.googleapis.com/olinxra-conteudo/{userId}/image.jpg?
  X-Goog-Expires=3600&...
```

**GLBs:** Expiração 7 dias (longa)
```
https://storage.googleapis.com/olinxra-conteudo/{userId}/ra/models/image.glb?
  X-Goog-Expires=604800&...
```

**Por quê 7 dias?**
- ✅ App mobile pode cachear GLB localmente
- ✅ Não precisa regenerar signed URL a cada abertura
- ✅ Mesmo GLB pode ser usado em múltiplas sessões AR

---

## 📝 Checklist de Validação

### Verificar estrutura no GCS:
```bash
# Listar pastas de usuários
gsutil ls gs://olinxra-conteudo/

# Ver conteúdo de um usuário específico
gsutil ls -r gs://olinxra-conteudo/{userId}/

# Verificar GLBs gerados
gsutil ls gs://olinxra-conteudo/{userId}/ra/models/
```

### Verificar no MongoDB:
```javascript
// Contar documentos com GLBs
db.conteudos.countDocuments({
  "blocos.glb_url": { $regex: /\/ra\/models\// }
})

// Verificar path correto (deve conter {userId}/ra/models/)
db.conteudos.findOne(
  { "blocos.glb_url": { $exists: true } },
  { "blocos.glb_url": 1, "owner_uid": 1 }
)
```

**Resultado esperado:**
```json
{
  "owner_uid": "TR77xSOJ...",
  "blocos": [
    {
      "glb_url": "gs://olinxra-conteudo/TR77xSOJ.../ra/models/image.glb"
      //                                └─ MESMO userId ─┘
    }
  ]
}
```

---

## 🎯 Resumo

✅ **Arquitetura atualizada e validada:**

```
Imagens:  {userId}/image.jpg
GLBs:     {userId}/ra/models/image.glb
          └─ sempre isolado por userId
          └─ não usa mais public/ra/models/
```

✅ **Benefícios:**
- Isolamento perfeito
- Gerenciamento simplificado
- Segurança por design
- Escalável

✅ **Código atualizado:**
- `/api/add-content-image` - já estava correto
- `/api/generate-glb-from-image` - ajustado para remover fallback público
- Script `generate_glbs_from_existing_images.py` - já usa owner_uid corretamente

---

**Última atualização:** 03/11/2025  
**Status:** ✅ Arquitetura validada e implementada
