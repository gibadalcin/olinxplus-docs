# Visualização de Múltiplos Modelos GLB no AR

## Visão Geral

Implementação da funcionalidade de navegação entre múltiplos modelos GLB na tela de Realidade Aumentada do app mobile. Permite que usuários visualizem diferentes modelos 3D de um mesmo conteúdo usando controles de navegação intuitivos.

---

## Arquitetura

### 1. Extração de GLBs dos Blocos

**Arquivo**: `app/(tabs)/ar-view.tsx`

**Estados adicionados**:
```typescript
const [glbModels, setGlbModels] = useState<Array<{ url: string; blockIndex: number }>>([]);
const [currentModelIndex, setCurrentModelIndex] = useState(0);
```

**Lógica de extração** (linhas ~305-363):
- Executa quando `payload` muda
- Normaliza estrutura de blocos (`payload.blocos` ou `payload.blocos.blocos`)
- Itera por cada bloco procurando por `glb_signed_url` ou `glb_url`
- Também verifica itens de carousel (`bloco.items[]`)
- Prioridade: `glb_signed_url` > `glb_url` (URLs assinadas não expiram)
- Armazena array de modelos com `{ url, blockIndex }`

**Logs de debug**:
```
[ARView] 🔍 Extraindo GLBs dos blocos...
[ARView] 📊 Blocos encontrados: 5
[ARView] ✅ GLB encontrado no bloco 2: https://...
[ARView] ✅ GLB encontrado no item 1 do bloco 3: https://...
[ARView] 🎯 Total de GLBs encontrados: 3
```

---

### 2. Seleção do Modelo Atual

**Arquivo**: `app/(tabs)/ar-view.tsx`

**Modificação no `finalModelUrl`** (linhas ~366-390):

**Prioridades** (ordem de verificação):
1. **GLB dos blocos** (`glbModels[currentModelIndex]`) - NOVO ✅
2. GLB gerado dinamicamente (`generatedGlbUrl`)
3. Modelo no payload principal (`findModelUrl(payload)`)

```typescript
const finalModelUrl = useMemo(() => {
    // PRIORIDADE 1: Modelo GLB dos blocos (array glbModels)
    if (glbModels.length > 0 && currentModelIndex < glbModels.length) {
        const selectedModel = glbModels[currentModelIndex];
        console.log('[ARView] ✅ Usando GLB do bloco', selectedModel.blockIndex, 
                    `(${currentModelIndex + 1}/${glbModels.length})`);
        return selectedModel.url;
    }
    
    // Fallbacks...
}, [glbModels, currentModelIndex, generatedGlbUrl, payload, findModelUrl]);
```

---

### 3. Componente de Navegação

**Arquivo**: `components/ar/ARNavigationControls.tsx` (NOVO)

**Props**:
```typescript
interface ARNavigationControlsProps {
    currentIndex: number;      // Índice do modelo atual (0-based)
    totalModels: number;        // Total de modelos disponíveis
    onPrevious: () => void;     // Callback para modelo anterior
    onNext: () => void;         // Callback para próximo modelo
}
```

**Comportamento**:
- Não renderiza se `totalModels <= 1` (apenas 1 modelo)
- Desabilita botão "Anterior" quando `currentIndex === 0`
- Desabilita botão "Próximo" quando `currentIndex === totalModels - 1`
- Exibe contador `"2/5"` (modelo atual / total)

**Visual**:
```
┌──────────────────────────────────┐
│   ◀   │   2/5    │   ▶           │
│  Prev │ Modelos  │  Next          │
│       │    AR    │                │
└──────────────────────────────────┘
```

