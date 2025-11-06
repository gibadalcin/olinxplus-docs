# 📝 Hierarquia Visual de Blocos de Texto

## 🎯 **Objetivo**

Criar uma apresentação textual **padronizada, harmônica e profissional** para os blocos de conteúdo, respeitando a hierarquia: **Título → Subtítulo → Texto**.

---

## 📊 **Hierarquia Visual (4 Níveis)**

### **Nível 1: Título Principal** (`tipo: "Título"`)
**Função**: Máxima hierarquia visual, título principal da seção/página

**Características visuais**:
```tsx
fontSize: 28
fontWeight: '800' (Extra Bold)
color: '#1a1a1a' (Preto intenso)
lineHeight: 36
letterSpacing: -0.5 (Ajuste ótico)
marginTop: 24px
marginBottom: 16px
```

**Uso**: Título de página, seção principal, destaque máximo

**Exemplo**:
```
"Bem-vindo ao G3 Caxias"
"História do Grêmio"
"Eventos Especiais"
```

---

### **Nível 2: Subtítulo** (`tipo: "Subtítulo"`)
**Função**: Segunda hierarquia, subdivide seções

**Características visuais**:
```tsx
fontSize: 20
fontWeight: '600' (Semi Bold)
color: '#2c2c2c' (Cinza escuro)
lineHeight: 28
letterSpacing: -0.3
marginTop: 20px
marginBottom: 12px
```

**Uso**: Subdivisões de conteúdo, tópicos principais

**Exemplo**:
```
"Nossos Jogadores"
"Próximos Eventos"
"Informações Importantes"
```

---

### **Nível 3: Título de Seção** (`titulo` dentro de `tipo: "Texto"`)
**Função**: Terceira hierarquia, título opcional dentro de blocos de texto

**Características visuais**:
```tsx
fontSize: 17
fontWeight: '600' (Semi Bold)
color: '#333' (Cinza médio)
lineHeight: 24
marginTop: 16px
marginBottom: 8px
```

**Uso**: Títulos internos, labels de parágrafos

**Exemplo**:
```
"Sobre o Evento:"
"Localização:"
"Horários:"
```

---

### **Nível 4: Texto Normal** (`tipo: "Texto"`)
**Função**: Corpo de texto, conteúdo principal

**Características visuais**:
```tsx
fontSize: 16
fontWeight: '400' (Regular)
color: '#555' (Cinza)
lineHeight: 25 (boa legibilidade)
letterSpacing: 0.1
marginBottom: 4px
```

**Uso**: Parágrafos, descrições, conteúdo geral

**Exemplo**:
```
"O G3 é um dos maiores centros culturais e esportivos de Caxias do Sul, oferecendo..."
```

---

## 📐 **Espaçamentos Padronizados**

### **Margens Verticais**

| Elemento | marginTop | marginBottom | Razão |
|----------|-----------|--------------|-------|
| **Título Principal** | 24px | 16px | Espaço generoso para destacar |
| **Subtítulo** | 20px | 12px | Separação clara da seção anterior |
| **Título de Seção** | 16px | 8px | Conecta com o texto abaixo |
| **Texto Normal** | 0px | 4px | Parágrafos fluem naturalmente |

### **Margens Horizontais**

```tsx
textBlock: {
    paddingHorizontal: 20px  // Espaço respiratório lateral
    paddingVertical: 12px    // Espaço vertical interno
}
```

---

## 🎨 **Paleta de Cores (Escala de Cinza)**

```tsx
Título Principal:  #1a1a1a  ████ (Preto intenso - máximo contraste)
Subtítulo:         #2c2c2c  ████ (Cinza escuro - forte presença)
Título de Seção:   #333333  ████ (Cinza médio - moderado)
Texto Normal:      #555555  ████ (Cinza - legibilidade confortável)
```

**Razão**: Escala progressiva cria hierarquia visual clara sem uso de cores vibrantes.

---

## 📱 **Exemplo Visual Completo**

```
┌─────────────────────────────────────┐
│                                     │
│  [24px espaço]                      │
│                                     │
│  📌 Título Principal                 │  ← 28px, 800 weight
│     (Bem-vindo ao G3)               │
│                                     │
│  [16px espaço]                      │
│                                     │
│  [20px espaço]                      │
│                                     │
│  📍 Subtítulo                        │  ← 20px, 600 weight
│     (Nossos Eventos)                │
│                                     │
│  [12px espaço]                      │
│                                     │
│  [16px espaço]                      │
│                                     │
│  🏷️ Título de Seção                  │  ← 17px, 600 weight
│     (Próximo Jogo:)                 │
│                                     │
│  [8px espaço]                       │
│                                     │
│  📄 Texto Normal                     │  ← 16px, 400 weight
│     O G3 enfrenta o Inter no        │
│     próximo domingo às 16h no       │
│     Estádio Centenário...           │
│                                     │
│  [4px espaço]                       │
│                                     │
│  📄 Texto Normal (cont.)             │
│     Ingressos disponíveis na        │
│     bilheteria a partir de...       │
│                                     │
└─────────────────────────────────────┘
```

---

## 🔄 **Ordem de Renderização**

Os blocos são renderizados na **ordem de inserção** no AdminUI:

```tsx
{textBlocks.map((bloco, index) => (
    <BlockRenderer key={`text-${index}`} bloco={bloco} index={index} />
))}
```

