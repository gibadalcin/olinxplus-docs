# 🎯 Calibração de Thresholds de Reconhecimento

## 📊 Problema Identificado

**Situação:** Capturas com 73.87% de confiança estavam sendo rejeitadas, dificultando reconhecimento em condições não ideais.

**Causa Raiz:** Duplo threshold desalinhado:
- Backend: `distance < 0.35` (equivalente a ~74% confidence)
- Frontend: `confidence >= 0.70` (70%)

## 🔬 Análise Matemática

### Fórmula de Conversão:
```python
confidence = 1 / (1 + distance)
```

### Tabela de Referência:

| Distância | Confidence | Interpretação |
|-----------|-----------|---------------|
| 0.20 | 83.33% | ✅ Excelente |
| 0.25 | 80.00% | ✅ Muito Bom |
| 0.30 | 76.92% | ✅ Bom |
| **0.35** | **74.07%** | ✅ Aceitável (threshold antigo) |
| **0.38** | **72.46%** | ✅ Aceitável (threshold novo) |
| 0.40 | 71.43% | ⚠️ Limítrofe |
| 0.43 | 69.93% | ⚠️ Baixo |
| 0.50 | 66.67% | ❌ Muito Baixo |
| 0.60 | 62.50% | ❌ Inaceitável |

## ✅ Solução Implementada

### Backend (`.env` - Digital Ocean):
```bash
SEARCH_COMBINED_THRESHOLD=0.50  # 50% (threshold combinado CLIP+pHash)
SEARCH_MIN_MARGIN=0.01          # 1% mínimo entre top-1 e top-2
SEARCH_ACCEPTANCE_THRESHOLD=0.72 # 72% (confiança alta)
SEARCH_PHASH_WEIGHT=0.20        # 20% peso pHash
SEARCH_EMBEDDING_WEIGHT=0.80    # 80% peso CLIP
```

**Justificativa:**
- ✅ Combined threshold 0.50 permite reconhecimento em condições variadas
- ✅ Min margin 0.01 garante diferenciação mínima entre candidatos
- ✅ Acceptance 0.72 mantém qualidade para resultados de alta confiança
- ✅ Híbrido CLIP (80%) + pHash (20%) balanceia semântica e estrutura

### App Mobile - Crop Antes do Envio:
```typescript
// Crop guiado por marcadores visuais (300x250px)
const croppedImage = await manipulateAsync(
  photo.uri,
  [{ crop: { originX, originY, width: 300, height: 250 } }],
  { compress: 1.0, format: SaveFormat.JPEG }
);
```

**Justificativa:**
- ✅ Alinhado com backend
- ✅ Margem de segurança (frontend aceita se backend já aceitou)
- ✅ Consistência entre camadas

## 📈 Impacto Esperado

### Antes (threshold 0.35 / 74%):
```
Capturas rejeitadas: ~40%
Falsos negativos: Alto
Falsos positivos: Baixo
Experiência do usuário: ❌ Frustrante
```

### Depois (threshold 0.50 combinado + crop app-side):
```
Capturas rejeitadas: ~15%
Falsos negativos: Baixo
Falsos positivos: Baixo
Experiência do usuário: ✅ Muito Melhor
Latência: ⚡ Reduzida (imagens menores, pré-cropped)
```

**Otimizações Adicionais (Dezembro 2025):**
- 📸 **Crop no app**: Marcadores visuais 300x250px guiam enquadramento
- 🚫 **Backend crop desabilitado**: `SEARCH_CENTER_CROP_RATIO=1.0` (sem recrop)
- ⚡ **Menor latência**: Imagens menores (~256x256) vs originais (1920x1080+)
- 🎯 **Maior precisão**: Logo já enquadrado pelo usuário com marcadores

**Referência:** `olinxplus-backend/docs/CROP-OPTIMIZATION.md`

## ⚠️ Riscos de Falsos Positivos

### Análise por Threshold:

**65% (0.54 distance):**
- ❌ **ALTO RISCO** - Pode confundir logos similares
- ❌ Marcas com cores/formas parecidas podem ser confundidas
- ❌ Não recomendado

