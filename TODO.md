# TODO — Suzanny Nail Designer

## 1. Vitrine pública e fluxo de agendamento
- [ ] Implementar UI mobile-first premium com Azul Imperial, Dourado Nobre, Grafite e Bronze, feedback tátil, sombras `shadow-md` e spinners.
- [ ] Permitir escolher um Serviço Principal e selecionar múltiplos Serviços Adicionais em card separado; cada adicional soma ao preço final e não altera a duração.
- [ ] Limitar o calendário entre Hoje e 60 dias no futuro; para bloqueio de Dia Inteiro, impedir a escolha e exibir: “Neste dia estaremos fechados por motivos pessoais. Por favor, escolha outra data.”
- [ ] Exibir horários em blocos de 15 minutos, ocultar horários passados no dia atual, filtrar pelo corredor livre da duração e ocultar inícios após 18:00 para clientes.
- [ ] Destacar em laranja horários cujo atendimento termine após 19:00; ao clicar, abrir modal de exceção com botão WhatsApp e data exata para aprovação.
- [ ] Exibir links de Google Maps, Instagram/portfólio e WhatsApp para imprevistos; os contatos devem ser configuráveis e não fictícios.
- [ ] Limitar a dois agendamentos por dia no mesmo aparelho via `localStorage`; ao exceder, orientar contato pelo WhatsApp.

## 2. Disponibilidade e sincronização Firebase
- [ ] Separar dados privados em `appointments` e dados de ocupação em `public_slots`, sem revelar nome, telefone ou preço em leituras públicas.
- [ ] Sincronizar disponibilidade pública e painel por listeners Firebase equivalentes a `onSnapshot`; uma vaga reservada deve desaparecer para outras clientes sem recarregar.
- [ ] Calcular disponibilidade cruzando grade 07:00–21:00, `public_slots` e `blocks`, bloqueando qualquer interseção.
- [ ] Aplicar reserva e reagendamento em operações atômicas/batch com prevenção de colisões; falhas não podem criar agendamentos fantasmas.

## 3. Painel privado e autorização
- [ ] Proteger o painel com Firebase Auth e autorização hierárquica; Recepção vê apenas Agenda, podendo alterar status e reagendar; Admin acessa todos os módulos.
- [ ] Na Agenda, mostrar faturamento diário somente de Agendado/Concluído, mapa térmico Verde/Livre, Vermelho/Ocupado e Cinza/Folga, e encaixe rápido em horários livres, sem antispam e permitindo após 18h.
- [ ] Nos cartões, oferecer WhatsApp, alerta de fraude quando valor pago for menor que a tabela, status Agendado/Concluído/Cancelado/Faltou, liberação imediata ao cancelar, reagendamento atômico e exclusão definitiva.
- [ ] Remoção da equipe deve deslogar/revogar o acesso em tempo real e bloquear novas operações no servidor.

## 4. Finanças e CRM
- [ ] Exibir faturamento real e quantidade de atendimentos dos últimos 30 dias.
- [ ] Identificar clientes cujo prazo de manutenção venceu, sem agendamento futuro, e gerar WhatsApp: “Oi [Nome], vi que passou X dias da sua [Serviço]. Vamos marcar?”.

## 5. Serviços, bloqueios e equipe
- [ ] Cadastrar/editar Serviços Principais com duração/preço e Serviços Adicionais somente com preço; permitir Dias de Manutenção.
- [ ] Implementar soft delete de serviços sem apagar histórico financeiro.
- [ ] Criar bloqueios de 15 minutos, período de hora inicial/final e Dia Inteiro.
- [ ] Permitir convite de funcionárias por e-mail, papel Recepção/Admin e criação da própria senha via Firebase Auth.

## 6. Segurança, LGPD e entrega
- [ ] Validar schema estrito com `hasOnly`/campos permitidos, nomes e telefones de até 100 caracteres, datas não anteriores a 2026 e preço mínimo do serviço principal mais adicionais.
- [ ] Escapar strings exibidas no painel contra XSS e proteger ações de edição com nomes contendo aspas/apóstrofos.
- [ ] Criar regras e índices do Firestore, configurar Firebase Auth/Firestore e manter credenciais fora do repositório.
- [ ] Configurar manifesto `/manus-routes.json`, health `/api/health`, logo da marca e Dockerfile de produção; executar diagnósticos TypeScript, build e revisão de integração antes do checkpoint.
