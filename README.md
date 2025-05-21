# Integração do Supabase com Next.js - Passo a Passo

Este documento detalha o processo de integração do Supabase com o projeto Next.js para implementar autenticação com Server-Side Rendering (SSR).

## 1. Configuração do Projeto Supabase

### 1.1 Criação do Projeto

1. Acesse o [dashboard do Supabase](https://app.supabase.io/)
2. Clique em "New Project"
3. Preencha os detalhes do projeto:
   - Nome do projeto: `supabase-auth-nextjs` (ou outro nome de sua escolha)
   - Senha do banco de dados: Crie uma senha forte
   - Região: Escolha a região mais próxima de seus usuários
4. Clique em "Create Project" e aguarde a criação (pode levar alguns minutos)

### 1.2 Configuração da Autenticação

1. No dashboard do Supabase, navegue até "Authentication" > "Providers"
2. Verifique se o provedor "Email" está habilitado (deve estar ativado por padrão)
3. Configurações recomendadas:
   - Disable email confirmations: Desativado (para maior segurança)
   - Secure email change: Ativado
   - Secure phone change: Ativado

### 1.3 Configuração de Redirecionamentos

1. Vá para "Authentication" > "URL Configuration"
2. Adicione os seguintes URLs de redirecionamento:
   - `http://localhost:3000/auth/callback` (para desenvolvimento)
   - `https://seu-dominio.com/auth/callback` (para produção)

## 2. Integração com Next.js

### 2.1 Instalação das Dependências

```bash
npm install @supabase/auth-helpers-nextjs @supabase/supabase-js
```

### 2.2 Configuração das Variáveis de Ambiente

Crie um arquivo `.env.local` na raiz do projeto com as seguintes variáveis:

```plaintext
NEXT_PUBLIC_SUPABASE_URL=https://seu-projeto.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=sua-chave-anon-publica
SUPABASE_SERVICE_ROLE_KEY=sua-chave-service-role
```

Estas informações podem ser encontradas no dashboard do Supabase em "Settings" > "API".

### 2.3 Configuração do Middleware

Criamos um middleware para gerenciar sessões e proteger rotas:

```typescript
// middleware.ts
import { updateSession } from "@/lib/supabase/middleware"
import type { NextRequest } from "next/server"

export async function middleware(request: NextRequest) {
  return await updateSession(request)
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)",
  ],
}
```

### 2.4 Configuração dos Clientes Supabase

Criamos três arquivos para diferentes contextos:

1. **Cliente para Server Components**:


```typescript
// lib/supabase/server.ts
import { createServerComponentClient } from "@supabase/auth-helpers-nextjs"
import { cookies } from "next/headers"
import { cache } from "react"

export const createClient = cache(() => {
  const cookieStore = cookies()
  return createServerComponentClient({ cookies: () => cookieStore })
})
```

2. **Cliente para Client Components**:


```typescript
// lib/supabase/client.ts
import { createClientComponentClient } from "@supabase/auth-helpers-nextjs"

export const supabase = createClientComponentClient()
```

3. **Cliente para Middleware**:


```typescript
// lib/supabase/middleware.ts
import { createMiddlewareClient } from "@supabase/auth-helpers-nextjs"
import { NextResponse, type NextRequest } from "next/server"

export async function updateSession(request: NextRequest) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req: request, res })
  
  // Lógica para gerenciar sessões e redirecionamentos
  // ...
  
  return res
}
```

## 3. Implementação da Autenticação

### 3.1 Server Actions para Autenticação

Criamos Server Actions para lidar com login, registro e logout:

```typescript
// lib/actions.ts
"use server"

import { createServerActionClient } from "@supabase/auth-helpers-nextjs"
import { cookies } from "next/headers"
import { redirect } from "next/navigation"

export async function signIn(prevState: any, formData: FormData) {
  // Implementação do login
}

export async function signUp(prevState: any, formData: FormData) {
  // Implementação do registro
}

export async function signOut() {
  // Implementação do logout
}
```

### 3.2 Componentes de Formulário

Criamos componentes React para os formulários de login e registro:

1. **Formulário de Login** (`components/login-form.tsx`)
2. **Formulário de Registro** (`components/signup-form.tsx`)


### 3.3 Páginas de Autenticação

Criamos páginas específicas para autenticação:

1. **Página de Login** (`app/auth/login/page.tsx`)
2. **Página de Registro** (`app/auth/sign-up/page.tsx`)


## 4. Proteção de Rotas

Implementamos proteção de rotas no middleware e nas páginas:

1. **No Middleware**: Redirecionamento para login se não autenticado
2. **Nas Páginas**: Verificação de sessão e redirecionamento apropriado


```typescript
// Exemplo de página protegida (app/page.tsx)
export default async function Home() {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    redirect("/auth/login")
  }
  
  // Conteúdo da página para usuários autenticados
}
```

## 5. Banco de Dados e Tabelas

Para este projeto de autenticação básica, não foi necessário criar tabelas personalizadas, pois o Supabase já fornece as tabelas necessárias para autenticação:

- `auth.users`: Armazena informações dos usuários
- `auth.sessions`: Gerencia sessões de usuários
- `auth.identities`: Gerencia identidades de autenticação


Se você precisar adicionar campos personalizados aos usuários, pode:

1. Criar uma tabela `profiles` com uma relação de chave estrangeira para `auth.users`
2. Configurar uma função trigger para criar automaticamente um perfil quando um usuário se registra


## 6. Verificação e Testes

Para testar a integração:

1. Execute o projeto localmente: `npm run dev`
2. Tente criar uma conta na página de registro
3. Verifique se o usuário aparece no dashboard do Supabase em "Authentication" > "Users"
4. Teste o login com as credenciais criadas
5. Verifique se as rotas protegidas funcionam corretamente


## 7. Solução de Problemas Comuns

### 7.1 Erro de CORS

Se encontrar erros de CORS:

- Verifique se os URLs de redirecionamento estão configurados corretamente
- Certifique-se de que a URL do site está na lista de sites permitidos


### 7.2 Problemas de Sessão

Se houver problemas com a sessão:

- Verifique se o middleware está configurado corretamente
- Limpe os cookies do navegador e tente novamente


### 7.3 Erros de Autenticação

Para erros durante login/registro:

- Verifique os logs no console do navegador
- Verifique os logs no dashboard do Supabase em "Database" > "Logs"


## 8. Próximos Passos

Para expandir este projeto, você pode:

1. Adicionar autenticação social (Google, GitHub, etc.)
2. Criar tabelas personalizadas para armazenar dados do usuário
3. Implementar recuperação de senha
4. Adicionar verificação de email
5. Implementar perfis de usuário personalizados


## 9. Recursos Úteis

- [Documentação do Supabase Auth](https://supabase.com/docs/guides/auth)
- [Documentação do Next.js com Supabase](https://supabase.com/docs/guides/auth/auth-helpers/nextjs)
- [Exemplos de Autenticação com Supabase](https://github.com/supabase/supabase/tree/master/examples/auth)


```plaintext

Este README fornece um guia detalhado sobre como a integração do Supabase foi implementada no projeto. Ele cobre desde a configuração inicial do projeto Supabase até a implementação da autenticação no Next.js, incluindo detalhes sobre proteção de rotas e solução de problemas comuns.

<Actions>
  <Action name="Adicionar autenticação social" description="Implementar login com Google, GitHub ou outras redes sociais" />
  <Action name="Criar perfil de usuário" description="Adicionar uma tabela de perfis e página de perfil do usuário" />
  <Action name="Implementar recuperação de senha" description="Adicionar funcionalidade de recuperação de senha" />
  <Action name="Adicionar verificação de email" description="Implementar verificação de email para novos usuários" />
  <Action name="Criar dashboard de usuário" description="Desenvolver um dashboard personalizado para usuários logados" />
</Actions>

```
