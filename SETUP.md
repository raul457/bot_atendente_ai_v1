# Sofia Bot — Guia de Instalação

## Pré-requisitos

- Node.js 20+
- PostgreSQL (ou conta Supabase gratuita)
- Redis (ou conta Upstash gratuita)
- Conta Anthropic (chave de API)
- Evolution API ou Z-API configurado

---

## 1. Banco de dados (Supabase — recomendado)

1. Acesse supabase.com e crie um projeto
2. Copie a `DATABASE_URL` (Connection String > URI)

---

## 2. Backend

```bash
cd backend
cp .env.example .env
# Preencha todas as variáveis no .env

npm install
npx prisma generate
npx prisma migrate dev --name init
npm run db:seed          # Cria admin e psicóloga de teste
npm run dev
```

Servidor rodando em: http://localhost:3001

### Credenciais do seed
- Admin: admin@clinicasofia.com / admin123
- Psicóloga: psi@clinicasofia.com / admin123

---

## 3. Frontend

```bash
cd frontend
cp .env.example .env.local
# .env.local já tem os valores padrão para desenvolvimento

npm install
npm run dev
```

Painel disponível em: http://localhost:3000

---

## 4. WhatsApp (Evolution API)

### Instalação local via Docker:
```bash
docker run -d \
  --name evolution-api \
  -p 8080:8080 \
  -e AUTHENTICATION_API_KEY=seu_token_aqui \
  atendai/evolution-api:latest
```

### Criar instância:
```
POST http://localhost:8080/instance/create
Headers: { apikey: seu_token_aqui }
Body: { "instanceName": "sofia", "qrcode": true }
```

### Configurar webhook para o backend:
```
POST http://localhost:8080/webhook/set/sofia
Body: {
  "url": "http://seu_servidor:3001/webhook",
  "webhook_by_events": false,
  "events": ["MESSAGES_UPSERT"]
}
```

---

## 5. Variáveis de ambiente obrigatórias (.env)

| Variável | Descrição |
|---|---|
| ANTHROPIC_API_KEY | Chave da API Anthropic |
| WHATSAPP_API_URL | URL do servidor Evolution API |
| WHATSAPP_API_TOKEN | Token da Evolution API |
| DATABASE_URL | URL do PostgreSQL (Supabase) |
| REDIS_URL | URL do Redis (Upstash) |
| JWT_SECRET | Chave secreta JWT (min 32 chars) |
| ADMIN_NOTIFICATION_PHONE | Número para notificações (55+DDD+número) |
| CLINIC_NAME | Nome da clínica |
| CLINIC_ADDRESS | Endereço físico |
| TELECONSULT_URL | Link padrão para teleconsultas |

---

## 6. Deploy (produção)

**Backend:** Railway ou Render
- Conecte o repositório
- Configure as variáveis de ambiente
- O Railway detecta o package.json automaticamente

**Frontend:** Vercel
- Conecte o repositório
- Configure: `NEXT_PUBLIC_API_URL=https://seu-backend.railway.app`
- Build command: `npm run build`
- Output directory: `.next`

---

## Estrutura do projeto

```
botwhats/
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma      # Modelos do banco de dados
│   │   └── seed.ts            # Dados iniciais
│   └── src/
│       ├── config/env.ts      # Validação de variáveis de ambiente
│       ├── controllers/       # Lógica das rotas
│       ├── services/          # Claude, WhatsApp, Redis, notificações
│       ├── queues/            # BullMQ — lembretes e notificações
│       ├── middlewares/       # Autenticação JWT
│       ├── routes/            # Registro de rotas
│       ├── websocket/         # Socket.io para o painel
│       └── server.ts          # Entrada da aplicação
└── frontend/
    └── src/
        ├── app/
        │   ├── login/         # Tela de login
        │   └── dashboard/     # Painel admin
        │       ├── page.tsx           # Dashboard com métricas
        │       ├── appointments/      # Gestão de consultas
        │       ├── patients/          # Gestão de pacientes
        │       ├── slots/             # Gestão de horários
        │       └── settings/          # Cadastro de usuários
        ├── lib/               # API client, Socket.io, utils
        └── types/             # Tipos TypeScript
```