**Estilos**:
- Background: `rgba(0, 0, 0, 0.7)` (semi-transparente)
- Botões: circulares, 44x44px, azul (#3498db)
- Botões desabilitados: cinza (#555), opacity 0.5
- Contador: texto branco (20px) + label cinza (11px)

---

### 4. Funções de Navegação

**Arquivo**: `app/(tabs)/ar-view.tsx` (linhas ~989-1002)

```typescript
const handlePreviousModel = useCallback(() => {
    if (currentModelIndex > 0) {
        console.log('[ARView] ⬅️ Navegando para modelo anterior:', currentModelIndex - 1);
        setCurrentModelIndex(prev => prev - 1);
    }
}, [currentModelIndex]);

const handleNextModel = useCallback(() => {
    if (currentModelIndex < glbModels.length - 1) {
        console.log('[ARView] ➡️ Navegando para próximo modelo:', currentModelIndex + 1);
        setCurrentModelIndex(prev => prev + 1);
    }
}, [currentModelIndex, glbModels.length]);
```

**Funcionamento**:
1. Verifica limites do array antes de navegar
2. Atualiza `currentModelIndex` via setState
3. `finalModelUrl` recomputa automaticamente (useMemo)
4. ARLauncher recebe novo modelo via props

---

### 5. Integração no Render

**Arquivo**: `app/(tabs)/ar-view.tsx` (linhas ~1424-1436)

**Posição no layout**:
```
CustomHeader
└── fullScreenContainer
    ├── Mensagem/Status (center)
    ├── ARNavigationControls (NOVO) ✅
    └── ARLauncher (botão "Ver em RA")
```

**Código**:
```tsx
{/* ✅ NOVO: Controles de navegação entre modelos */}
{glbModels.length > 1 && (
    <ARNavigationControls
        currentIndex={currentModelIndex}
        totalModels={glbModels.length}
        onPrevious={handlePreviousModel}
        onNext={handleNextModel}
    />
)}

<ARLauncher isReady={!finalModelUrl || isReady} ... />
```

**Renderização condicional**:
- `glbModels.length > 1`: mostra controles apenas se há múltiplos modelos
- `glbModels.length === 0 ou 1`: oculta controles completamente

---

## Fluxo de Uso

### Cenário 1: Conteúdo com 3 GLBs

**Setup inicial**:
1. Usuário captura logo da marca
2. Backend retorna payload com 3 blocos, cada um com `glb_signed_url`
3. App extrai GLBs: `glbModels = [{ url: "glb1", blockIndex: 0 }, ...]`
4. `currentModelIndex = 0` (primeiro modelo)

**Navegação**:
```
Estado Inicial:
- Exibe: "1/3 Modelos AR"
- Botão ◀ (desabilitado)
- Botão ▶ (ativo)

Usuário clica ▶:
- currentModelIndex: 0 → 1
- finalModelUrl atualiza para glbModels[1].url
- Exibe: "2/3 Modelos AR"
- Ambos botões ativos

Usuário clica ▶ novamente:
- currentModelIndex: 1 → 2
- finalModelUrl atualiza para glbModels[2].url
- Exibe: "3/3 Modelos AR"
- Botão ▶ (desabilitado)
- Botão ◀ (ativo)

Usuário clica "Ver em RA":
- ARLauncher lança AR com glbModels[2].url
- Scene Viewer (Android) ou AR Quick Look (iOS) abre
```

### Cenário 2: Conteúdo com 1 GLB

**Comportamento**:
- `glbModels.length === 1`
- Controles de navegação **não são renderizados**
- ARLauncher funciona normalmente com modelo único
- UX clean, sem elementos desnecessários

### Cenário 3: Conteúdo sem GLBs

**Comportamento**:
- `glbModels.length === 0`
- Controles de navegação **não são renderizados**
- Fallback para GLB gerado dinamicamente (`generatedGlbUrl`)
- Se nem GLB gerado existe, mostra mensagem padrão

---

## Estrutura de Dados

### Payload de Blocos

**Estrutura esperada do backend**:
```json
{
  "nome_marca": "Coca-Cola",
  "blocos": [
    {
      "tipo": "imagem_topo",
      "subtipo": "header",
      "url": "https://storage/image1.png",
      "signed_url": "https://storage/image1.png?token=...",
      "glb_url": "https://storage/totem1.glb",
      "glb_signed_url": "https://storage/totem1.glb?token=..."  ← EXTRAÍDO
    },
    {
      "tipo": "carousel",
      "items": [
        {
          "url": "https://storage/carousel1.png",
          "signed_url": "https://storage/carousel1.png?token=...",
          "glb_url": "https://storage/carousel1.glb",
          "glb_signed_url": "https://storage/carousel1.glb?token=..."  ← EXTRAÍDO
        }
      ]
    }
  ]
}
```

### Array de Modelos (Estado Interno)

```typescript
glbModels = [
  { url: "https://storage/totem1.glb?token=...", blockIndex: 0 },
  { url: "https://storage/carousel1.glb?token=...", blockIndex: 1 }
]
currentModelIndex = 0  // Primeiro modelo selecionado
```

---

## Testes Necessários

### 1. Teste Manual - Múltiplos Modelos

**Preparação**:
1. Criar conteúdo no AdminUI com 3 blocos
2. Fazer upload de GLB customizado para cada bloco
3. Publicar conteúdo

**Execução no App**:
1. Capturar logo da marca
2. Verificar que controles aparecem: "1/3 Modelos AR"
3. Clicar ▶, verificar mudança para "2/3"
4. Clicar "Ver em RA", confirmar que AR abre com modelo correto
5. Fechar AR, clicar ◀, verificar mudança para "1/3"
6. Clicar "Ver em RA", confirmar que AR abre com primeiro modelo

**Verificações**:
- [ ] Contador atualiza corretamente
- [ ] Botões habilitam/desabilitam nos limites
- [ ] AR lança com modelo correto
- [ ] Navegação fluida sem travamentos

### 2. Teste Manual - Modelo Único

**Preparação**:
1. Criar conteúdo com apenas 1 bloco + GLB

**Execução**:
1. Capturar logo
2. Verificar que controles **NÃO aparecem**
3. ARLauncher funciona normalmente

### 3. Teste Manual - Sem GLBs

**Preparação**:
1. Criar conteúdo sem GLBs customizados

**Execução**:
1. Capturar logo
2. Verificar que controles **NÃO aparecem**
3. Fallback para geração automática funciona

### 4. Teste de Regressão

**Verificar que funcionalidades antigas continuam funcionando**:
- [ ] Geração automática de GLB (quando sem GLB customizado)
- [ ] Exibição de conteúdo após fechar AR
- [ ] Navegação de volta ao Explorer
- [ ] Auto-launch de AR (quando vindo do Explorer)

---

## Logs de Debug

### Extração de GLBs

```
[ARView] 🔍 Extraindo GLBs dos blocos...
[ARView] 📊 Blocos encontrados: 3
[ARView] ✅ GLB encontrado no bloco 0: https://storage.googleapis.com/...totem1.glb?token=...
[ARView] ✅ GLB encontrado no bloco 1: https://storage.googleapis.com/...totem2.glb?token=...
[ARView] 🎯 Total de GLBs encontrados: 2
```

### Seleção de Modelo

```
[ARView] 🔍 Buscando modelo final...
[ARView] ✅ Usando GLB do bloco 1 (2/2)
[ARView] 📊 URL: https://storage.googleapis.com/...totem2.glb?token=...
```

### Navegação

```
[ARView] ➡️ Navegando para próximo modelo: 1
[ARView] ⬅️ Navegando para modelo anterior: 0
```

---

## Possíveis Melhorias Futuras

1. **Preview de Modelos**:
   - Thumbnail de cada modelo nos controles
   - Carrossel horizontal de previews
   
2. **Indicador de Carregamento**:
   - Spinner ao trocar modelos
   - Preload do próximo modelo
   
3. **Gestos de Navegação**:
   - Swipe horizontal para trocar modelos
   - Melhor UX mobile
   
4. **Informações Contextuais**:
   - Nome/descrição do modelo atual
   - Origem do bloco (topo, carousel, etc)
   
5. **Persistência**:
   - Lembrar último modelo visualizado
   - Restaurar posição ao voltar para tela

---

## Arquivos Modificados

1. **app/(tabs)/ar-view.tsx**:
   - Adicionados estados `glbModels` e `currentModelIndex`
   - Novo useEffect para extração de GLBs
   - Modificado `finalModelUrl` (prioridade para array)
   - Funções `handlePreviousModel` e `handleNextModel`
   - Integração de `ARNavigationControls` no render

2. **components/ar/ARNavigationControls.tsx** (NOVO):
   - Componente completo de navegação
   - Props, estilos, lógica de habilitação

3. **components/ar/index.ts**:
   - Export de `ARNavigationControls`

---

## Dependências

- React Native (View, Text, TouchableOpacity, StyleSheet)
- Expo Image (Image)
- Estados React (useState, useMemo, useCallback)
- Context API (useARPayload)

---

## Compatibilidade

- ✅ Android (Scene Viewer)
- ✅ iOS (AR Quick Look)
- ✅ Sem AR (graceful degradation)

---

## Conclusão

A implementação permite navegação fluida entre múltiplos modelos GLB customizados, mantendo compatibilidade total com o fluxo existente. O design é progressivo: se há múltiplos modelos, mostra controles; se há apenas um, comportamento padrão; se não há nenhum, fallback para geração automática.

**Status**: ✅ Implementado, pronto para testes
