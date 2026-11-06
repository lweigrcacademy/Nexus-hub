## DOCUMENTO MOSCOW - NEXUSHUB / LWEI
MATRIZ DE PRIORIZAÇÃO MOSCOW - NEXUSHUB ENTERPRISE HUB
Projeto: Sistema LWEI - Base para Mini-SaaS
Stack: Next.js + NestJS + PostgreSQL + Redis
Data: Outubro 2026
Entrega MVP: Dezembro 2026

OBJETIVO DA PRIORIZAÇÃO:
Garantir que o MVP da Lwei entregue valor B2B real sem scope creep, preparando a base multi-tenant para vender no Brasil e Europa.

MUST HAVE - Tem que ter no MVP (Sem isso não lança)

Essencial para a Lwei parar de usar WhatsApp + Excel.

1.  [RF01] Site público 100% dinâmico (Home, Sobre missão/visão/valores, Serviços, Projetos, Clientes, Contacto) editável sem deploy
    - Porquê MUST: Resolve problema #1 do BRD. Critério de Sucesso CS01.

2.  [RN01] Multi-tenancy base: campo empresaId obrigatório em TODAS as tabelas (empresa, serviços, projetos, membros, clientes)
    - Porquê MUST: Sem isso, nunca vira SaaS. Critério CS05.

3.  [RF05 + RF06] Módulo de Projetos Core: CRUD, status (orçamento, em execução, concluído), vincular cliente, alocar equipa
    - Porquê MUST: Core do negócio.

4.  [RF04] Módulo Financeiro: Orçamento, Custo, Lucro = Orçamento - Custo calculado automaticamente (RN04)
    - Porquê MUST: Saber lucro real por projeto.

5.  [RF08 + RN03] Área do Cliente com login JWT + Roles (ADMIN, STAFF, CLIENT) - Cliente só vê seus projetos
    - Porquê MUST: Reduzir 80% das mensagens de WhatsApp. Critério CS02 e CS04. Segurança.

6.  [RNF02] Autenticação segura: bcrypt, JWT 8h, rate limit login 5 tent/min no Redis
    - Porquê MUST: B2B exige segurança.

7.  [RNF01 + RN02] Cache Redis no site público: public:site:{empresaId} com TTL 10min + invalidação no Admin
    - Porquê MUST: Performance < 1.5s e Lighthouse > 90 (CS03).

8.  [RF03] Upload de logos/fotos com limite 5MB e compressão (RN05)

SHOULD HAVE - Deveria ter no MVP (Importante, mas dá para lançar sem se apertar o prazo)


9.  Dashboard Admin com KPIs: Total projetos, em execução, lucro total do mês
    - Impacto: Direção vê saúde da empresa rápido.

10. Relatórios PDF por projeto para download na Área do Cliente
    - Impacto: Profissionalismo B2B.

11. Filtros avançados no Admin: Filtrar projetos por cliente, status, membro alocado

12. Histórico de alterações (audit log): Quem alterou missão, quem mudou status do projeto

13. SEO dinâmico: Título e descrição meta editáveis por página via Admin

COULD HAVE - Poderia ter (Bom ter se sobrar tempo, não atrasa MVP)


14. Galeria de imagens por projeto (até 5 fotos)

15. Testemunhos de clientes com nota e aprovação no Admin antes de ir pro site

16. Exportar financeiro para Excel (lucro por projeto)

17. Modo escuro/claro no site público (mesma logo branca e azul que já criámos)

18. Busca global no Admin (procurar projeto por nome do cliente)

WON'T HAVE (THIS TIME) - Não terá nesta fase (Fase 2 - SaaS)

Congelado de propósito para evitar scope creep - Risco 1 do BRD.

19. Pagamento recorrente Stripe / Gerencianet / Multicaixa Express
20. Criação self-service de nova empresa (cadastro e checkout)
21. Subdomínio automático (cliente1.nexushub.ao)
22. Chat em tempo real entre equipa e cliente
23. Notificações por Email / WhatsApp / SMS
24. App mobile
25. Kanban drag-and-drop tipo Trello
26. Integração com contabilidade externa
