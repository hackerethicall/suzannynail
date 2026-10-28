# Plano de implementação — Suzanny Nail Designer

## Estado e decisão pendente

O projeto web foi inicializado em `/home/ubuntu/suzannynail` com React, TypeScript, Tailwind, Express, tRPC e Drizzle. Servidor e banco MySQL gerenciados estão provisionados, mas ainda não há implementação dos módulos do salão, publicação ou dados de clientes. Os diagnósticos TypeScript estão registrados. Publicação automática está desligada.

O documento original exige Firebase Auth, Firestore, `onSnapshot`, regras de segurança e gravações atômicas. Essa preferência será respeitada, a menos que a proprietária autorize expressamente a alternativa abaixo. O fato de um banco gerenciado existir não autoriza substituir Firebase silenciosamente.

- **Opção A — Firebase, conforme a especificação:** React usa Firebase Auth e `onSnapshot` para dados públicos de disponibilidade. Express verifica tokens e executa operações sensíveis/atômicas via Firebase Admin; Firestore persiste serviços, appointments, public_slots, blocks e team. Exige configuração do projeto Firebase, conta de serviço fornecida em cartão protegido, provedores de login habilitados e aplicação das regras/índices no projeto Firebase. O código não inventará configuração nem afirmará conexão antes de uma conexão real.
- **Opção B — alternativa somente se autorizada:** React/Express/tRPC com MySQL gerenciado. SSE transmite atualizações de ocupação e bloqueios; leituras públicas nunca expõem cliente ou preço. Transações SQL, bloqueio de concorrência e unicidade por data/bloco protegem reservas. A autenticação e o ingresso da equipe serão adaptados com aprovação explícita; a alternativa não será descrita como Firebase nem como `onSnapshot`.

A aprovação deve escolher a infraestrutura antes da implementação. Não haverá autenticação simulada, dados pessoais de demonstração apresentados como reais nem acesso administrativo público.

## Produto e organização das telas

O app terá uma vitrine pública com agendamento e uma área privada de gestão. A vitrine será a página inicial; o painel terá navegação própria por Agenda, Finanças e CRM, Serviços, Bloqueios e Equipe. Recepção acessa somente Agenda, alterações de status e reagendamento. Admin tem todos os módulos. A autorização será validada no servidor, não apenas na interface.

### Vitrine e agendamento

Escolher serviço principal e adicionais em card separado com múltiplos checkboxes. Somar preços dinamicamente; adicionais não alteram a duração. Selecionar datas de hoje até 60 dias à frente. Adotar o fuso do salão, inicialmente America/Sao_Paulo, e usá-lo consistentemente no servidor e na interface.

A grade base é 07:00–21:00, dividida em blocos de 15 minutos. Mostrar apenas início com todos os blocos necessários livres, cruzando public_slots e blocks. Ocultar horários passados no dia atual e inícios após 18:00 para clientes. Se o atendimento terminar após 19:00, o horário será laranja e abrirá pedido de exceção por WhatsApp com a data exata, sem reservar indevidamente.

Dia inteiro bloqueado impede a reserva e mostra: “Neste dia estaremos fechados por motivos pessoais. Por favor, escolha outra data.”

Coletar nome e telefone com limites de 100 caracteres e consentimento/informações claras de tratamento de dados. Limitar a 2 agendamentos por dia por aparelho usando localStorage, com direcionamento ao WhatsApp ao exceder. Esse controle local é uma barreira de conveniência, não uma garantia antifraude; exclusão do localStorage não poderá comprometer validações financeiras ou de colisão do servidor.

Disponibilidade e bloqueios serão sincronizados entre clientes e painel sem recarregar a página. Uma colisão simultânea será recusada atomicamente com mensagem para escolher outro horário.

Rodapé terá Google Maps, Instagram/portfólio e WhatsApp para imprevistos. Número, endereço e perfil reais serão configuráveis; links desconhecidos não usarão contatos fictícios.

