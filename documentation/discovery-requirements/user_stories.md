## USER STORIES & ACCEPTANCE CRITERIA - NEXUSHUB / SISTEMA LWEI
Stack: Next.js + NestJS + PostgreSQL + Redis

=== EPICO 1: SITE PÚBLICO DINÂMICO ===

US01 - Editar conteúdo institucional
Como Admin da Lwei
Quero editar missão, visão e valores sem precisar de dev
Para que a direção atualize o site sozinha

Critérios de Aceite:
Dado que estou logado como ADMIN
Quando entro em Admin > Empresa > Institucional e altero a Missão
Então o texto deve aparecer no site público em até 10 minutos (cache Redis)
E o histórico de alteração deve ser salvo com data e usuário
E não deve exigir deploy

US02 - Gestão de Serviços
Como Admin
Quero cadastrar serviços com título, descrição, ícone e status ativo/inativo
Para que o site mostre apenas serviços ativos

Critérios de Aceite:
Dado que cadastro um serviço com status inativo
Então ele NÃO deve aparecer em /servicos no site público
Quando ativo o serviço
Então ele deve aparecer com cache invalidado no Redis (key: public:services)

US03 - Portfólio Público
Como visitante do site
Quero ver apenas projetos concluídos
Para ter prova social da Lwei

Critérios de Aceite:
Dado que existem projetos com status ORÇAMENTO, EM_EXECUCAO e CONCLUIDO
Quando acesso /projetos
Então devo ver APENAS projetos com status = CONCLUIDO
E cada projeto deve mostrar cliente, logo e descrição curta

=== EPICO 2: GESTÃO DE PROJETOS (CORE) ===

US04 - Criar Projeto com multi-tenant
Como Admin
Quero criar um projeto vinculado a uma company_code, cliente e orçamento
Para garantir isolamento de dados para o futuro SaaS

Critérios de Aceite:
Dado que crio um projeto
Então o campo company_code deve ser obrigatório (RN01)
E o sistema deve validar que cliente pertence à mesma company_code
E o orçamento não pode ser negativo

US05 - Alocação de Equipa
Como Admin
Quero alocar membros da equipa em um projeto
Para saber quem está trabalhando onde

Critérios de Aceite:
Dado que tenho um projeto criado
Quando adiciono 2 membros da equipa
Então o sistema deve criar registros em tabela project_members
E impedir alocação de membro de outra company_code

US06 - Atualização de Status por Staff
Como Staff (engenheiro/técnico)
Quero atualizar status do projeto e adicionar custos
Para que o financeiro reflita a realidade

Critérios de Aceite:
Dado que estou logado como STAFF
Quando altero status de ORÇAMENTO para EM_EXECUCAO
Então o sistema deve permitir
Mas NÃO devo poder editar orçamento inicial (apenas Admin)
Quando adiciono custo de 5000 Kz
Então lucro = orçamento - soma(custos) deve ser recalculado automaticamente

=== EPICO 3: FINANCEIRO ===

US07 - Cálculo automático de lucro
Como Admin
Quero ver lucro por projeto calculado automaticamente
Para saber rentabilidade real

Critérios de Aceite:
Dado um projeto com orçamento = 100.000 e custos = 30.000 + 20.000
Então lucro deve ser exibido como 50.000
E nunca pode ser exibido lucro com valor negativo por erro de cálculo
Se custo > orçamento, lucro = negativo e deve mostrar alerta visual vermelho

=== EPICO 4: ÁREA DO CLIENTE ===

US08 - Login do Cliente Final
Como Cliente Final da Lwei
Quero fazer login com meu email e senha
Para acompanhar meus projetos

Critérios de Aceite:
Dado que sou cliente cadastrado no projeto
Quando faço login em /cliente/login
Então recebo JWT com expiração de 8h e role = CLIENT
E só devo ver projetos onde meu email está em project.clientEmail
E rate limit: máximo 5 tentativas por minuto (controlado via Redis key: ratelimit:login:{email})

US09 - Visualização restrita
Como Cliente
Quero ver status, timeline e baixar relatório do meu projeto
Para não precisar perguntar no WhatsApp

Critérios de Aceite:
Dado que estou logado como cliente A (a@empresa.com)
Quando tento acessar projeto do cliente B via URL /api/projects/{id}
Então API deve retornar 403 Forbidden
E deve registrar tentativa em log
Quando acesso meu projeto, devo ver status, membros alocados (apenas nome e especialidade) e botão Download Relatório se houver arquivo

=== EPICO 5: SEGURANÇA E PERFORMANCE ===

US10 - Cache Público com Redis
Como Sistema
Quero servir site público do Redis
Para carregar em < 1.5s

Critérios de Aceite:
Dado que acesso GET /api/public/site-data
Então sistema deve primeiro buscar em Redis key public:site:{company_code}
Se não existir, busca no PSQL e salva no Redis com TTL 600s (10 min)
Toda edição no Admin deve dar DEL na key correspondente

US11 - Isolamento multi-tenant
Como Sistema
Quero garantir que dados nunca vazem entre empresas
Para futuro SaaS seguro

Critérios de Aceite:
Dado que existem company_code = LWEI e company_code = AGROTRAFO
Quando faço query com JWT de LWEI
Então TODAS as queries no NestJS devem ter WHERE company_code = LWEI automaticamente via Guard/Middleware
E teste automatizado deve garantir: usuário LWEI não lista projeto AGROTRAFO
