# Olinx Plus

<div align="center">

**Plataforma de Realidade Aumentada com Reconhecimento Visual de Logos**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Backend](https://img.shields.io/badge/backend-FastAPI-009688.svg)](https://github.com/gibadalcin/olinxplus-backend)
[![Frontend](https://img.shields.io/badge/admin-React%20%2B%20Vite-61DAFB.svg)](https://github.com/gibadalcin/olinxplus-adminui)
[![Mobile](https://img.shields.io/badge/app-Expo%20%2B%20React%20Native-000020.svg)](https://github.com/gibadalcin/olinxplus)

[Sobre](#-sobre) • [Funcionalidades](#-funcionalidades) • [Arquitetura](#-arquitetura) • [Começando](#-começando) • [Documentação](#-documentação)

</div>

---

## 📋 Sobre

**Olinx Plus** é uma plataforma completa de Realidade Aumentada que permite às marcas criar experiências interativas através do reconhecimento visual de logos. O sistema combina tecnologias avançadas de visão computacional (CLIP + FAISS), armazenamento em nuvem (Google Cloud Storage) e realidade aumentada (WebGL/Three.js) para proporcionar experiências imersivas aos usuários finais.

### 🎯 Diferenciais

- **Reconhecimento Visual Inteligente**: Usa CLIP (OpenAI) para embeddings de alta precisão
- **Busca Vetorial Ultra-Rápida**: FAISS para busca sublinear em milhões de logos
- **Crop Adaptativo**: App mobile envia imagens já pré-processadas para o backend
- **Geolocalização Inteligente**: Conteúdo contextual baseado em proximidade e região
- **Modo Offline**: Cache local de logos para reconhecimento sem conexão
- **AR Nativo**: Visualização de modelos 3D com WebGL/Three.js

### Casos de Uso

- **Marketing Interativo**: Marcas podem criar campanhas de AR vinculadas aos seus logos
- **Eventos e Experiências**: Conteúdo exclusivo de AR para eventos corporativos
- **Gamificação**: Experiências gamificadas através de reconhecimento de marca
- **Educação**: Material educativo interativo vinculado a produtos

## ✨ Funcionalidades

### Para Administradores (Admin UI)
- 🎨 **Gestão de Conteúdo**: Interface intuitiva para criar e editar blocos de conteúdo AR
- 🖼️ **Upload de Mídia**: Suporte para imagens, vídeos e modelos 3D (GLB)
- 🎪 **Carrosséis Dinâmicos**: Criação de carrosséis de imagens com ações personalizadas
- 🗺️ **Geolocalização**: Definição de raio de alcance para conteúdo baseado em localização
- 🏷️ **Gestão de Logos**: Upload e indexação de logos de marcas para reconhecimento visual

### Para Usuários Finais (Mobile App)
- 📷 **Captura Inteligente**: Detecção automática de logos através da câmera
- 🔍 **Reconhecimento Visual**: Busca por similaridade usando CLIP e FAISS
- 🌟 **Visualização AR**: Experiência de realidade aumentada com modelos 3D
- 📍 **Conteúdo Contextual**: Exibição de conteúdo baseado em localização e marca
- 💾 **Modo Offline**: Cache de logos para funcionamento sem conexão

### Backend (API)
- ⚡ **API REST**: FastAPI de alta performance
- 🤖 **IA de Reconhecimento**: CLIP (OpenAI) para embedding de imagens
- 🔎 **Busca Vetorial**: FAISS para busca eficiente por similaridade
- ☁️ **Cloud Storage**: Google Cloud Storage para mídia
- 🗄️ **Banco de Dados**: MongoDB para dados estruturados
- 🔐 **Autenticação**: Firebase Authentication

## 🔄 Fluxo de Uso

### Pipeline de Reconhecimento e Visualização

```
1. Usuário abre câmera no app
   ↓
2. Tira foto do logo OU seleciona da galeria
   ↓
3. Modal de decisão exibido:
   • Se foto capturada: "Buscar conteúdo" | "Salvar na galeria" | "Cancelar"
   • Se da galeria: "Buscar conteúdo" | "Cancelar"
   ↓
4. Usuário escolhe "Buscar conteúdo"
   ↓
5. Imagem redimensionada (max 800px)
   ↓
6. Enviada para API (Base64)
   ↓
7. Backend: Gera CLIP embedding da imagem
   ↓
8. Backend: FAISS busca similar no índice de logos
   ↓
9. Melhor resultado retornado (top-1 match)
   ↓
10. App carrega conteúdo associado automaticamente
    ↓
11. Exibe conteúdo em tela comum (imagens, textos, botões)
    ↓
12. Usuário visualiza conteúdo e pode:
    • Interagir com botões/links
    • Visualizar modelos 3D/AR (se disponível e dispositivo suportar)
    • Navegar pelos carrosséis de imagens
```

<!-- SCREENSHOT: Fluxo completo de reconhecimento -->

## 🏗️ Arquitetura

### Estrutura de Repositórios

```
olinxplus/               # 📱 Mobile App (Expo + React Native)
├── app/                 # Navegação e telas (Expo Router)
│   ├── _layout.tsx      # Layout raiz com providers
│   ├── index.tsx        # Tela splash inicial
│   ├── GlobalSplashOverlay.tsx  # Overlay animado de splash
│   └── _tabs/           # Navegação em abas
│       ├── recognizer/  # Tela de captura e reconhecimento
│       ├── ar-view.tsx  # Visualização AR com modelos 3D
│       ├── explorer.tsx # Exploração de conteúdos
│       ├── help.tsx     # Ajuda e suporte
│       └── options.tsx  # Configurações
├── components/
│   ├── ar/              # Componentes de AR (ARLauncher, ARPreviewViewer)
│   └── ui/              # UI components (CameraMarkers, ImageDecisionModal)
├── context/             # Contexts React
│   ├── ARPayloadContext.tsx        # Estado de conteúdo AR
│   ├── CaptureSettingsContext.tsx  # Configurações de captura
│   └── SplashFadeContext.tsx       # Controle de splash
├── hooks/               # Custom hooks
│   ├── useARContent.ts  # Busca de conteúdo AR
│   ├── useLogoCompare.ts # Comparação de logos
│   └── useLogoCache.ts  # Cache offline de logos
└── utils/               # Utilitários e helpers

olinxplus-adminui/       # 🎨 Admin Dashboard (React + Vite)
├── src/
│   ├── pages/
│   │   ├── Content.jsx  # Editor de conteúdo AR
│   │   ├── Dashboard.jsx # Dashboard principal
│   │   ├── ImageManager.jsx # Gestão de imagens
│   │   └── Login.jsx    # Autenticação
│   ├── components/
│   │   ├── contentContext/ # Componentes de edição de conteúdo
│   │   ├── globalContext/  # Componentes globais
│   │   └── imageContext/   # Gestão de imagens
│   ├── hooks/
│   │   ├── useBlocos.js    # Estado de blocos de conteúdo
│   │   ├── useMarcas.js    # Gestão de marcas
│   │   └── useImages.js    # Upload e gestão de imagens
│   └── utils/
│       ├── fileUtils.js    # Utilitários de arquivo
│       └── uploadHelper.js # Helper de upload
└── public/

olinxplus-backend/       # ⚡ API Backend (FastAPI + Python)
├── main.py              # Entrypoint da API (4146 linhas)
├── schemas.py           # Modelos Pydantic
├── firebase_utils.py    # Firebase Authentication
├── gcs_utils.py         # Google Cloud Storage
├── clip_utils.py        # CLIP embeddings (ONNX)
├── faiss_index.py       # Busca vetorial FAISS
├── glb_generator.py     # Geração de modelos GLB
├── tools/               # Scripts utilitários
│   ├── preprocess_variants.py  # Pré-processamento de imagens
│   └── generate_glbs_from_existing_images.py
├── docs/
│   └── CROP-OPTIMIZATION.md  # Documentação de otimizações
└── debug/               # Scripts de debug e análise

olinxplus-docs/          # 📚 Documentação central
└── docs/
    ├── ARQUITETURA-STORAGE.md        # Arquitetura de armazenamento
    ├── CAMADAS-DE-ACESSO.md          # Controle de acesso
    ├── MULTIPLOS-MODELOS-AR.md       # Múltiplos modelos 3D
    ├── SINCRONIZACAO-DELECAO-GLB.md  # Sync de GLBs
    ├── THRESHOLD_CALIBRATION.md      # Calibração de thresholds
    └── LOADING_IMPROVEMENTS.md       # Otimizações de loading
```

### Stack Tecnológico

**Frontend (Admin UI)**
- React 19 + Vite 6
- Material-UI (MUI) v7
- React Router v7
- Leaflet 1.9.4 (mapas interativos)
- Firebase SDK 11.2.0
- Recharts (visualização de dados)

**Mobile (App)**
- Expo SDK 54
- React Native 0.76.5
- React 19.0.0
- Expo Router (navegação file-based)
- Expo GL + React Three Fiber (renderização 3D/AR)
- expo-image-manipulator (crop de imagens)
- Firebase SDK 11.2.0
- AsyncStorage (cache offline)

**Backend (API)**
- FastAPI 0.115.6
- Python 3.11+
- ONNX Runtime 1.16+ (inferência CLIP)
- FAISS-CPU 1.8+ (busca vetorial IVF)
- Motor 3.3+ (MongoDB assíncrono)
- Firebase Admin SDK 6.4+
- Google Cloud Storage 2.14+
- Pillow (pré-processamento de imagens)
- imagehash (pHash para similaridade estrutural)

**Infraestrutura**
- **Firebase**: Authentication + Firebase Storage (conteúdo temporário)
- **Google Cloud Storage**: Bucket `olinxra-conteudo` (imagens) e `olinxra-logos` (logos)
- **MongoDB Atlas**: Cluster M10+ (coleções: blocos, conteudos, marcas, logos)
- **Digital Ocean App Platform**: Hospedagem do backend FastAPI
- **Expo EAS**: Build e publicação do app mobile

## 🚀 Começando

### Pré-requisitos

- **Node.js** 20+ (para AdminUI e App)
- **Python** 3.11+ (para Backend)
- **Firebase** project configurado (Auth + Storage)
- **Google Cloud Storage** buckets: `olinxra-conteudo` e `olinxra-logos`
- **MongoDB Atlas** (ou instância local para desenvolvimento)
- **Expo CLI** (`npm install -g expo-cli`)
- **Digital Ocean** account (para deploy do backend)

### Instalação Rápida

#### 1. **Mobile App (olinxplus)**
```bash
git clone https://github.com/gibadalcin/olinxplus.git
cd olinxplus
npm install
cp firebaseConfig.ts.example firebaseConfig.ts
# Configure Firebase credentials em firebaseConfig.ts
npx expo start
```

#### 2. **Backend API (olinxplus-backend)**
```bash
git clone https://github.com/gibadalcin/olinxplus-backend.git
cd olinxplus-backend
pip install -r requirements.txt
# Configure credenciais Firebase e GCS:
# - firebase-cred.json (Firebase Admin)
# - cloud-storage-cred.json (Google Cloud Storage)
# - .env com parâmetros de reconhecimento
python main.py
# API rodará em http://localhost:8000
```

#### 3. **Admin UI (olinxplus-adminui)**
```bash
git clone https://github.com/gibadalcin/olinxplus-adminui.git
cd olinxplus-adminui
npm install
# Configure API endpoint em .env ou vite config
npm run dev
# Admin UI rodará em http://localhost:5173
```

### Configuração de Credenciais

#### Firebase (firebase-cred.json)
Baixe o arquivo JSON do Console Firebase (Project Settings → Service Accounts → Generate New Private Key) e salve como:
- `olinxplus-backend/firebase-cred.json`

#### Google Cloud Storage (cloud-storage-cred.json)
Crie uma service account no GCP (IAM & Admin → Service Accounts) com permissões `Storage Object Admin` e salve como:
- `olinxplus-backend/cloud-storage-cred.json`

#### MongoDB Atlas
Configure a connection string no `.env` do backend:
```env
MONGODB_URL=mongodb+srv://usuario:senha@cluster.mongodb.net/olinxplus
```

#### Digital Ocean (Deploy Backend)
1. Crie novo App no Digital Ocean App Platform
2. Conecte o repositório `olinxplus-backend`
3. Configure environment variables no painel (Firebase, GCS, MongoDB)
4. Deploy automático a cada push na branch principal

### Documentação Detalhada

Consulte os READMEs específicos de cada repositório:
- [Backend Setup](https://github.com/gibadalcin/olinxplus-backend/blob/main/README.md)
- [Admin UI Setup](https://github.com/gibadalcin/olinxplus-adminui/blob/main/README.md)
- [Mobile App Setup](https://github.com/gibadalcin/olinxplus/blob/main/README.md)

## 📚 Documentação

### 📖 Documentos Técnicos (olinxplus-docs/docs)

**Arquitetura e Design**
- [Arquitetura de Storage](docs/ARQUITETURA-STORAGE.md) - Estrutura de armazenamento de conteúdo
- [Camadas de Acesso](docs/CAMADAS-DE-ACESSO.md) - Sistema de permissões e controle de acesso
- [Hierarquia Visual/Texto](docs/HIERARQUIA-VISUAL-TEXTO.md) - Estrutura de conteúdo
- [Header Preview Spec](docs/HEADER_PREVIEW_SPEC.md) - Especificação de preview de headers

**Funcionalidades AR**
- [Múltiplos Modelos AR](docs/MULTIPLOS-MODELOS-AR.md) - Gestão de múltiplos modelos 3D
- [Sincronização e Deleção GLB](docs/SINCRONIZACAO-DELECAO-GLB.md) - Sync de modelos 3D
- [Plano Carrossel AR](docs/PLANO-CARROSSEL-AR.md) - Implementação de carrosséis AR
- [Geração Automática GLB](docs/GERACAO-AUTOMATICA-GLB.md) - Geração automática de modelos
- [Acesso GLB App](docs/ACESSO-GLB-APP.md) - Como o app acessa modelos 3D

**Otimizações e Performance**
- [Calibração de Threshold](docs/THRESHOLD_CALIBRATION.md) - Calibração de reconhecimento
- [Loading Improvements](docs/LOADING_IMPROVEMENTS.md) - Otimizações de carregamento
- [Correção Delay Imagem](docs/CORRECAO-DELAY-IMAGEM.md) - Correções de delay

**Changelogs e Testes**
- [Changelog Carrossel AR](docs/CHANGELOG-CARROSSEL-AR.md)
- [Teste Carrossel GLB](docs/TESTE-CARROSSEL-GLB.md)
- [Teste Fluxo AR](docs/TESTE-FLUXO-AR.md)
- [Histórico AR Android](docs/HISTORICO-AR-ANDROID.md)

### 🔧 Guias Específicos de Repositório

**olinxplus-backend**
- [README Backend](https://github.com/gibadalcin/olinxplus-backend/blob/main/README.md) - Setup e deployment
- [Crop Optimization](https://github.com/gibadalcin/olinxplus-backend/blob/main/docs/CROP-OPTIMIZATION.md) - Otimizações de reconhecimento

**olinxplus-adminui**
- [Upload GLB Frontend](https://github.com/gibadalcin/olinxplus-adminui/blob/main/UPLOAD-GLB-FRONTEND.md) - Upload de modelos 3D
- [Esquema AR](https://github.com/gibadalcin/olinxplus-adminui/blob/main/AR_SCHEMA.md) - Schema de conteúdo AR
- [Endpoints API](https://github.com/gibadalcin/olinxplus-adminui/blob/main/ENDPOINTS.md) - Documentação da API
**olinxplus (mobile)**
- [README App](https://github.com/gibadalcin/olinxplus/blob/main/README.md) - Configuração e build
- [Correção Delay Imagem](https://github.com/gibadalcin/olinxplus/blob/main/CORRECAO-DELAY-IMAGEM.md) - Correções de UX
- [Teste Fluxo AR](https://github.com/gibadalcin/olinxplus/blob/main/TESTE-FLUXO-AR.md) - Validação de AR
- [Histórico AR Android](https://github.com/gibadalcin/olinxplus/blob/main/HISTORICO-AR-ANDROID.md) - Evolução AR

## 🔑 Funcionalidades Principais

### 📸 Reconhecimento Visual de Logos
- **Captura Guiada**: Marcadores visuais (300x250px) para enquadramento preciso
- **Crop Inteligente**: App processa imagem antes de enviar (reduz latência)
- **CLIP Embeddings**: Vetores de 512 dimensões para busca semântica
- **FAISS IVF**: Índice otimizado para busca sublinear (~15ms em 10k+ logos)
- **pHash Híbrido**: Combinação de similaridade estrutural (20%) + semântica (80%)
- **Thresholds Calibrados**: 
  - Combined Score: 0.50 (aceitação)
  - Min Margin: 0.01 (diferença entre top-1 e top-2)
  - Acceptance: 0.72 (confiança alta)

### 🎨 Gestão de Conteúdo AR
- **Editor Visual**: Admin UI com preview em tempo real
- **Blocos Dinâmicos**: Textos, imagens, botões, carrosséis, modelos 3D
- **Upload Multi-formato**: Suporte a JPEG, PNG, GLB/GLTF
- **Geração Automática de GLB**: Conversão de imagens para modelos 3D planos
- **Versionamento**: Histórico de mudanças em blocos de conteúdo

### 🥽 Realidade Aumentada Nativa
- **AR Core/Kit**: Suporte nativo para Android e iOS
- **Múltiplos Modelos**: Até 10 modelos 3D por conteúdo
- **Carrossel AR**: Navegação entre modelos com gestos
- **Fallback Inteligente**: Modo 3D quando AR não disponível
- **Preview de Assets**: Visualização antes do download completo

### 🌐 Sistema Multi-tenant
- **Marcas**: Isolamento de conteúdo por marca/cliente
- **Geolocalização**: Conteúdo restrito por coordenadas geográficas
- **Controle de Acesso**: Camadas de permissão (admin, editor, viewer)
- **API RESTful**: Endpoints seguros com Firebase Auth

### 📦 Performance e Cache
- **Modo Offline**: Cache de logos e conteúdos visualizados
- **Lazy Loading**: Carregamento progressivo de assets pesados
- **Compressão Adaptativa**: Qualidade ajustada por banda/dispositivo
- **CDN**: Google Cloud Storage como CDN global

## 🏗️ Deploy e Infraestrutura

### Digital Ocean (Backend)
```yaml
# app.yaml (exemplo)
name: olinxplus-api
region: nyc
services:
  - name: api
    source_dir: /
    github:
      repo: gibadalcin/olinxplus-backend
      branch: main
    build_command: pip install -r requirements.txt
    run_command: uvicorn main:app --host 0.0.0.0 --port 8080
    envs:
      - key: MONGODB_URL
        scope: RUN_TIME
        type: SECRET
      - key: SEARCH_COMBINED_THRESHOLD
        value: "0.50"
```

### Expo EAS (Mobile)
```json
// eas.json
{
  "build": {
    "production": {
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleRelease"
      },
      "ios": {
        "buildConfiguration": "Release"
      }
    }
  }
}
```

```bash
# Build e publicação
eas build --platform android --profile production
eas submit -p android
```

### Vite (Admin UI)
```bash
# Build para produção
npm run build
# Deploy para Vercel/Netlify
vercel --prod
```

## 🔧 Desenvolvimento

### Estrutura de Branches

- `main` - Produção estável
- `develop` - Desenvolvimento ativo
- `feature/*` - Novas funcionalidades
- `fix/*` - Correções de bugs

### Scripts Úteis (Backend)

```bash
# Desenvolvimento local
cd olinxplus-backend
python main.py  # API em http://localhost:8000

# Reindexar logos FAISS
python faiss_index.py

# Gerar GLBs automaticamente
python generate_carousel_glbs.py

# Verificar conteúdos no MongoDB
python verify_conteudos.py

# Processar deleções pendentes
python process_pending_deletes.py
```

### Scripts Úteis (Admin UI)

```bash
cd olinxplus-adminui
npm run dev      # Desenvolvimento (http://localhost:5173)
npm run build    # Build para produção
npm run preview  # Preview do build
```

### Scripts Úteis (Mobile)

```bash
cd olinxplus
npx expo start   # Desenvolvimento
npx expo start --clear  # Limpar cache
eas build --platform android --profile production  # Build produção
eas build --platform ios --profile production
```

## 🤝 Contribuindo

Contribuições são bem-vindas! Por favor, siga estes passos:

1. Fork o repositório desejado (olinxplus, olinxplus-backend, ou olinxplus-adminui)
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças seguindo [Conventional Commits](https://www.conventionalcommits.org/)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request com descrição detalhada

### Convenções de Código

**TypeScript/JavaScript**
- ESLint + Prettier configurados
- Naming: camelCase para variáveis/funções, PascalCase para componentes
- Use TypeScript types sempre que possível

**Python**
- PEP 8 style guide
- Type hints obrigatórios
- Docstrings para funções públicas

## 📄 Licença

Este projeto está sob a licença MIT. Veja os arquivos LICENSE em cada repositório para mais detalhes:
- [olinxplus/LICENSE](https://github.com/gibadalcin/olinxplus/blob/main/LICENSE)
- [olinxplus-backend/LICENSE](https://github.com/gibadalcin/olinxplus-backend/blob/main/LICENSE)
- [olinxplus-adminui/LICENSE](https://github.com/gibadalcin/olinxplus-adminui/blob/main/LICENSE)

## 👥 Equipe

**Desenvolvido por Gibanet Tecnologia** 🚀

Projeto iniciado em 2024, com foco em democratizar acesso a tecnologias de AR e reconhecimento visual.

## 📞 Suporte e Contato

Para suporte técnico e questões:
- 💬 Issues: [Abra uma issue](https://github.com/gibadalcin) no repositório correspondente
- 📧 Email: contato@gibanet.com.br
- 🌐 Site: [gibanet.com.br](https://gibanet.com.br)

### Status dos Serviços

- **Backend API**: Digital Ocean App Platform (uptime 99.9%)
- **Storage**: Google Cloud Storage (redundância regional)
- **Database**: MongoDB Atlas M10 (backup automático diário)
- **Mobile**: Expo EAS (builds automatizados)

---

<div align="center">

**Olinx Plus** - Conectando o mundo físico ao digital através de AR 🎯

Feito com ❤️ usando React, FastAPI, e CLIP

[![GitHub](https://img.shields.io/badge/GitHub-gibadalcin-black?logo=github)](https://github.com/gibadalcin)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

</div>