### Agenda privada

Resumo de faturamento diário considerando somente Agendado/Concluído. Mapa de ocupação: livre verde, ocupado vermelho e folga cinza. Clicar num bloco livre abre encaixe rápido, incluindo principal e adicionais, sem limite de aparelho e permitindo início após 18h.

Cartões mostram cliente, serviço, adicionais, período, valor e WhatsApp. Um valor registrado inferior à tabela gera alerta vermelho. Status: Agendado, Concluído, Cancelado e Faltou. Cancelamento libera vagas imediatamente. Reagendamento altera serviço, data, hora e adicionais em uma operação atômica, preservando reserva original quando falhar. Exclusão definitiva será oferecida com confirmação explícita no app.

### Finanças e CRM

Faturamento real e número de atendimentos dos últimos 30 dias. Separar valor tabelado, valor efetivamente registrado e situação do atendimento para não apresentar agendamentos futuros como receita realizada. Contabilizar atendimentos concluídos no relatório realizado.

Retornos vencidos cruzam atendimentos recentes com Dias de Manutenção do serviço e ausência de agendamento futuro. Gerar WhatsApp: “Oi [Nome], vi que passou X dias da sua [Serviço]. Vamos marcar?”.

### Serviços, bloqueios e equipe

Serviços principais têm duração e preço; adicionais somam preço sem duração. Cadastro/edição inclui dias de manutenção. Exclusão é soft delete, mantendo o histórico e snapshots de preço/duração nos atendimentos.

Bloqueios permitem 15 minutos, período entre duas horas e dia inteiro. Reservas e bloqueios usam a mesma prevenção de colisões; o app não sobrescreverá silenciosamente um atendimento existente.

Admin cadastra e-mails e papéis Recepção/Admin. A própria funcionária define sua senha na opção Firebase, usando o fluxo autorizado de ingresso; os links de entrada poderão ser compartilhados pela chefe sem prometer envio de e-mail por serviço não configurado. A remoção de acesso será refletida em tempo real, encerrando acesso à área restrita. A verificação no servidor também revoga imediatamente novos pedidos.

## Segurança e persistência

Separar appointments (nome, telefone, preço e dados privados) de public_slots (ocupação sem identidade). Nenhuma rota pública listará dados pessoais de clientes. Bloqueios públicos expõem apenas ocupação e indicação de dia fechado, não motivos particulares.

Validar esquemas estritos para rejeitar chaves extras; limitar strings; aceitar somente datas válidas, no intervalo autorizado e nunca em ano anterior a 2026; validar duração e alinhamento de 15 minutos; validar serviços ativos, adicionais e preços no servidor. O valor da reserva será calculado com a tabela atual, reforçando a regra de mínimo do serviço principal e incluindo adicionais sem confiar no preço do navegador.

No Firebase, regras usam campos permitidos/hasOnly e acesso hierárquico, mantendo escritas sensíveis sob operações verificadas; fornecer regras e índices junto do código. No MySQL autorizado, o equivalente é schema validado no servidor e acesso restrito às consultas/procedimentos privados.

React exibirá strings como texto com escape automático, sem `dangerouslySetInnerHTML` e sem handlers JavaScript construídos por concatenação. Isso protege também nomes com apóstrofos como Joana D'arc, sem depender de `safeJS` em HTML inline.

Reserva e reagendamento exigem transação, verificação de concorrência e gravação integral de dados e blocos. Uma simples escrita em lote não substitui o controle de conflito; falhas revertem todos os efeitos. Registros históricos preservam preços, duração e adicionais da ocasião.

## Design aprovado no documento de origem

**Movimento:** luxo editorial contemporâneo, traduzido para uma ferramenta leve e acessível no celular.

**Princípios:** hierarquia clara; refinamento sem excesso; legibilidade e contraste; agendamento progressivo com feedback imediato.

