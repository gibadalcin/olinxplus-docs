# 🧪 Guia de Teste - Pré-geração de GLB

## Pré-requisitos
- Backend rodando: `python run.py` em `olinxra-backend/`
- Firebase Admin SDK configurado
- GCS buckets configurados (logos e conteudo)
- Usuário autenticado no AdminUI

---

## Teste 1: Upload de imagem com pré-geração de GLB

### Método Manual (Postman/Insomnia)

**Endpoint:** `POST http://localhost:8000/api/add-content-image`

**Headers:**
```
Authorization: Bearer {seu_firebase_id_token}
Content-Type: multipart/form-data
```

**Body (form-data):**
```
file: [selecionar arquivo de imagem .jpg/.png]
name: "teste_carrossel_1.jpg"
tipo_bloco: "imagem"
subtipo: "header"
temp_id: "temp_12345"
```

**Response esperado (200 OK):**
```json
{
  "success": true,
  "url": "gs://olinxra-conteudo/{uid}/teste_carrossel_1.jpg",
  "signed_url": "https://storage.googleapis.com/...",
  "bloco": {
    "tipo": "imagem",
    "subtipo": "header",
    "url": "gs://olinxra-conteudo/{uid}/teste_carrossel_1.jpg",
    "glb_url": "gs://olinxra-conteudo/{uid}/ra/models/teste_carrossel_1.glb",
    "glb_signed_url": "https://storage.googleapis.com/olinxra-conteudo/{uid}/ra/models/teste_carrossel_1.glb?...",
    "nome": "teste_carrossel_1.jpg",
    "filename": "{uid}/teste_carrossel_1.jpg",
    "type": "image/jpeg",
    "created_at": "2025-11-03T..."
  },
  "temp_id": "temp_12345"
}
```

**Verificações:**
- [x] Status 200 OK
- [x] Campo `glb_url` presente
- [x] Campo `glb_signed_url` presente
- [x] `glb_signed_url` é acessível (copiar e colar no navegador → deve baixar .glb)
- [x] Tamanho do GLB: ~50-500KB (dependendo da imagem)

**Logs no terminal do backend:**
```
[add_content_image] recebendo upload: filename=teste_carrossel_1.jpg ...
[add_content_image] Tempo até upload GCS: 0.XX s
[add_content_image] Tempo upload GCS: 1.XX s (total: 1.XX s)
[add_content_image] Iniciando pré-geração de GLB para {uid}/teste_carrossel_1.jpg
[add_content_image] GLB gerado com sucesso em 2.XX s: {uid}/ra/models/teste_carrossel_1.glb
[add_content_image] Upload concluído (não persiste no DB). Tempo total: 4.XX s
[add_content_image] upload ok uid={uid} filename={uid}/teste_carrossel_1.jpg type=image/jpeg glb=SIM
```

---

## Teste 2: Upload de vídeo (não deve gerar GLB)

**Body (form-data):**
```
file: [selecionar arquivo .mp4]
name: "teste_video.mp4"
tipo_bloco: "video"
subtipo: "card"
```

**Response esperado:**
```json
{
  "success": true,
  "url": "gs://olinxra-conteudo/{uid}/teste_video.mp4",
  "signed_url": "https://storage.googleapis.com/...",
  "bloco": {
    "tipo": "video",
    "subtipo": "card",
    "url": "gs://olinxra-conteudo/{uid}/teste_video.mp4",
    // ❌ NÃO deve conter glb_url
    // ❌ NÃO deve conter glb_signed_url
    "nome": "teste_video.mp4",
    "filename": "{uid}/teste_video.mp4",
    "type": "video/mp4",
    "created_at": "2025-11-03T..."
  }
}
```

**Verificações:**
- [x] Upload bem-sucedido
- [x] `glb_url` não presente (vídeos não geram GLB)
- [x] Log mostra `glb=NÃO`

---

## Teste 3: Verificar GLB no GCS

### Via gsutil (CLI)
```bash
# Listar GLBs gerados
gsutil ls gs://olinxra-conteudo/{seu_uid}/ra/models/

# Ver metadados do GLB
gsutil stat gs://olinxra-conteudo/{seu_uid}/ra/models/teste_carrossel_1.glb
```

**Metadados esperados:**
```
Metadata:
  auto_generated: true
  base_height: 0.0
  generated_from_image: gs://olinxra-conteudo/{uid}/teste_carrossel_1.jpg
Cache-Control: public, max-age=31536000
```

### Via Google Cloud Console
1. Abrir: https://console.cloud.google.com/storage/browser/olinxra-conteudo
2. Navegar para: `{seu_uid}/ra/models/`
3. Verificar existência de `teste_carrossel_1.glb`
4. Clicar no arquivo → Ver metadados

---

## Teste 4: Validar Signed URL (expiração 7 dias)