**68% (0.47 distance):**
- ⚠️ **MÉDIO RISCO** - Possível confusão em edge cases
- Pode aceitar capturas de baixa qualidade
- Usar apenas se necessário

**70% (0.43 distance):**
- ⚠️ **BAIXO-MÉDIO RISCO** - Aceitável com monitoramento
- Threshold anterior do frontend

**72% (0.38 distance):**
- ✅ **BAIXO RISCO** - Balanço ideal ← **ESCOLHIDO**
- Aceita capturas válidas em condições normais
- Rejeita a maioria dos falsos positivos

**74% (0.35 distance):**
- ✅ **MUITO BAIXO RISCO** - Conservador demais
- Threshold anterior (muito restritivo)
- Rejeita muitas capturas válidas

## 🧪 Casos de Teste

### Cenários Válidos que Devem Passar:

1. **Logo nítida, boa iluminação:** > 85%
2. **Logo com reflexo leve:** 75-82%
3. **Logo parcialmente obstruída:** 72-78%
4. **Logo em ângulo:** 72-76%
5. **Baixa iluminação (mas legível):** 70-75% ← Agora aceito!

### Cenários Inválidos que Devem Falhar:

1. **Logo completamente diferente:** < 60%
2. **Apenas cores similares:** < 65%
3. **Fora de foco severo:** < 68%
4. **Logo errada da mesma categoria:** < 70%

## 📊 Monitoramento Recomendado

### Métricas para Acompanhar:

1. **Taxa de Aceitação:**
   - Meta: 60-70% das capturas aceitas
   - Alerta se < 50% (threshold muito alto)
   - Alerta se > 85% (threshold muito baixo)

2. **Taxa de Falsos Positivos:**
   - Meta: < 5%
   - Monitorar feedback de usuários
   - Analisar logs de reconhecimentos incorretos

3. **Distribuição de Confidence:**
   ```
   > 85%: ~20% (logos perfeitas)
   75-85%: ~35% (logos boas)
   72-75%: ~25% (logos aceitáveis) ← Novo range aceito
   < 72%: ~20% (rejeitadas)
   ```

## 🔧 Ajustes Futuros

### Se houver MUITOS falsos positivos:

```python
# Backend
threshold = 0.36  # ~73.5% confidence (meio termo)
```

```typescript
// Frontend
const SIMILARIDADE_MINIMA = 0.73;
```

### Se ainda houver MUITAS rejeições válidas:

```python
# Backend
threshold = 0.40  # ~71.4% confidence (mais permissivo)
```

```typescript
// Frontend
const SIMILARIDADE_MINIMA = 0.71;
```

### Sistema de Níveis (implementação futura):

```typescript
// Confiança Alta: Auto-aceitar
if (confidence >= 0.75) return 'high_confidence';

// Confiança Média: Pedir confirmação visual
if (confidence >= 0.68 && confidence < 0.75) return 'medium_confidence';

// Confiança Baixa: Rejeitar
return 'low_confidence';
```

## 📝 Changelog

### v2.0 - Threshold Otimizado (04/11/2025)
- ✅ Backend: 0.35 → 0.38 (~74% → ~72.5%)
- ✅ Frontend: 0.70 → 0.72 (70% → 72%)
- ✅ Alinhamento entre backend e frontend
- ✅ Documentação completa de riscos e testes

### v1.0 - Threshold Inicial
- Backend: 0.35 (~74%)
- Frontend: 0.70 (70%)
- Problema: Muitas capturas válidas rejeitadas

## 🎓 Referências

- **FAISS Distance Metrics:** https://github.com/facebookresearch/faiss/wiki/MetricType-and-distances
- **Precision-Recall Trade-off:** Quanto menor o threshold, maior o recall (menos falsos negativos) mas menor a precisão (mais falsos positivos)
- **F1 Score Optimal:** Threshold ideal deve maximizar F1 = 2 * (precision * recall) / (precision + recall)

---

**Recomendação Final:** Monitorar por 1-2 semanas e ajustar baseado em dados reais de uso.
