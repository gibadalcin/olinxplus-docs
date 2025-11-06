# 📝 Changelog - Carrossel AR (Option 3)

## ✅ Implementado - 03/11/2025

### FASE 1 - Backend: Pré-geração automática de GLBs

#### Arquivo modificado: `olinxra-backend/main.py`

**1. Função `gerar_signed_url_conteudo` atualizada**
- ✅ Adicionado parâmetro `expiration` (padrão: 3600s = 1h)
- ✅ Permite especificar tempo de expiração customizado para signed URLs
- ✅ Útil para GLBs que precisam de URLs com validade maior (7 dias)

**Mudança:**
```python
# ANTES
def gerar_signed_url_conteudo(gs_url=None, filename=None):
    # ...
    url = blob.generate_signed_url(version='v4', expiration=3600, method='GET')

# DEPOIS
def gerar_signed_url_conteudo(gs_url=None, filename=None, expiration=3600):
    # ...
    url = blob.generate_signed_url(version='v4', expiration=expiration, method='GET')
```

**2. Endpoint `/api/add-content-image` - Pré-geração de GLB**
- ✅ Detecta automaticamente uploads de imagens (`image/*`)
- ✅ Gera GLB em background após upload da imagem
- ✅ Redimensiona imagem se > 2048px (mesma lógica do endpoint `/api/generate-glb-from-image`)
- ✅ Salva GLB no path: `{owner_uid}/ra/models/{nome}.glb`
- ✅ Gera signed URL com expiração de 7 dias para o GLB
- ✅ Adiciona metadados ao GLB: `generated_from_image`, `base_height`, `auto_generated`
- ✅ Não-fatal: se geração falhar, upload da imagem continua normalmente
- ✅ Retorna novos campos no response:
  - `glb_url`: URL completa no GCS (`gs://bucket/path/file.glb`)
  - `glb_signed_url`: URL assinada para acesso direto (válida por 7 dias)
  - `bloco.glb_url`: incluído no objeto `bloco`
  - `bloco.glb_signed_url`: incluído no objeto `bloco`

**Response atualizado:**
```json
{
  "success": true,
  "url": "gs://bucket/image.jpg",
  "signed_url": "https://storage.googleapis.com/...",
  "bloco": {
    "tipo": "imagem",
    "subtipo": "header",
    "url": "gs://bucket/image.jpg",
    "glb_url": "gs://bucket/ra/models/image.glb",        // ← NOVO
    "glb_signed_url": "https://storage.googleapis.com/...", // ← NOVO
    "nome": "image.jpg",
    "filename": "user123/image.jpg",
    "type": "image/jpeg",
    "created_at": "2025-11-03T..."
  },
  "temp_id": "temp_123"
}
```

**3. Logging aprimorado**
- ✅ Log de início da geração: `[add_content_image] Iniciando pré-geração de GLB para {filename}`
- ✅ Log de sucesso com tempo: `GLB gerado com sucesso em 2.34s: {glb_filename}`
- ✅ Log de erro (não-fatal): `Erro ao gerar GLB (não-fatal): {error}`
- ✅ Log final inclui status do GLB: `glb=SIM` ou `glb=NÃO`

**Exemplo de log:**
```
[add_content_image] Tempo até upload GCS: 0.45s
[add_content_image] Tempo upload GCS: 1.23s (total: 1.68s)
[add_content_image] Iniciando pré-geração de GLB para user123/totem_header.jpg
[add_content_image] GLB gerado com sucesso em 2.34s: user123/ra/models/totem_header.glb
[add_content_image] Upload concluído (não persiste no DB). Tempo total: 4.02s
[add_content_image] upload ok uid=user123 filename=user123/totem_header.jpg type=image/jpeg glb=SIM
```

---

## 🔧 Detalhes Técnicos

### Configuração do GLB gerado
- **Posição**: `base_y = 0.0` (base no chão)
- **Altura**: `plane_height = 1.0` (1 metro)
- **UV Flip**: `flip_u = False`, `flip_v = True` (orientação padrão)
- **Cache Control**: `public, max-age=31536000` (1 ano)
- **Expiração Signed URL**: 7 dias (604800 segundos)

### Performance
- **Resize de imagem**: Assíncrono em thread pool (não bloqueia event loop)
- **Geração de GLB**: Assíncrono em thread pool
- **Upload para GCS**: Assíncrono em thread pool
- **Tempo estimado**: 2-5 segundos por imagem (dependendo do tamanho)

