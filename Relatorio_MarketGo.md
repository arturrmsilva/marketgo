# Relatório de Soluções de Correção — Projeto MarketGo (React Native)

**Disciplina:** Programação para Dispositivos Móveis · **Professor:** Prof. Me. Bruno Assunção Dias · **Instituição:** IESB · **Aluno:** Artur (completar nome completo e matrícula) · **Entrega:** 28/10, 23:59

## 1. Objetivo e escopo

Este relatório registra os erros encontrados durante a construção do MarketGo, os prompts usados com as IAs, as respostas recebidas e como cada solução foi implementada para o projeto seguir adiante. O projeto implementa o aplicativo descrito no guia do professor: cadastro e login, catálogo de lojas, carrinho persistente (uma loja por vez), checkout com validação, pedidos com linha do tempo, cancelamento e "Pedir novamente". O total do pedido é calculado no banco por funções SQL, e o app não escreve direto nas tabelas de pedidos.

**Tecnologias:** React Native, Expo, Expo Router, TypeScript, Supabase (Auth, Postgres, RLS, funções SQL), TanStack Query, React Hook Form, Zod, Jest e PGlite.

## 2. Ferramentas de IA utilizadas e divisão do trabalho

| Fase | IA | O que foi feito |
| --- | --- | --- |
| Montagem do projeto, testes automáticos e correção de erros de compilação | Claude (Anthropic) | Leitura do guia em PDF, criação de todos os arquivos, instalação das dependências, execução de typecheck, Jest, testes do banco (PGlite) e empacotamento Android |
| Execução no celular/Supabase e correções que surgirem | Gemini (Google) | Ver seção 6 |

## 3. Prompts utilizados com o Claude

**Prompt 1.** Enviado com a imagem do enunciado (Projeto MarketGo React Native, 100 pontos) e o PDF "MarketGo-Guia-Passo-a-Passo":

> "esse é o projeto 2 do professor, faça ele por favor. a parte de rodar o sistema eu faço depois com a ajuda do gemini"

**Resposta resumida da IA:** o Claude leu as 104 páginas do guia, extraiu os 66 blocos de código e os organizou na estrutura de pastas do Apêndice A. Em seguida criou o projeto Expo, instalou as bibliotecas, formatou o código com Prettier e rodou as verificações. Os problemas encontrados no caminho estão na seção 4.

## 4. Erros encontrados e soluções implementadas

Todos os erros abaixo ocorreram de fato durante a montagem e foram reproduzidos e corrigidos com as ferramentas do projeto.

### Erro 1 — Versões do template diferentes das do guia

**Onde:** Etapa 2.1 (`npx create-expo-app@latest marketgo --template blank-typescript`).
**O que aconteceu:** o template gerou Expo SDK 58, React Native 0.88, React 19.3 e TypeScript 6.0. O guia foi escrito com Expo SDK 57, React Native 0.86 e React 19.2.
**Impacto:** as versões novas expõem diferenças de tipagem, descritas nos erros 4 e 5. O restante do código funcionou sem mudanças.
**Solução:** manter as versões mais novas e corrigir apenas o que a verificação de tipos apontou. As versões ficaram registradas no `package.json` e no `package-lock.json`.

### Erro 2 — `expo install` antes de instalar as dependências

**Mensagem:** `ConfigError: Cannot determine the project's Expo SDK version because the module expo is not installed.`
**Causa:** o projeto foi criado com `--no-install`, então o pacote `expo` ainda não existia em `node_modules`.
**Solução:** rodar `npm install` primeiro e só depois `npx expo install ...`.

### Erro 3 — Conflito de dependências (`ERESOLVE`) em `@hookform/resolvers`

**Comando:** `npm install react-hook-form zod @hookform/resolvers`
**Mensagem (resumida):** `ERESOLVE unable to resolve dependency tree ... peerOptional effect@"^4.0.0" from @hookform/resolvers@6.0.0 ... Found: effect@3.22.4`
**Causa:** a versão 6.0.0 do `@hookform/resolvers`, instalada por padrão, declara uma dependência opcional (`effect` 4.x) incompatível com a `effect` 3.x que já estava na árvore. Não é o problema de `react` / `react-dom` citado no guia.
**Solução:** fixar a versão compatível, sem recorrer a `--legacy-peer-deps`: `npm install react-hook-form zod @hookform/resolvers@^5`. Foi instalada a 5.x e o uso com `zodResolver` continuou igual.

### Erro 4 — Tipo do ícone das abas (TS2322)

**Arquivo:** `app/(app)/(tabs)/_layout.tsx`
**Mensagem:** `Type '____ColorValue_Internal' is not assignable to type 'string | NativeColorValue | undefined'. Type 'null' is not assignable ...`
**Causa:** o tipo de cor que o `tabBarIcon` entrega nesta versão aceita `null` e `number`, mas o componente `Ionicons` não aceita esses valores.
**Solução:** manter o parâmetro como `ColorValue` e converter na hora de usar:

```tsx
<Ionicons name={name} size={size}
  color={typeof color === 'string' ? color : colors.muted} />
```

Uma primeira tentativa (trocar o tipo do parâmetro para `string`) gerou novo erro, porque o `Tabs` entrega `ColorValue`. Uma segunda (`color ?? undefined`) ainda deixava passar `number`. A versão final com `typeof` resolveu.

### Erro 5 — `onRefresh={refetch}` (TS2769)

**Arquivos:** `app/(app)/(tabs)/index.tsx` e `app/(app)/(tabs)/orders.tsx`
**Mensagem:** `Type '(options?) => Promise<QueryObserverResult<...>>' is not assignable to type '() => void | Promise<void>'`
**Causa:** o `refetch` do TanStack Query devolve uma Promise com o resultado da consulta, e o `RefreshControl` espera `Promise<void>`.
**Solução:** `onRefresh={() => void refetch()}`.

