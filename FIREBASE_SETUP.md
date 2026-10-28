# Configuração do Firebase — Suzanny Nail Designer

O app já lê as configurações do Firebase pelo ambiente do WebDev e mantém a chave privada apenas no armazenamento protegido do projeto. Nunca coloque o JSON da conta de serviço, a chave privada ou valores `.env` no Git.

## 1. Ative os produtos

No console do projeto informado na configuração protegida, ative **Authentication > Sign-in method > Email/Password** e crie um banco **Cloud Firestore** em produção. O app usa as coleções `services`, `appointments`, `public_slots`, `blocks`, `public_blocks` e `team`.

## 2. Publique regras e índices

Com o Firebase CLI autenticado e apontando para o projeto correto, execute na raiz do app:

```bash
firebase use SEU_PROJECT_ID
firebase deploy --only firestore:rules,firestore:indexes
```

As regras permitem leitura pública somente de `public_slots`, `public_blocks` e serviços ativos. Dados de clientes, bloqueios privados e equipe ficam fechados ao navegador; o servidor usa Firebase Admin com a conta de serviço protegida.

## 3. Crie a primeira administradora

Cadastre a conta da Suzanny em Authentication. Depois, no Firestore, crie `team/<email-em-minúsculas>` com:

```json
{
  "email": "suzanny@exemplo.com",
  "name": "Suzanny",
  "role": "admin",
  "active": true
}
```

A partir daí, a Suzanny pode cadastrar recepção e demais administradoras na aba Equipe. O primeiro acesso cria a senha no fluxo de e-mail/senha; a autorização continua sendo conferida no servidor pelo documento `team`.

## 4. Autorize a conta de serviço no Firestore

A chave privada usada pelo servidor deve pertencer a uma conta de serviço com acesso ao banco Firestore. No **Google Cloud Console > IAM**, localize o e-mail configurado em `FIREBASE_CLIENT_EMAIL` e atribua, no mínimo, o papel **Cloud Datastore User** (`roles/datastore.user`) no projeto Firebase. Depois de salvar, recarregue o painel; ele valida uma leitura mínima antes de liberar o login.

## 5. Contatos da vitrine

Para substituir os links de fallback do rodapé, configure no ambiente do frontend os valores `VITE_BUSINESS_WHATSAPP`, `VITE_INSTAGRAM_URL` e `VITE_MAPS_URL`. Sem esses valores, o app não inventa telefone, endereço ou perfil: os links exibem o destino de configuração correspondente.