**Script Python:**
```python
import re
from urllib.parse import parse_qs, urlparse

# Cole aqui o glb_signed_url retornado
signed_url = "https://storage.googleapis.com/olinxra-conteudo/..."

# Parsear URL
parsed = urlparse(signed_url)
params = parse_qs(parsed.query)

# Verificar expiração
expires = params.get('X-Goog-Expires', [None])[0]
print(f"Expiração: {expires} segundos")
print(f"Dias: {int(expires) / (24*60*60):.1f} dias")

# Deve mostrar: ~7.0 dias (604800 segundos)
```

**Verificação:**
- [x] Expiração = 604800 segundos (7 dias)
- [x] URL acessível (download do GLB funciona)

---

## Teste 5: Performance (tempo de geração)

**Imagens de teste:**
- Pequena (< 500KB): ~2s
- Média (1-2MB): ~3-4s
- Grande (> 5MB): ~5-8s (inclui resize)

**Como testar:**
1. Fazer upload de imagens de tamanhos diferentes
2. Observar logs: `GLB gerado com sucesso em X.XXs`
3. Verificar tempo total: `Tempo total: X.XXs`

**Targets:**
- [x] Pequena: < 3s
- [x] Média: < 5s
- [x] Grande: < 10s

---

## Teste 6: Visualizar GLB gerado

### Opção 1: Online Viewer
1. Copiar `glb_signed_url` do response
2. Abrir: https://gltf-viewer.donmccurdy.com/
3. Colar URL no campo "URL"
4. Verificar se modelo aparece corretamente

**Verificações:**
- [x] Plano vertical (em pé)
- [x] Imagem aplicada como textura
- [x] Dimensões: ~1m de largura × 1m de altura
- [x] Posição: base no chão (Y=0)

### Opção 2: Blender
1. Baixar GLB via signed URL
2. Abrir Blender
3. File → Import → glTF 2.0 (.glb/.gltf)
4. Selecionar arquivo baixado

---

## Teste 7: Erro não-fatal (simulação)

**Como simular:**
1. Modificar temporariamente `glb_generator.py` para lançar exceção
2. Fazer upload de imagem
3. Verificar que upload continua funcionando (response sem `glb_url`)
4. Log deve mostrar: `Erro ao gerar GLB (não-fatal): ...`

**Verificação:**
- [x] Upload da imagem funciona
- [x] Response não contém `glb_url`
- [x] Status 200 OK (não 500)
- [x] Log de erro presente

---

## Teste 8: AdminUI (Integração end-to-end)

### Fluxo completo:
1. Abrir AdminUI (http://localhost:5173)
2. Login como admin
3. Ir para Content Manager
4. Criar novo conteúdo (ou editar existente)
5. Adicionar bloco de imagem (tipo: header)
6. Fazer upload de imagem
7. **Verificar no Network tab do DevTools:**
   - Request para `/api/add-content-image`
   - Response contém `glb_url` e `glb_signed_url`
8. Salvar conteúdo
9. Verificar no MongoDB se documento contém campos GLB

**MongoDB Query:**
```javascript
db.conteudos.findOne(
  { "blocos.glb_url": { $exists: true } },
  { "blocos.$": 1 }
)
```

**Resultado esperado:**
```json
{
  "_id": ObjectId("..."),
  "blocos": [
    {
      "tipo": "imagem",
      "url": "gs://...",
      "glb_url": "gs://olinxra-conteudo/{uid}/ra/models/...",
      "glb_signed_url": "https://storage.googleapis.com/..."
    }
  ]
}
```

---

## 📝 Checklist Final

- [ ] Teste 1: Upload de imagem → GLB gerado ✅
- [ ] Teste 2: Upload de vídeo → sem GLB ✅
- [ ] Teste 3: GLB existe no GCS ✅
- [ ] Teste 4: Signed URL válida por 7 dias ✅
- [ ] Teste 5: Performance aceitável (< 10s) ✅
- [ ] Teste 6: GLB visualizado corretamente ✅
- [ ] Teste 7: Erro não-fatal funciona ✅
- [ ] Teste 8: AdminUI persiste GLB no MongoDB ⏳

---

## 🐛 Troubleshooting

### Problema: GLB não é gerado
**Sintomas:** Response não contém `glb_url`  
**Verificar:**
1. Logs do backend: erro na geração?
2. Tipo do arquivo: é imagem (`image/*`)?
3. Permissões GCS: bucket `olinxra-conteudo` acessível?

### Problema: Signed URL inválida
**Sintomas:** 403 Forbidden ao acessar `glb_signed_url`  
**Verificar:**
1. Credenciais do GCS configuradas corretamente
2. Service account tem permissão de leitura no bucket
3. URL expirou? (deveria durar 7 dias)

### Problema: Tempo muito lento (> 15s)
**Sintomas:** Upload demora muito  
**Verificar:**
1. Tamanho da imagem: > 5MB? (será redimensionada)
2. Rede: latência alta para GCS?
3. CPU: servidor sobrecarregado?

**Otimização:**
- Reduzir `GLB_MAX_DIM` (padrão: 2048px)
- Adicionar cache de GLBs já gerados (verificar hash antes de gerar)

---

**Última atualização:** 03/11/2025  
**Próximo:** Após validar testes, avançar para FASE 2 (AdminUI persistência)