### Erro 6 — Jest executando o teste do banco

**Comando:** `npm run test:app` (e `npm test`)
**Mensagem:** `Must use import to load ES Module: .../supabase/tests/db.test.mjs`
**Causa:** o Jest, por padrão, também procura arquivos `.mjs` e tentou rodar o teste do banco, que usa o executor nativo do Node.
**Solução:** excluir a pasta do Supabase do Jest, em `package.json`:

```json
"jest": {
  "preset": "jest-expo",
  "moduleNameMapper": { "^@/(.*)$": "<rootDir>/src/$1" },
  "testPathIgnorePatterns": ["/node_modules/", "<rootDir>/supabase/"]
}
```

### Observações menores

- O template do Expo gera um arquivo `AGENTS.md` que manda colocar as rotas em `src/app/`. O guia usa `app/` na raiz, então o arquivo não foi incluído e a estrutura do guia foi mantida.
- Ao copiar o código do PDF, algumas linhas em branco se perderam (por exemplo, no README). O `npm run format` (Prettier) e um ajuste manual no README resolveram.
- Foram personalizados o autor e os links do README e os identificadores `com.arturrmsilva.marketgo` no `app.json`, como o guia orienta.

## 5. Resultado das verificações

| Verificação | Comando | Resultado |
| --- | --- | --- |
| Tipos | `npm run typecheck` | Sem erros |
| Regras do app | `npm run test:app` | 5 suítes, 37 testes passando |
| Banco de dados (PGlite) | `npm run test:db` | 13 testes passando, 0 falhas |
| Instalação limpa | `npm ci` em cópia sem `node_modules` | Instala e passa nos testes |
| Empacotamento Android | `npx expo export --platform android` | Bundle gerado com sucesso |

As verificações acima não incluem a execução no celular nem a conexão com um projeto Supabase real. Essas etapas estão na seção 6.

## 6. Erros durante a execução no celular e no Supabase (Gemini)

Esta seção sintetiza as principais adequações técnicas e correções de ambiente realizadas durante a implantação do projeto MarketGo, na etapa de execução no celular e de conexão com o Supabase, conduzida com o apoio do Gemini. A Tabela 1 apresenta, para cada ocorrência, a causa raiz identificada, a solução aplicada e o resultado obtido, servindo como registro de troubleshooting e como referência para a reprodução futura do ambiente de desenvolvimento.

**Tabela 1 – Problemas identificados, soluções aplicadas e resultados**

| Nº | Problema | Causa raiz | Solução aplicada | Resultado obtido |
| --- | --- | --- | --- | --- |
| 1 | Bloqueio de scripts do PowerShell | Política de execução padrão do Windows impedia a execução de comandos do Node e do Expo. | Ajuste da política de execução por meio do comando `Set-ExecutionPolicy`. | Comandos `npx` e `npm` liberados no terminal do VS Code. |
| 2 | Conflito de sincronização do OneDrive | Projeto armazenado em pasta sincronizada pelo OneDrive, o que corrompeu o diretório `node_modules`. | Limpeza de cache, remoção da pasta corrompida e reinstalação das dependências. | Metro Bundler funcional, sem travamentos por arquivos bloqueados. |
| 3 | Alertas visuais no `schema.sql` (VS Code) | Extensão do MSSQL tentando validar sintaxe e funções nativas do PostgreSQL. | Identificação da incompatibilidade do validador e verificação direta no Supabase SQL Editor. | Tabelas, funções RPC e políticas RLS criadas e aplicadas com sucesso. |
| 4 | Carga inicial de dados (`seed.sql`) | Necessidade de massa de dados para testar catálogo e pedidos. | Execução do `seed.sql` no SQL Editor do Supabase. | Cadastro de 6 lojas e 50 produtos na base. |
| 5 | Bloqueio no cadastro de usuários | Exigência padrão de confirmação por e-mail do Supabase Auth. | Desativação da opção *Confirm email* em *Authentication > Providers > Email*. | Registro e login imediatos, sem necessidade de e-mail real. |
| 6 | Falha de integração entre app e backend | Ausência das credenciais de acesso no ambiente local do Expo. | Renomeação de `.env.example` para `.env`, com `EXPO_PUBLIC_SUPABASE_URL` e `EXPO_PUBLIC_SUPABASE_KEY` (Publishable/Anon), e execução de `npx expo start -c`. | Conexão com o Supabase estabelecida. |
| 7 | Erro de execução no iOS (Expo Go) | Exigência do iOS de pareamento entre a conta do terminal e a do dispositivo. | Autenticação com `npx expo login` no terminal e login na mesma conta no Expo Go. | Aplicação executada normalmente em dispositivos iOS. |

## 7. Conclusão

O projeto foi montado a partir do guia do professor e validado por tipos, 50 testes automatizados (37 do app e 13 do banco) e empacotamento Android. Os seis erros de montagem foram causados principalmente por versões mais novas das bibliotecas que as usadas no guia, e cada um foi corrigido sem alterar as regras de negócio. Na execução no celular e no Supabase, as sete ocorrências registradas na seção 6 foram de ambiente e configuração (política do PowerShell, sincronização do OneDrive, credenciais, autenticação e pareamento no iOS) e também foram resolvidas sem alterar o código da aplicação. A arquitetura de segurança do guia foi mantida: o app envia só ids e quantidades, o banco calcula os preços e não existe política de escrita direta em `orders` e `order_items`.