**Sequência típica**:
1. **Imagem Topo** (HeaderBlock)
2. **Título Principal** (primeiro bloco de texto)
3. **Subtítulo** (segundo bloco)
4. **Texto Normal** (parágrafos seguintes)
5. **Outros blocos** (carrossel, vídeos, etc.)
6. **Botões** (fixos no bottom)

---

## ✅ **Boas Práticas**

### **✓ Recomendado**

```
1. Título Principal
   ↓ (espaço 16px)
2. Subtítulo
   ↓ (espaço 12px)
3. Texto Normal (parágrafo 1)
   ↓ (espaço 4px)
4. Texto Normal (parágrafo 2)
```

### **✗ Evitar**

```
❌ Múltiplos Títulos Principais seguidos (cria confusão visual)
❌ Texto Normal antes de Título (quebra hierarquia)
❌ Subtítulo sem Título acima (contexto perdido)
```

---

## 🎯 **Responsividade**

### **Ajustes Automáticos**

```tsx
lineHeight: fontSize * 1.28-1.56  // Proporção áurea para legibilidade
letterSpacing: Negativo para títulos (-0.5 a -0.3) → Ótica visual
               Positivo para texto (0.1) → Legibilidade
```

### **Larguras**

```tsx
paddingHorizontal: 20px  // Margens laterais confortáveis
width: 100%              // Blocos ocupam largura total
```

---

## 📊 **Comparação Antes/Depois**

### **❌ ANTES** (Inconsistente)

```tsx
mainTitle: {
    fontSize: 24,
    marginTop: 32,    // Muito espaço
    marginBottom: 8,  // Pouco espaço
}
subtitle: {
    fontSize: 18,     // Diferença pequena vs título
    marginBottom: 8,
}
textContent: {
    fontSize: 15,     // Muito pequeno
    lineHeight: 22,   // Apertado
}
```

**Problemas**:
- Pouca distinção visual entre níveis
- Espaçamentos irregulares
- Texto apertado (lineHeight baixo)

---

### **✅ DEPOIS** (Padronizado)

```tsx
mainTitle: {
    fontSize: 28,      // +17% maior
    marginTop: 24,     // Balanceado
    marginBottom: 16,  // Dobro do anterior
}
subtitle: {
    fontSize: 20,      // +11% - clara distinção
    marginTop: 20,     // Consistente
    marginBottom: 12,
}
textContent: {
    fontSize: 16,      // +7% - mais legível
    lineHeight: 25,    // +14% - respiração
    letterSpacing: 0.1, // Novo!
}
```

**Melhorias**:
- ✅ Hierarquia visual clara (28 → 20 → 17 → 16)
- ✅ Espaçamentos proporcionais
- ✅ Legibilidade otimizada (lineHeight, letterSpacing)
- ✅ Contraste progressivo de cores

---

## 🧪 **Teste de Cenários**

### **Cenário 1: Artigo Longo**
```
Título Principal: "História do G3 Caxias"
Subtítulo: "Fundação e Primeiros Anos"
Texto Normal: "O clube foi fundado em..."
Texto Normal: "Durante a primeira década..."
Subtítulo: "Era Moderna"
Texto Normal: "A partir dos anos 2000..."
```

**Resultado**: ✅ Hierarquia clara, fácil escaneamento visual

---

### **Cenário 2: Lista de Eventos**
```
Título Principal: "Próximos Eventos"
Subtítulo: "Futebol"
Texto (título): "G3 vs Internacional"
Texto Normal: "Data: 15/11/2025..."
Subtítulo: "Basquete"
Texto (título): "Torneio Estadual"
Texto Normal: "Inscrições abertas até..."
```

**Resultado**: ✅ Estrutura organizada, seções distinguíveis

---

### **Cenário 3: Informações Curtas**
```
Título Principal: "Contato"
Texto (título): "Endereço:"
Texto Normal: "Rua Example, 123..."
Texto (título): "Telefone:"
Texto Normal: "(54) 1234-5678"
```

**Resultado**: ✅ Compacto sem perder clareza

---

## 🔧 **Customização (Futuro)**

Se necessário ajustar para temas específicos:

```tsx
// Theme Provider (exemplo)
const textTheme = {
    mainTitle: {
        light: { fontSize: 28, color: '#1a1a1a' },
        dark: { fontSize: 28, color: '#f0f0f0' },
    },
    // ...
}
```

---

## 📋 **Checklist de Implementação**

- [x] Hierarquia de 4 níveis definida
- [x] Espaçamentos padronizados (24→16→12→8→4)
- [x] Tipografia escalada (28→20→17→16)
- [x] Pesos de fonte diferenciados (800→600→600→400)
- [x] Cores em escala de cinza (#1a1a1a→#555)
- [x] lineHeight otimizado (legibilidade)
- [x] letterSpacing para ajuste ótico
- [x] Margens laterais respiratórias (20px)
- [x] Renderização na ordem de inserção
- [x] Sem erros de compilação

---

## 📖 **Referências Tipográficas**

**Escala Modular**: Base 16px × 1.25 (Quarta Perfeita)
```
16px (base)
  ↓ × 1.06 = 17px (título seção)
  ↓ × 1.25 = 20px (subtítulo)
  ↓ × 1.40 = 28px (título principal)
```

**Line Height**: 1.4-1.56× para corpo de texto (legibilidade ideal)

**Letter Spacing**: 
- Títulos grandes: Negativo (-0.5 a -0.3) → Compacta visualmente
- Corpo: Positivo (+0.1) → Areja texto

---

**Data**: 06/11/2025  
**Autor**: Copilot + Gibadalcin  
**Status**: ✅ Implementado