**Cores:** Azul Imperial profundo como assinatura, Dourado Nobre em acentos, Grafite para texto e Bronze em detalhes. Fundos marfim/brancos equilibram a paleta e reduzem esforço visual. Verde/vermelho/cinza/laranja ficam reservados aos significados operacionais especificados.

**Layout:** vitrine com cabeçalho de marca, introdução editorial compacta e fluxo vertical de agendamento; desktop pode manter resumo lateral. Painel com barra lateral imperial, área de trabalho clara e cabeçalho contextual; celular usa navegação compacta, cartões empilhados e controles confortáveis ao toque.

**Elementos de assinatura:** monograma SN delineado, filetes dourados, títulos editoriais combinados com indicadores compactos de agenda.

**Interação e movimento:** botões com estados hover/pressed, sombras suaves, transições de 160–220ms, indicadores de carregamento e confirmação. Respeitar `prefers-reduced-motion`; nada de animações que escondam informações essenciais ou atrasem o fluxo.

**Tipografia:** Cormorant Garamond para marca e títulos expressivos; Manrope para campos, indicadores e leitura funcional. Hierarquia com títulos concisos e microcopy legível no celular.

**Essência:** cuidado de unhas premium com uma agenda tão organizada quanto o atendimento. Personalidade: refinada, acolhedora, precisa.

**Voz:** direta e gentil. Exemplos: “Seu próximo momento de cuidado.” e “Escolha o horário que combina com você.”

**Wordmark:** Suzanny em composição editorial com monograma SN e subtítulo Nail Designer; não utilizar logo de outro negócio. Cor assinatura: Azul Imperial.

## Estrutura de projeto

- `client/src/pages/`: vitrine/agendamento, entrada da equipe, agenda, finanças/CRM, serviços, bloqueios e equipe.
- `client/src/components/`: identidade, layout privado, passos do agendamento, grade, cartões, formulários e modais reutilizáveis.
- `client/src/lib/`: cliente tRPC, estado de autenticação, adaptador de disponibilidade em tempo real e utilitários de formatação.
- `shared/`: schemas, contratos, status, cálculos de minutos, datas e totais compartilháveis sem segredos.
- `server/routers.ts` e `server/`: APIs públicas restritas, APIs privadas, validação de sessão/papel, reservas transacionais e relatórios.
- `server/_core/`: infraestrutura Express, integração de sessão e inicialização, preservando suporte à Preview HTTPS e cookies seguros quando aplicáveis.
- Opção Firebase: `server/firebase/`, `client/src/lib/firebase.ts`, `firestore.rules`, `firestore.indexes.json` e documentação de configuração; credenciais só em ambiente protegido.
- Opção gerenciada autorizada: `drizzle/schema.ts`, migrações versionadas em `drizzle/` e consultas/transações em `server/db.ts` e módulos de domínio.
- `client/public/`: ícones, assets e `manus-routes.json` sincronizado com as páginas realmente implementadas.
- `app.config.ts`: logo público real da marca do projeto.
- `Dockerfile`: instalação fixada, build frontend/backend e servidor de produção que honra PORT, com `/api/health` público sem dados privados.
- `TODO.md`: resultados com cláusulas de aceitação preservadas, criado somente após aprovação; itens sem integração real permanecerão abertos.

## Fluxo de entrega

Após escolher a infraestrutura e aprovar o plano: registrar TODO, implementar interface e domínio em módulos, integrar autenticação e dados reais, verificar TypeScript e testes existentes/adicionados e fazer uma revisão somente leitura por subagente. Iniciar o servidor para Preview quando houver conteúdo funcional, verificar health e manifesto de rotas. Salvar a versão no histórico do projeto. A publicação é separada do checkpoint; não será anunciada sem confirmação de publicação bem-sucedida.

Configuração de Firebase e contatos do salão, caso ainda ausentes, será apresentada como pendência real, não como funcionalidade já conectada. Não lançar para clientes reais antes de configurar papéis administrativos, credenciais, regras e contatos.
