# 📋 Plano de Implementação - Carrossel AR (Option 3)

**Data de criação:** 03/11/2025  
**Última atualização:** 10/12/2025  
**Status:** ✅ Implementado e em produção  
**Objetivo:** Permitir navegação entre múltiplos modelos 3D em AR nativa

**Projeto:** Olinx Plus  
**Repositórios:**
- Mobile: [github.com/gibadalcin/olinxplus](https://github.com/gibadalcin/olinxplus)
- Backend: [github.com/gibadalcin/olinxplus-backend](https://github.com/gibadalcin/olinxplus-backend)
- Admin: [github.com/gibadalcin/olinxplus-adminui](https://github.com/gibadalcin/olinxplus-adminui)

---

## ✅ O que já foi feito (03/11/2025)

### 1. **ARPayloadContext.tsx** - Context preparado
- ✅ Adicionado `glbModels: string[]` - Array com URLs dos GLBs pré-gerados
- ✅ Adicionado `currentModelIndex: number` - Índice do modelo atual
- ✅ Implementado `setGlbModels(models)` - Carrega array de modelos
- ✅ Implementado `setCurrentModelIndex(index)` - Navega entre modelos
- ✅ Helper: reset de índice ao carregar novos modelos

**Localização:** `olinxplus/context/ARPayloadContext.tsx`

### 2. **Bug Fix - Usuários Anônimos**
- ✅ Removida autenticação anônima do `_layout.tsx`
- ✅ Removido `signInAnonymously` que criava 300+ usuários
- ✅ Script Python criado para limpar usuários anônimos existentes
- ✅ App agora é totalmente público (sem autenticação)

**Arquivos modificados:**
- `olinxplus/app/_layout.tsx`
- `olinxplus/app/_tabs/ar-view.tsx`
- `olinxplus-backend/tools/delete_anonymous_users.py`

### 3. **AdminUI - Build otimizado**
- ✅ Upgrade React 18 → 19 (compatibilidade react-leaflet)
- ✅ Code-splitting com React.lazy (Dashboard, Content, ImageManager, Register)
- ✅ Chunking otimizado (vendor único 757KB → 218KB gzipped)
- ✅ Removidos imports wildcard de react-icons (-5.6MB)
- ✅ Preview funcionando sem erros de forwardRef/createContext
- ✅ Deploy em produção (Vercel/Netlify)

**Arquivos modificados:**
- `olinxplus-adminui/package.json` (React 19)
- `olinxplus-adminui/vite.config.js` (chunking simplificado)
- `olinxplus-adminui/src/App.jsx` (lazy routes)
- `olinxplus-adminui/src/hooks/useImages.js` (imports estáticos)
- `olinxplus-adminui/src/components/imageContext/ImageCard.jsx` (imports estáticos)
- `olinxplus-adminui/src/components/contentContext/ContentBlockType.jsx` (sem wildcard icons)

---

## 🎯 Próximos Passos (Prioridade)

### **FASE 1 - Backend: Pré-geração de GLBs** 🔴 ALTA PRIORIDADE

#### 1.1 AdminUI - Upload com pré-geração automática

**Arquivo:** `olinxra-backend/main.py`

**Modificações necessárias:**
1. Endpoint `/api/upload-content-image` (ou similar)
   - Ao fazer upload de imagem/vídeo, gerar GLB automaticamente
   - Chamar `glb_generator.py` para cada mídia
   - Armazenar URL do GLB gerado junto com a imagem

2. Estrutura de dados esperada no MongoDB:
```json
{
  "blocos": [
    {
      "tipo": "imagem",
      "url": "gs://bucket/imagem1.jpg",
      "signed_url": "https://...",
      "glb_url": "gs://bucket/glb/imagem1.glb",  // <- NOVO
      "glb_signed_url": "https://...",            // <- NOVO
      "subtipo": "header",
      "meta": { ... }
    },
    {
      "tipo": "carousel",
      "imagens": [
        {
          "url": "gs://bucket/imagem2.jpg",
          "signed_url": "https://...",
          "glb_url": "gs://bucket/glb/imagem2.glb",  // <- NOVO
          "glb_signed_url": "https://...",            // <- NOVO
          "subtipo": "card"
        }
      ]
    }
  ]
}
```

**Checklist de implementação:**
- [ ] Adicionar campo `glb_url` ao schema de blocos
- [ ] Modificar endpoint de upload para gerar GLB automaticamente
- [ ] Gerar signed URL para cada GLB (expiração longa, ex: 7 dias)
- [ ] Salvar `glb_url` e `glb_signed_url` no MongoDB
- [ ] Testar upload de uma imagem e verificar se GLB foi gerado
- [ ] Testar upload de carousel (múltiplas imagens) e verificar GLBs

**Endpoint sugerido (novo ou modificado):**
```python
@app.post("/api/upload-content-with-glb")
async def upload_content_with_glb(
    file: UploadFile,
    owner_uid: str,
    tipo: str,
    subtipo: str = None
):
    # 1. Upload da imagem/vídeo original
    image_url = upload_to_gcs(file)
    
    # 2. Gerar GLB automaticamente
    glb_url = await generate_glb_from_image(image_url)
    
    # 3. Gerar signed URLs
    image_signed = get_signed_url(image_url, expiration=7*24*60*60)
    glb_signed = get_signed_url(glb_url, expiration=7*24*60*60)
    
    # 4. Retornar ambos
    return {
        "url": image_url,
        "signed_url": image_signed,
        "glb_url": glb_url,
        "glb_signed_url": glb_signed
    }
```

---

### **FASE 2 - App Mobile: Carregar múltiplos GLBs** 🟡 MÉDIA PRIORIDADE

#### 2.1 ar-view.tsx - Extração e carregamento de GLBs

**Arquivo:** `olinxra-app/app/(tabs)/ar-view.tsx`

**Modificações necessárias:**
1. Ao receber payload, extrair todos os `glb_url` ou `glb_signed_url`
2. Chamar `setGlbModels([...urls])`
3. Usar `glbModels[currentModelIndex]` como modelo atual

**Implementação sugerida:**
```typescript
// Dentro do useEffect que processa payload
useEffect(() => {
  if (!payload || !payload.blocos) return;
  
  // Extrair todos os GLBs pré-gerados
  const glbUrls: string[] = [];
  
  payload.blocos.forEach(bloco => {
    if (bloco.tipo === 'imagem' && bloco.glb_signed_url) {
      glbUrls.push(bloco.glb_signed_url);
    } else if (bloco.tipo === 'carousel' && bloco.imagens) {
      bloco.imagens.forEach(img => {
        if (img.glb_signed_url) {
          glbUrls.push(img.glb_signed_url);
        }
      });
    }
  });
  
  if (glbUrls.length > 0) {
    setGlbModels(glbUrls);
    console.log(`📦 Carregados ${glbUrls.length} modelos GLB para AR`);
  }
}, [payload]);

// Usar modelo atual baseado no índice
const currentGlbUrl = glbModels[currentModelIndex] || generatedGlbUrl;
```

**Checklist:**
- [ ] Adicionar lógica de extração de `glb_signed_url` de blocos
- [ ] Chamar `setGlbModels` com array de URLs
- [ ] Usar `glbModels[currentModelIndex]` no ARLauncher
- [ ] Adicionar fallback para `generatedGlbUrl` se não houver GLBs pré-gerados
- [ ] Log no console para debug (quantidade de modelos carregados)
- [ ] Testar com payload contendo múltiplas imagens

---

### **FASE 3 - ARLauncher: Controles de navegação** 🟢 BAIXA PRIORIDADE

#### 3.1 Componente de controles flutuantes

**Arquivo:** `olinxra-app/components/ar/ARLauncher.tsx`

**UI proposta:**
```
┌─────────────────────────────┐
│                             │
│         AR Scene            │
│                             │
│                             │
│      ◀    [2/5]    ▶       │ <- Controles flutuantes
└─────────────────────────────┘
```

**Implementação sugerida:**
```tsx
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { useARPayload } from '@/context/ARPayloadContext';

export function ARNavigationControls() {
  const { glbModels, currentModelIndex, setCurrentModelIndex } = useARPayload();
  
  // Se houver apenas 1 modelo, não mostrar controles
  if (glbModels.length <= 1) return null;
  
  const nextModel = () => {
    const nextIndex = (currentModelIndex + 1) % glbModels.length;
    setCurrentModelIndex(nextIndex);
  };
  
  const previousModel = () => {
    const prevIndex = (currentModelIndex - 1 + glbModels.length) % glbModels.length;
    setCurrentModelIndex(prevIndex);
  };
  
  return (
    <View style={styles.container}>
      <Pressable onPress={previousModel} style={styles.button}>
        <Text style={styles.arrow}>◀</Text>
      </Pressable>
      
      <Text style={styles.counter}>
        {currentModelIndex + 1}/{glbModels.length}
      </Text>
      
      <Pressable onPress={nextModel} style={styles.button}>
        <Text style={styles.arrow}>▶</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    position: 'absolute',
    bottom: 40,
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    gap: 20,
    alignSelf: 'center',
  },
  button: {
    backgroundColor: 'rgba(255, 255, 255, 0.9)',
    width: 56,
    height: 56,
    borderRadius: 28,
    justifyContent: 'center',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 5,
  },
  arrow: {
    fontSize: 24,
    color: '#000',
  },
  counter: {
    fontSize: 18,
    fontWeight: '600',
    color: '#fff',
    textShadowColor: '#000',
    textShadowOffset: { width: 0, height: 1 },
    textShadowRadius: 2,
  },
});
```

**Checklist:**
- [ ] Criar componente `ARNavigationControls`
- [ ] Adicionar botões Previous/Next
- [ ] Implementar lógica circular (último → primeiro e vice-versa)
- [ ] Adicionar contador visual "2/5"
- [ ] Estilizar com glass effect e sombras
- [ ] Integrar no ARLauncher
- [ ] Ocultar controles se houver apenas 1 modelo
- [ ] (Opcional) Adicionar swipe gestures com PanResponder
- [ ] (Opcional) Animação de transição suave

#### 3.2 Integração com Scene do ARCore

**Quando trocar modelo:**
1. Atualizar `currentModelIndex`
2. Recarregar Scene com novo GLB
3. Manter posição/escala atual (ou resetar)

**Código sugerido:**
```typescript
useEffect(() => {
  if (!glbModels || glbModels.length === 0) return;
  
  const currentGlb = glbModels[currentModelIndex];
  
  // Recarregar modelo no ARCore/SceneView
  loadModelInAR(currentGlb);
  
}, [currentModelIndex, glbModels]);
```

**Checklist:**
- [ ] Implementar reload de Scene ao trocar índice
- [ ] Adicionar loading state durante troca
- [ ] Testar performance (troca deve ser rápida)
- [ ] Decidir se mantém ou reseta posição/escala
- [ ] Animação de fade in/out (opcional)

---

## 🧪 Testes Necessários

### Backend
- [ ] Upload de 1 imagem → verifica se GLB foi gerado
- [ ] Upload de carousel (3 imagens) → verifica 3 GLBs gerados
- [ ] Signed URLs válidas e acessíveis
- [ ] Performance: tempo de geração de GLB aceitável

### App Mobile
- [ ] Payload com 1 imagem → carrega 1 GLB
- [ ] Payload com carousel (5 imagens) → carrega 5 GLBs
- [ ] Navegação Previous/Next funciona
- [ ] Contador "2/5" atualiza corretamente
- [ ] Último modelo → Next → volta ao primeiro (circular)
- [ ] Primeiro modelo → Previous → vai para o último
- [ ] Performance: troca de modelo é fluida
- [ ] Memória: não vaza ao trocar modelos repetidamente

---

## 📝 Notas Importantes

### Decisões de Design
- **Opção escolhida:** Carrossel AR (Option 3)
- **Motivo:** Menor uso de memória/GPU, UX mais focada
- **Trade-off:** Usuário vê 1 modelo por vez (não todos simultaneamente)

### Alternativas Descartadas
- ❌ **Option 1 (GLB em runtime):** Lento, ruim para UX
- ❌ **Option 2 (Gallery AR):** Muito pesado (múltiplos modelos carregados)

### Performance Expectations
- Geração de GLB: < 5s por imagem (backend)
- Troca de modelo: < 1s (app mobile)
- Memória: ~50-100MB por modelo (depende da complexidade)

---

## 🚀 Ordem de Execução Recomendada

**Dia 1 (Backend):**
1. Modificar schema do MongoDB (adicionar `glb_url`)
2. Implementar endpoint de upload com pré-geração
3. Testar upload de 1 imagem e verificar GLB
4. Testar upload de carousel (múltiplas imagens)

**Dia 2 (App Mobile - Integração):**
1. Implementar extração de GLBs em `ar-view.tsx`
2. Testar com payload mockado
3. Integrar com backend real

**Dia 3 (App Mobile - UI):**
1. Criar componente `ARNavigationControls`
2. Integrar controles no ARLauncher
3. Implementar lógica de reload de Scene

**Dia 4 (Polish & Testes):**
1. Testar fluxo completo end-to-end
2. Ajustar animações e transições
3. Performance profiling
4. Correção de bugs

---

## 📚 Referências Técnicas

### Arquivos envolvidos
- `olinxra-backend/main.py` - Endpoints de upload
- `olinxra-backend/glb_generator.py` - Geração de GLB
- `olinxra-app/context/ARPayloadContext.tsx` - Context (já pronto)
- `olinxra-app/app/(tabs)/ar-view.tsx` - Lógica de AR
- `olinxra-app/components/ar/ARLauncher.tsx` - Componente AR (a criar controles)

### Dependências
- Backend: `trimesh`, `pillow`, `firebase-admin`, `motor` (MongoDB)
- App: `expo-gl`, `expo-three`, `react-native-gesture-handler`

---

**Status Final:** Planejamento completo | Pronto para implementação 🚀  
**Próxima ação:** Começar pela FASE 1 (Backend - Pré-geração de GLBs)