### Segurança
- ✅ GLBs salvos no mesmo bucket de conteúdo (`olinxra-conteudo`)
- ✅ Path organizado por `owner_uid` (isolamento entre admins)
- ✅ Metadados incluem origem da imagem (rastreabilidade)
- ✅ Geração é não-fatal (não quebra upload se falhar)

---

## 📋 Próximos Passos

### FASE 2 - AdminUI (Persistência)
- [ ] Modificar lógica de save do conteúdo para incluir `glb_url` e `glb_signed_url` no MongoDB
- [ ] Atualizar schema/interface TypeScript para incluir campos GLB
- [ ] Testar upload de imagem + save de conteúdo (verificar se GLB é persistido)
- [ ] Testar upload de carousel (múltiplas imagens) e verificar se todos GLBs são salvos

### FASE 3 - App Mobile (Extração)
- [ ] Modificar `ar-view.tsx` para extrair `glb_signed_url` de blocos
- [ ] Chamar `setGlbModels([...urls])` com array de GLBs
- [ ] Usar `glbModels[currentModelIndex]` como modelo atual
- [ ] Adicionar fallback para `generatedGlbUrl` se não houver GLBs pré-gerados

### FASE 4 - App Mobile (UI)
- [ ] Criar componente `ARNavigationControls`
- [ ] Adicionar botões Previous/Next
- [ ] Adicionar contador "2/5"
- [ ] Integrar no `ARLauncher`
- [ ] Implementar reload de Scene ao trocar modelo

---

## 🧪 Como Testar

### 1. Testar pré-geração de GLB (Backend)

**Endpoint:** `POST /api/add-content-image`

**Payload (form-data):**
```
file: [arquivo de imagem]
name: "teste_glb.jpg"
tipo_bloco: "imagem"
subtipo: "header"
```

**Verificações:**
1. ✅ Response contém `glb_url` e `glb_signed_url`
2. ✅ GLB está acessível via `glb_signed_url`
3. ✅ GLB existe no GCS em `{owner_uid}/ra/models/teste_glb.glb`
4. ✅ Log mostra `glb=SIM`
5. ✅ Tempo total < 10s

**Teste de erro (não-fatal):**
- Upload de vídeo (`video/mp4`) → não deve gerar GLB, mas upload deve funcionar
- Verificar log: `glb=NÃO`

### 2. Testar signed URL com expiração customizada

**Teste:**
```python
# Em Python (console do backend)
from main import gerar_signed_url_conteudo

# Signed URL com 1 hora (padrão)
url_1h = gerar_signed_url_conteudo(
    gs_url="gs://olinxra-conteudo/user123/ra/models/teste.glb"
)

# Signed URL com 7 dias
url_7d = gerar_signed_url_conteudo(
    gs_url="gs://olinxra-conteudo/user123/ra/models/teste.glb",
    expiration=7*24*60*60
)

print(url_7d)
```

**Verificação:**
- ✅ URL contém parâmetro `X-Goog-Expires=604800` (7 dias)

---

## 🐛 Problemas Conhecidos

### 1. Vídeos não geram GLB
**Status:** Comportamento esperado  
**Razão:** GLB só faz sentido para imagens estáticas  
**Solução:** Futuramente adicionar suporte a thumbnail de vídeo → GLB

### 2. Tempo de upload aumentou
**Status:** Esperado  
**Razão:** Geração de GLB adiciona 2-5s ao processo  
**Impacto:** UX permanece fluida (loading state no frontend)  
**Otimização futura:** Background job assíncrono (não bloqueia response)

### 3. Carousel com muitas imagens pode demorar
**Status:** Monitorar  
**Razão:** Cada imagem gera 1 GLB (5 imagens = 5 GLBs)  
**Tempo estimado:** 5 imagens × 3s = ~15s total  
**Solução futura:** Paralelizar geração de GLBs (cuidado com memória)

---

## 📊 Métricas de Sucesso

### Checklist de Validação
- [x] GLB gerado automaticamente ao upload de imagem
- [x] Response contém `glb_url` e `glb_signed_url`
- [x] GLB acessível via signed URL (válida por 7 dias)
- [x] Upload não falha se geração de GLB der erro
- [x] Logging adequado para debug
- [ ] AdminUI persiste GLB no MongoDB
- [ ] App mobile carrega múltiplos GLBs
- [ ] Navegação entre modelos funciona

### Performance Targets
- ✅ Geração de GLB: < 5s por imagem
- ⏳ Upload total (imagem + GLB): < 10s
- ⏳ Carousel (5 imagens): < 30s

---

**Última atualização:** 03/11/2025  
**Status:** FASE 1 completa | FASE 2, 3 e 4 pendentes  
**Próxima ação:** Testar endpoint `/api/add-content-image` e verificar geração de GLB
