# Como executar o MarketGo (checklist)

1. **Node 22+** (`node -v`) e o app **Expo Go** no celular (mesma rede Wi-Fi do computador).
2. Na pasta do projeto: `npm install`
3. **Supabase** (supabase.com), projeto novo `marketgo`:
   - SQL Editor: rode `supabase/schema.sql` e depois `supabase/seed.sql`
     (Table Editor deve mostrar 6 lojas e 50 produtos).
   - Authentication > Sign In / Providers > Email: desligue **Confirm email** (apenas para testes).
   - Project Settings > API: copie a **Project URL** e a chave **anon/publishable**.
4. Copie `.env.example` para `.env` e preencha:
   ```
   EXPO_PUBLIC_SUPABASE_URL=https://SEU-PROJETO.supabase.co
   EXPO_PUBLIC_SUPABASE_KEY=SUA-CHAVE-ANON-OU-PUBLISHABLE
   ```
   Nunca use a chave `service_role` e nunca envie o `.env` ao GitHub.
5. `npx expo start -c` e escaneie o QR Code. Se a rede bloquear: `npx expo start --tunnel`.
6. Antes de entregar: `npm run typecheck` e `npm test` (37 testes do app + 13 do banco).

## Problemas já conhecidos (e solução)

| Sintoma | Solução |
| --- | --- |
| `ERESOLVE` ao instalar `@hookform/resolvers` | O projeto já usa `@hookform/resolvers@^5` (a 6.x conflita com `effect`). |
| "Could not find the function ... in the schema cache" | Rode `notify pgrst, 'reload schema';` no SQL Editor. |
| "Variáveis do Supabase ausentes" | Crie o `.env` e reinicie com `npx expo start -c`. |
| Jest tentando rodar `db.test.mjs` | Já ignorado em `package.json` (`testPathIgnorePatterns`). |
