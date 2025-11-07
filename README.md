# OlinxRA

<div align="center">

**Plataforma de Realidade Aumentada para Reconhecimento e Visualização de Marcas**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Backend](https://img.shields.io/badge/backend-FastAPI-009688.svg)](olinxra-backend/)
[![Frontend](https://img.shields.io/badge/admin-React%20%2B%20Vite-61DAFB.svg)](olinxra-adminui/)
[![Mobile](https://img.shields.io/badge/app-Expo%20%2B%20React%20Native-000020.svg)](olinxra-app/)

[Sobre](#-sobre) • [Funcionalidades](#-funcionalidades) • [Arquitetura](#-arquitetura) • [Começando](#-começando) • [Documentação](#-documentação)

</div>

---

## 📋 Sobre

OlinxRA é uma plataforma completa de Realidade Aumentada que permite às marcas criar experiências interativas através do reconhecimento visual de logos. O sistema combina tecnologias de visão computacional, armazenamento em nuvem e realidade aumentada para proporcionar experiências imersivas aos usuários finais.

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

```
OlinxRA/
├── olinxra-app/          # Mobile App (Expo + React Native)
│   ├── app/              # Navegação e telas (Expo Router)
│   ├── components/       # Componentes reutilizáveis
│   ├── hooks/            # Custom hooks (AR, logo cache, etc)
│   └── utils/            # Utilitários e helpers
│
├── olinxra-adminui/      # Admin Dashboard (React + Vite)
│   ├── src/
│   │   ├── pages/        # Páginas principais (Content, Logos)
│   │   ├── components/   # Componentes UI
│   │   └── hooks/        # Hooks de estado (useBlocos, etc)
│   └── public/           # Assets estáticos
│
├── olinxra-backend/      # API Backend (FastAPI + Python)
│   ├── main.py           # Entrypoint da API
│   ├── schemas.py        # Modelos Pydantic
│   ├── firebase_utils.py # Integração Firebase
│   ├── gcs_utils.py      # Google Cloud Storage
│   ├── clip_utils.py     # CLIP embeddings
│   └── faiss_index.py    # Índice FAISS
│
├── docs/                 # Documentação técnica
└── modelos-3d/           # Modelos 3D de exemplo
```

### Stack Tecnológico

**Frontend (Admin UI)**
- React 19 + Vite
- Material-UI (MUI)
- React Router
- Leaflet (mapas)
- Firebase SDK

**Mobile (App)**
- Expo 54
- React Native
- Expo Router
- Expo GL (WebGL)
- React Three Fiber
- Firebase SDK

**Backend (API)**
- FastAPI
- Python 3.11+
- ONNX Runtime (CLIP)
- FAISS (busca vetorial)
- Motor (MongoDB async)
- Firebase Admin SDK
- Google Cloud Storage

**Infraestrutura**
- Firebase (Auth + Storage)
- Google Cloud Storage
- MongoDB Atlas
- DigitalOcean / Cloud Provider

## 🚀 Começando

### Pré-requisitos

- **Node.js** 18+ (para AdminUI e App)
- **Python** 3.11+ (para Backend)
- **Firebase** project configurado
- **Google Cloud Storage** bucket
- **MongoDB** instance (local ou Atlas)

### Instalação Rápida

1. **Clone o repositório**
```bash
git clone https://github.com/seu-usuario/OlinxRA.git
cd OlinxRA
```

2. **Configure o Backend**
```bash
cd olinxra-backend
pip install -r requirements.txt
cp .env.example .env
# Edite .env com suas credenciais
python main.py
```

3. **Configure o Admin UI**
```bash
cd olinxra-adminui
npm install
npm run dev
```

4. **Configure o Mobile App**
```bash
cd olinxra-app
npm install
npm start
```

### Configuração de Credenciais

Consulte a documentação específica de cada módulo para configuração detalhada:

- [Backend Setup](olinxra-backend/README.md)
- [Admin UI Setup](olinxra-adminui/README.md)
- [Mobile App Setup](olinxra-app/README.md)

## 📚 Documentação

### Documentos Técnicos

- [Arquitetura de Storage](docs/ARQUITETURA-STORAGE.md)
- [Camadas de Acesso](docs/CAMADAS-DE-ACESSO.md)
- [Múltiplos Modelos AR](docs/MULTIPLOS-MODELOS-AR.md)
- [Sincronização e Deleção GLB](docs/SINCRONIZACAO-DELECAO-GLB.md)
- [Calibração de Threshold](docs/THRESHOLD_CALIBRATION.md)

### Guias de Desenvolvimento

- [Upload GLB Frontend](olinxra-adminui/UPLOAD-GLB-FRONTEND.md)
- [Esquema AR](olinxra-adminui/AR_SCHEMA.md)
- [Endpoints API](olinxra-adminui/ENDPOINTS.md)
- [Teste de Fluxo AR](olinxra-app/TESTE-FLUXO-AR.md)
- [Histórico AR Android](olinxra-app/HISTORICO-AR-ANDROID.md)

## 🔧 Desenvolvimento

### Estrutura de Branches

- `master` - Produção estável
- `develop` - Desenvolvimento ativo
- `feature/*` - Novas funcionalidades
- `fix/*` - Correções de bugs

### Scripts Úteis

```bash
# Desenvolvimento completo (Admin UI + Backend)
cd olinxra-adminui
npm run dev:full

# Apenas Frontend
npm run dev

# Apenas Backend
npm run backend

# Build para produção
npm run build
```

## 🤝 Contribuindo

Contribuições são bem-vindas! Por favor, siga estes passos:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'Adiciona MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 👥 Time

Desenvolvido por Olinx Digital 2025

## 📞 Suporte

Para suporte e dúvidas:
- 📧 Email: suporte@olinxra.com
- 💬 Issues: [GitHub Issues](https://github.com/seu-usuario/OlinxRA/issues)

---

<div align="center">
Feito com ❤️ e tecnologias de ponta
</div>
