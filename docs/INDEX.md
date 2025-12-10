# 📚 Índice de Documentação Técnica - Olinx Plus

**Última atualização:** 10 de dezembro de 2025  
**Projeto:** Olinx Plus - Plataforma de Realidade Aumentada com Reconhecimento Visual

---

## 🏗️ Arquitetura e Infraestrutura

### [ARQUITETURA-STORAGE.md](./ARQUITETURA-STORAGE.md)
**Tema:** Estrutura de armazenamento no Google Cloud Storage  
**Conteúdo:**
- Organização de pastas por usuário (`{userId}/`)
- Estrutura de GLBs (`{userId}/ra/models/`)
- Isolamento de conteúdo entre admins
- Regras de derivação de paths
- Metadados e signed URLs

**Tags:** `GCS`, `Storage`, `Arquitetura`, `Buckets`

---

### [CAMADAS-DE-ACESSO.md](./CAMADAS-DE-ACESSO.md)
**Tema:** Sistema de autenticação e controle de acesso  
**Conteúdo:**
- Camada AdminUI (autenticada com Firebase)
- Camada App Mobile (pública, sem autenticação)
- Fluxos de autenticação e autorização
- Isolamento de recursos por `owner_uid`
- Endpoints protegidos vs públicos

**Tags:** `Firebase Auth`, `Segurança`, `ACL`, `Autenticação`

---

## 🎯 Reconhecimento Visual e IA

### [THRESHOLD_CALIBRATION.md](./THRESHOLD_CALIBRATION.md)
**Tema:** Calibração de thresholds de reconhecimento CLIP + FAISS  
**Conteúdo:**
- Análise matemática de conversão distância → confiança
- Thresholds atuais: Combined (0.50), Min Margin (0.01), Acceptance (0.72)
- Comparação antes/depois
- Riscos de falsos positivos
- Casos de teste e validação

**Tags:** `CLIP`, `FAISS`, `Thresholds`, `Machine Learning`, `Reconhecimento`

**Atualização mais recente:** Dezembro 2025 - Thresholds otimizados para crop app-side

---

### [CROP-OPTIMIZATION-MOBILE.md](./CROP-OPTIMIZATION-MOBILE.md)
**Tema:** Otimização de crop no app mobile com marcadores visuais  
**Conteúdo:**
- Marcadores de enquadramento 300x250px
- Crop preciso usando expo-image-manipulator
- Desabilitação de crop adaptativo no backend
- Comparação de performance e latência
- Validação em produção (Digital Ocean logs)

**Tags:** `Mobile`, `Crop`, `UX`, `Performance`, `Latência`

**Atualização mais recente:** Dezembro 2025 - Implementação completa

---

## 🥽 Realidade Aumentada

### [MULTIPLOS-MODELOS-AR.md](./MULTIPLOS-MODELOS-AR.md)
**Tema:** Visualização e navegação entre múltiplos modelos 3D em AR  
**Conteúdo:**
- Extração de GLBs dos blocos de conteúdo
- Componente de navegação (ARNavigationControls)
- Estados e gerenciamento de índice
- Suporte a carrosséis de modelos 3D
- Integração com Expo GL + React Three Fiber

**Tags:** `AR`, `GLB`, `3D Models`, `Carousel`, `Navigation`

---

### [PLANO-CARROSSEL-AR.md](./PLANO-CARROSSEL-AR.md)
**Tema:** Plano de implementação do carrossel AR  
**Status:** ✅ Implementado e em produção  
**Conteúdo:**
- Fases de implementação (Backend, Frontend, Mobile)
- ARPayloadContext preparado
- Bug fixes (usuários anônimos)
- AdminUI otimizado (React 19, code-splitting)
- Scripts de geração de GLBs

**Tags:** `Planejamento`, `AR`, `Implementação`, `Status`

---

### [CHANGELOG-CARROSSEL-AR.md](./CHANGELOG-CARROSSEL-AR.md)
**Tema:** Histórico de mudanças do carrossel AR  
**Conteúdo:**
- Modificações no endpoint `/api/add-content-image`
- Geração automática de GLBs
- Signed URLs com expiração customizada
- Response atualizado com campos `glb_url` e `glb_signed_url`

**Tags:** `Changelog`, `GLB`, `Backend`, `API`

---

### [TESTE-CARROSSEL-GLB.md](./TESTE-CARROSSEL-GLB.md)
**Tema:** Guia de testes para pré-geração de GLB  
**Conteúdo:**
- Testes com Postman/Insomnia/curl
- Verificação de arquivos no GCS
- Validação de signed URLs
- Testes de navegação no app
- Troubleshooting comum

**Tags:** `Testing`, `QA`, `GLB`, `Validação`

---

### [SINCRONIZACAO-DELECAO-GLB.md](./SINCRONIZACAO-DELECAO-GLB.md)
**Tema:** Sincronização de deleção entre imagens e GLBs  
**Conteúdo:**
- Derivação de path do GLB a partir da imagem
- Função `delete_image_and_glb()`
- Limpeza automática de arquivos órfãos
- Integração com endpoints de deleção
- Exemplos de transformação de paths

