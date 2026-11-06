### BUSINESS REQUIREMENT DOCUMENT - SISTEMA Nexus Hub

*1. CONTEXTO DE NEGÓCIO*

A LWEI é uma empresa Angolana de serviços em Engenharia, TI e Cybersecurity. Atua com clientes B2B que exigem profissionalismo, rastreabilidade e relatórios técnicos.

Hoje a Lwei usa site estático + WhatsApp + Excel para gerir projetos. Isso impede escala e não transmite confiança para clientes corporativos e internacionais. O objetivo deste projeto é transformar o site da Lwei em uma plataforma de gestão com área do cliente, servindo como produto base para um mini-SaaS a ser vendido no Brasil (R$) e Europa (€).

Stack definida e imutável: *Next.js + NestJS + PostgreSQL + Redis*

*2. PROBLEMA ATUAL*

- Site atual não é editável pela equipa. Depende de dev para trocar missão, serviço, logo de cliente.
- Sem gestão centralizada de projetos: quem está alocado, qual status, qual cliente.
- Sem área do cliente: cliente pergunta no WhatsApp "como está meu pentest?".
- Sem financeiro vinculado ao projeto: não se sabe lucro real por projeto.
- Sem base para multi-empresa: impossível vender o mesmo sistema para outra empresa.
- Sem prova social dinâmica: logos de clientes e cases desatualizados.

*3. OBJETIVO DO SISTEMA*

*Objetivo Geral:* Desenvolver uma plataforma web onde a Lwei gere seu portfólio público, seus projetos, sua equipa e seu financeiro, com acesso restrito para clientes acompanharem seus serviços.

*Objetivos Específicos:*
- Tornar o site 100% dinâmico e editável via Admin.
- Centralizar projetos, clientes de portfólio, equipa e financeiro.
- Criar Área do Cliente com login para acompanhamento de status.
- Preparar arquitetura multi-tenant (empresaId) para futuro SaaS.
- Garantir performance com Redis cache no site público.

*4. STAKEHOLDERS*

- *Sponsor:* Direção LWEI / Prof. David
- *Product Owner:* Tu - Responsável técnico e de negócio
- *Usuários Admin:* Direção Lwei - Edita conteúdo institucional, vê financeiro global
- *Usuários Staff:* Engenheiros / Técnicos - Atualiza status de projetos, sobe evidências
- *Usuários Client:* Clientes finais da Lwei - Apenas visualizam seus projetos e relatórios
- *Futuros Stakeholders SaaS:* Outras empresas (AgroTrafo, etc.) que usarão a mesma plataforma

*5. ESCOPO*

*Incluído no MVP (Entrega Lwei):*
- Site público dinâmico: Home, Sobre (missão, visão, valores), Serviços, Projetos/Portfólio, Clientes, Contacto
- Módulo Admin: CRUD de Empresa, Serviços, Clientes de Portfólio, Membros da Equipa, Projetos
- Módulo Projetos: status (orçamento, em execução, concluído), alocação de equipa, cliente vinculado
- Módulo Financeiro: Orçamento, Custo, Lucro = Orçamento - Custo
- Área do Cliente: Login, listagem dos seus projetos, status, download de relatório
- Autenticação: JWT + Roles + Redis para sessão/rate limit

*Fora do Escopo MVP (Fase 2 - SaaS):*
- Pagamento recorrente (Stripe), Criação self-service de nova empresa, Subdomínio automático, Chat em tempo real, Notificações por email/WhatsApp, App mobile.

*6. REQUISITOS DE NEGÓCIO*

- RN01: Todo dado deve pertencer a uma Empresa (empresaId) para permitir multi-tenant no futuro.
- RN02: Site público deve carregar em < 1.5s, usando cache Redis de 10 min para conteúdo institucional.
- RN03: Cliente final só vê projetos onde seu email está vinculado como cliente.
- RN04: Financeiro não pode ter valor negativo e lucro deve ser calculado automaticamente.
- RN05: Logos e fotos de projetos devem ser armazenados com compressão e limite de 5MB.

*7. REQUISITOS FUNCIONAIS*

- RF01: Admin pode editar missão, visão, valores sem deploy.
- RF02: Admin pode cadastrar/editar/desativar serviços.
- RF03: Admin pode cadastrar clientes de portfólio (nome, logo, testemunho).
- RF04: Admin pode cadastrar membros da equipa e definir especialidade.
- RF05: Admin pode criar projeto, vincular cliente de portfólio, alocar membros e definir orçamento.
- RF06: Staff pode atualizar status do projeto e adicionar custos.
- RF07: Sistema deve exibir lista pública de projetos filtrada por status = concluído.
- RF08: Cliente pode fazer login e visualizar apenas seus projetos.

*8. REQUISITOS NÃO FUNCIONAIS*

- RNF01: Stack: Next.js (FE), NestJS (API), PSQL, Redis.
- RNF02: Segurança: Senhas com bcrypt, JWT expiração 8h, rate limit login 5 tentativas/min no Redis.
- RNF03: Performance: 95% das requisições públicas servidas do cache Redis.
- RNF04: Escalabilidade: Arquitetura monolito modular, preparada para extração futura de módulos.
- RNF05: Disponibilidade: Uptime alvo 99.5% no MVP.

*9. RESTRIÇÕES*

- Orçamento e tempo limitados - Entrega até Dezembro 2026.
- Equipa de 1 desenvolvedor.
- Sem orçamento para serviços pagos caros no MVP - Priorizar Supabase (PSQL) free tier e Upstash Redis free tier.
- Conteúdo inicial deve ser migrado manualmente do site antigo.

*10. CRITÉRIOS DE SUCESSO*

- CS01: Lwei consegue editar sozinha 100% do conteúdo da Home em < 2 min sem dev.
- CS02: Redução de 80% de mensagens "como está meu projeto?" no WhatsApp após liberação da área do cliente.
- CS03: Site público com Lighthouse > 90 em performance.
- CS04: 0 vazamento de dados entre clientes (cliente A não vê projeto do cliente B).
- CS05: Base pronta para cadastrar 2ª empresa (AgroTrafo) sem refazer BD, apenas adicionando nova empresaId.

*11. RISCOS*

- Risco 1: Scope creep - querer colocar chat, notificação agora. Mitigação: Congelar MVP.
- Risco 2: Upload de arquivos pesados derruba servidor. Mitigação: Limite de tamanho + fila futura com Redis.
- Risco 3: Cliente não adota área do cliente. Mitigação: Treinamento de 30 min na entrega.