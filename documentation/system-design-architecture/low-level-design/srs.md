SRS - SOFTWARE REQUIREMENT SPECIFICATION
Projeto: NexusHub Enterprise Hub / Sistema LWEI
Stack Final: Next.js + NestJS + PostgreSQL + Redis + BullMQ + TypeORM
Data: 26-10-2026

1. VISÃO GERAL TÉCNICA

Sistema multi-tenant com Shared Database Shared Schema. Toda tabela tem empresaId.
Frontend Next.js serve site público com ISR + Redis cache.
Backend NestJS em monolito modular com TenantGuard global.
Banco PostgreSQL via Supabase. Redis via Upstash para cache e rate limit. BullMQ para fila de compressão de imagens.

2. MODELO DE DADOS - ENTITIES TYPEORM

Todas herdam de BaseTenantEntity

BaseTenantEntity:
- id: uuid, PK, default uuid_generate_v4()
- empresaId: uuid, FK -> empresas.id, INDEX, NOT NULL
- createdAt: timestamptz
- updatedAt: timestamptz

Tabela empresas:
- id: uuid PK
- nome: varchar (LWEI)
- slug: varchar unique (lwei)
- moedaDefault: varchar (AOA) - AOA, BRL, EUR
- localeDefault: varchar (pt-AO)
- missao: text nullable
- visao: text nullable
- valores: text nullable
- siteDataJson: jsonb nullable (dados da home editáveis)

Tabela users:
- id: uuid PK
- empresaId: FK
- email: varchar, unique por empresaId (INDEX composto empresaId+email)
- senhaHash: varchar (bcrypt)
- role: enum (ADMIN, STAFF, CLIENT)
- nome: varchar

Tabela servicos:
- empresaId FK
- titulo: varchar
- descricao: text
- icone: varchar nullable
- ativo: boolean default true

Tabela clientes_portfolio (logos do site):
- empresaId FK
- nome: varchar
- logoUrl: varchar
- testemunho: text nullable
- ativo: boolean

Tabela membros_equipa:
- empresaId FK
- nome: varchar
- especialidade: varchar (Ex: Cybersecurity)
- fotoUrl: varchar nullable
- ativo: boolean

Tabela projetos:
- empresaId FK
- titulo: varchar
- descricao: text
- status: enum (ORCAMENTO, EM_EXECUCAO, CONCLUIDO)
- clientePortfolioId: FK nullable -> clientes_portfolio
- clientEmail: varchar nullable (email do cliente que vai logar na área do cliente)
- orcamentoCentavos: int (armazenar em centavos, ex: 10000000 = 100.000,00)
- moeda: varchar (AOA)
- relatorioUrl: varchar nullable (PDF)

Tabela project_members (N:N):
- empresaId FK
- projetoId: FK
- membroId: FK

Tabela custos:
- empresaId FK
- projetoId: FK
- descricao: varchar (Ex: Licença firewall)
- valorCentavos: int
- criadoPorUserId: FK

Regra financeira: lucroCentavos = orcamentoCentavos - SUM(custos.valorCentavos) -> calculado em View ou no Service, nunca salvo para evitar inconsistência.

3. ENDPOINTS DA API NESTJS

Prefixo global: /api
Todos os endpoints exceto /public/* exigem JWT. TenantGuard extrai empresaId do JWT e injeta automaticamente.

3.1 PUBLICO - Sem Auth, com cache Redis
GET /public/site/:slug -> retorna siteDataJson + serviços ativos + projetos concluídos + clientes + equipa. Lógica: busca Redis key public:site:{empresaId}, se miss busca PSQL e seta TTL 600s.
GET /public/servicos/:empresaId -> lista serviços ativos
GET /public/projetos/:empresaId -> lista projetos status CONCLUIDO

3.2 ADMIN - Role ADMIN
POST /admin/auth/login -> retorna JWT {userId, empresaId, role}
CRUD /admin/empresa/:id -> editar missão, visão, valores. Ao editar faz DEL public:site:{empresaId}
CRUD /admin/servicos -> create, update, soft delete
CRUD /admin/clientes-portfolio
CRUD /admin/membros-equipa
CRUD /admin/projetos -> criar com orcamentoCentavos, vincular clientEmail, alocar membros via body {membroIds: []}
POST /admin/projetos/:id/custos -> adicionar custo
GET /admin/financeiro/dashboard -> retorna {totalProjetos, totalOrcamentoCentavos, totalCustoCentavos, totalLucroCentavos} filtrado por empresaId
POST /admin/upload -> upload de logo/foto. Valida 5MB. Salva em temp/{empresaId}/ e cria job BullMQ image:compress

3.3 STAFF - Role ADMIN, STAFF
PATCH /staff/projetos/:id/status -> atualizar status ORCAMENTO -> EM_EXECUCAO -> CONCLUIDO
POST /staff/projetos/:id/custos

3.4 CLIENT - Role CLIENT
POST /client/auth/login -> login com email que está em projetos.clientEmail
GET /client/meus-projetos -> retorna apenas projetos onde projetos.clientEmail = JWT.email AND empresaId = JWT.empresaId
GET /client/projetos/:id -> detalhe, se tentar ver projeto de outro email retorna 403

4. REDIS KEYS E BULLMQ

Cache:
public:site:{empresaId} -> JSON completo do site, TTL 600s
public:services:{empresaId} -> TTL 600s
public:projects:{empresaId} -> TTL 600s
ratelimit:login:{email} -> int, TTL 60s, INCR a cada tentativa. Se >5 bloqueia.

Blacklist JWT (logout):
blacklist:jwt:{jti} -> 1, TTL = tempo restante do token

Filas BullMQ:
Nome: image:compress
Payload: {empresaId, tempFilePath, destino: logos|projetos|relatorios, projetoId?}
Processador: baixa temp, sharp resize max 1200px webp 80%, upload para public/{empresaId}/..., atualiza logoUrl/fotoUrl no PSQL, deleta temp.

Nome futuro: report:generate, email:send (Fase 2)

5. SEGURANÇA

- Bcrypt salt 10 para senhas
- JWT expiração 8h, secret em env JWT_SECRET
- TenantGuard global: se request não tem empresaId no JWT, bloqueia. Se tem, adiciona empresaId em todas queries TypeORM
- Rate limit login via Redis
- File validation: max 5MB, mime image/jpeg, image/png, image/webp, application/pdf
- CORS: apenas domínios da Lwei e localhost

6. REGRAS DE CÁLCULO E FORMATAÇÃO

- Sempre salvar dinheiro em centavos int
- Cálculo lucro: lucroCentavos = orcamentoCentavos - sum(custos)
- Se custo > orçamento, lucro negativo, frontend mostra vermelho
- Formatação no Next.js: new Intl.NumberFormat(localeDefault, {style: 'currency', currency: moedaDefault}).format(centavos/100)

7. DEPLOY

- Next.js -> Vercel, env NEXT_PUBLIC_API_URL
- NestJS + Worker BullMQ -> Render/Railway, mesmo serviço com dois processos: npm run start:prod e npm run worker:prod
- Supabase PSQL + Storage
- Upstash Redis

8. CRITÉRIOS DE ACEITE TÉCNICO DO SRS

- [ ] Nenhum repository.find() sem where empresaId (validado por ESLint custom)
- [ ] Cliente A não consegue GET projeto do cliente B (teste 403)
- [ ] Ao editar missão no Admin, site público atualiza em até 10 min ou imediatamente após DEL Redis
- [ ] Upload de 6MB retorna 413
- [ ] GET /public/site/:slug < 200ms quando vem do Redis
- [ ] Lucro calculado corretamente com centavos