**Tags:** `Storage`, `Cleanup`, `Sincronização`, `GCS`

---

### [ACESSO-GLB-APP.md](./ACESSO-GLB-APP.md)
**Tema:** Como o app mobile acessa GLBs privados no GCS  
**Conteúdo:**
- Fluxo completo: Upload → App
- Geração de Signed URLs (7-365 dias)
- Como funcionam signed URLs
- Carregamento de GLBs em AR
- Segurança e expiração

**Tags:** `GCS`, `Signed URLs`, `Mobile`, `AR`, `Segurança`

---

### [GERACAO-AUTOMATICA-GLB.md](./GERACAO-AUTOMATICA-GLB.md)
**Tema:** Sistema de geração automática de modelos 3D  
**Conteúdo:**
- Geração automática no upload de imagens
- Função `generate_plane_glb()`
- Mesclagem de campos GLB no MongoDB
- Retorno de conteúdo com GLBs pré-gerados
- Regeneração sob demanda

**Tags:** `GLB`, `3D`, `Automação`, `Pipeline`

---

## 🎨 UI/UX e Frontend

### [HIERARQUIA-VISUAL-TEXTO.md](./HIERARQUIA-VISUAL-TEXTO.md)
**Tema:** Padronização visual de blocos de texto  
**Conteúdo:**
- 4 níveis hierárquicos (Título, Subtítulo, Título de Seção, Texto)
- Especificações de fontSize, fontWeight, lineHeight
- Espaçamentos e cores padronizadas
- Exemplos de uso
- Guidelines de consistência

**Tags:** `Design System`, `Typography`, `UI`, `Padronização`

---

### [LOADING_IMPROVEMENTS.md](./LOADING_IMPROVEMENTS.md)
**Tema:** Melhorias de performance e UX no carregamento  
**Conteúdo:**
- Sistema de cache local (AsyncStorage, 30min TTL)
- Loading inteligente com dicas rotativas
- Feedback de estágio de carregamento
- Comparação de performance (3-5s → 50-200ms com cache)
- Componente LoadingWithTips

**Tags:** `Performance`, `Cache`, `UX`, `Loading`, `AsyncStorage`

**Nota:** Backend hospedado no Digital Ocean App Platform (NYC region)

---

## 📋 Especificações Técnicas

### [HEADER_PREVIEW_SPEC.md](./HEADER_PREVIEW_SPEC.md)
**Tema:** Especificação de preview de headers  
**Conteúdo:** *(Documento não foi lido completamente, adicionar conteúdo se necessário)*

**Tags:** `Especificação`, `Headers`, `Preview`

---

## 📊 Status e Referências

### Documentos Principais Atualizados (Dezembro 2025):
1. ✅ THRESHOLD_CALIBRATION.md - Thresholds otimizados
2. ✅ CROP-OPTIMIZATION-MOBILE.md - Novo documento sobre crop app-side
3. ✅ ARQUITETURA-STORAGE.md - Nome do projeto corrigido
4. ✅ CAMADAS-DE-ACESSO.md - Nome do projeto corrigido
5. ✅ LOADING_IMPROVEMENTS.md - Info sobre Digital Ocean
6. ✅ PLANO-CARROSSEL-AR.md - Status atualizado, repositórios corretos
7. ✅ CHANGELOG-CARROSSEL-AR.md - Repositórios e projeto corretos
8. ✅ TESTE-CARROSSEL-GLB.md - URLs e comandos atualizados
9. ✅ ACESSO-GLB-APP.md - Repositórios e stack tech atualizados
10. ✅ GERACAO-AUTOMATICA-GLB.md - Processo de crop atualizado

---

## 🔗 Links Úteis

### Repositórios GitHub:
- **Mobile App:** [github.com/gibadalcin/olinxplus](https://github.com/gibadalcin/olinxplus)
- **Backend API:** [github.com/gibadalcin/olinxplus-backend](https://github.com/gibadalcin/olinxplus-backend)
- **Admin UI:** [github.com/gibadalcin/olinxplus-adminui](https://github.com/gibadalcin/olinxplus-adminui)
- **Documentação:** [github.com/gibadalcin/olinxplus-docs](https://github.com/gibadalcin/olinxplus-docs)

### Documentação Externa:
- [README Principal](../README.md) - Visão geral do projeto
- [Backend CROP-OPTIMIZATION.md](https://github.com/gibadalcin/olinxplus-backend/blob/master/docs/CROP-OPTIMIZATION.md) - Otimizações backend

---

## 📞 Suporte

**Desenvolvido por:** Gibanet Tecnologia  
**Email:** contato@gibanet.com.br  
**Website:** [gibanet.com.br](https://gibanet.com.br)

---

**Olinx Plus** - Conectando o mundo físico ao digital através de AR 🎯
