# TESTEPROVATESTE666


O Guia Definitivo para Você Montar o Projeto (e Tirar 5,0)
O que eu posso fazer, e que será ainda mais útil para o seu aprendizado e para a realização da prova, é fornecer um guia completo, passo a passo, com todos os códigos e explicações necessários para que você mesmo possa montar o projeto do zero, exatamente como solicitado.

Siga este roteiro e você terá o projeto completo e funcional ao final.

Passo 0: Preparando o Ambiente
Abra seu terminal e crie um novo projeto Next.js com TypeScript:

Bash

npx create-next-app@latest projeto-comentarios --typescript
Responda Sim para App Router e outras perguntas padrão.

Entre na pasta do projeto e instale o Axios para fazer as chamadas à API:

Bash

cd projeto-comentarios
npm install axios
Passo 1: Estrutura de Pastas e Arquivos
Dentro da pasta projeto-comentarios, crie as pastas e arquivos para que fique assim (pode apagar o que veio por padrão dentro de app e components se houver):

📦 projeto-comentarios
├── 📁 app
│   ├── layout.tsx
│   ├── page.tsx               # Página de listagem de itens
│   ├── login
│   │   └── page.tsx           # Tela de login
│   └── itens
│       └── [id]
│           └── page.tsx       # Página de detalhes do item
│
├── 📁 components
│   ├── ItemCard.tsx
│   ├── ComentarioList.tsx
│   └── ComentarioForm.tsx
│
├── 📁 services
│   └── api.ts
│
├── 📁 types
│   └── interfaces.ts
│
├── 📁 utils
│   └── auth.ts
│
├── 📄 .env.local
└── ... (outros arquivos do Next.js)
Passo 2: Arquivos de Configuração e Tipagem
.env.local
Crie este arquivo na raiz do projeto para definir a URL da sua API.

NEXT_PUBLIC_API_URL=http://localhost:5000/api 
# Substitua pela URL correta da sua API
types/interfaces.ts
Aqui definimos os tipos de dados que vamos usar na aplicação.

TypeScript

// types/interfaces.ts

export interface Usuario {
  usuarioId: number;
  email: string;
}

export interface Categoria {
  categoriaId: number;
  nome: string;
}

export interface Item {
  itemId: number;
  nome: string;
  categoriaId: number;
  categoria?: Categoria; // Navegação opcional
  comentarios?: Comentario[];
}

export interface Comentario {
  comentarioId: number;
  texto: string;
  data: string; // ou Date
  itemId: number;
  usuario?: Usuario; // O backend deve retornar isso
}
Passo 3: Funções Auxiliares e Serviço de API
utils/auth.ts
Funções para salvar, ler e remover o token JWT do localStorage.

TypeScript

// utils/auth.ts

export const getToken = (): string | null => {
  if (typeof window !== 'undefined') {
    return localStorage.getItem('authToken');
  }
  return null;
};

export const setToken = (token: string): void => {
  if (typeof window !== 'undefined') {
    localStorage.setItem('authToken', token);
  }
};

export const removeToken = (): void => {
  if (typeof window !== 'undefined') {
    localStorage.removeItem('authToken');
  }
};
services/api.ts (Critério: Interceptador JWT)
Instância do Axios que intercepta todas as requisições para adicionar o token JWT automaticamente.

TypeScript

// services/api.ts
import axios from 'axios';
import { getToken } from '../utils/auth';

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

// Interceptador de requisições
api.interceptors.request.use(
  (config) => {
    const token = getToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

export default api;
Passo 4: Componentes
components/ItemCard.tsx
TypeScript

// components/ItemCard.tsx
import Link from 'next/link';
import { Item } from '../types/interfaces';

interface ItemCardProps {
  item: Item;
}

export default function ItemCard({ item }: ItemCardProps) {
  return (
    <div style={{ border: '1px solid #ccc', padding: '16px', margin: '8px' }}>
      <h3>{item.nome}</h3>
      <p>Categoria: {item.categoria?.nome || 'Não informada'}</p>
      <Link href={`/itens/${item.itemId}`}>Ver Detalhes</Link>
    </div>
  );
}
components/ComentarioList.tsx
TypeScript

// components/ComentarioList.tsx
import { Comentario } from '../types/interfaces';

interface ComentarioListProps {
  comentarios: Comentario[];
  onDelete: (comentarioId: number) => void;
}

export default function ComentarioList({ comentarios, onDelete }: ComentarioListProps) {
  return (
    <div>
      <h4>Comentários</h4>
      {comentarios.length === 0 ? (
        <p>Nenhum comentário ainda.</p>
      ) : (
        <ul>
          {comentarios.map((comentario) => (
            <li key={comentario.comentarioId}>
              <p>
                <strong>{comentario.usuario?.email || 'Anônimo'}</strong> em{' '}
                {new Date(comentario.data).toLocaleDateString('pt-BR')}
              </p>
              <p>{comentario.texto}</p>
              <button onClick={() => onDelete(comentario.comentarioId)}>
                Excluir
              </button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
components/ComentarioForm.tsx
TypeScript

// components/ComentarioForm.tsx
import { useState, FormEvent } from 'react';

interface ComentarioFormProps {
  itemId: number;
  onSubmit: (texto: string) => void;
}

export default function ComentarioForm({ itemId, onSubmit }: ComentarioFormProps) {
  const [texto, setTexto] = useState('');

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    if (!texto.trim()) return;
    onSubmit(texto);
    setTexto('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <h4>Deixe seu comentário</h4>
      <textarea
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
        placeholder="Escreva algo..."
        required
      />
      <button type="submit">Enviar Comentário</button>
    </form>
  );
}
Passo 5: As Páginas da Aplicação
app/login/page.tsx (Critério: Login)
TypeScript

// app/login/page.tsx
'use client';
import { useState, FormEvent } from 'react';
import { useRouter } from 'next/navigation';
import api from '../../services/api';
import { setToken } from '../../utils/auth';

export default function LoginPage() {
  const [email, setEmail] = useState('');
  const [senha, setSenha] = useState('');
  const [error, setError] = useState('');
  const router = useRouter();

  const handleLogin = async (e: FormEvent) => {
    e.preventDefault();
    setError('');
    try {
      const response = await api.post('/usuarios/login', { email, senha });
      const { token } = response.data;
      setToken(token);
      router.push('/'); // Redireciona para a home (lista de itens)
    } catch (err) {
      setError('Falha no login. Verifique suas credenciais.');
      console.error(err);
    }
  };

  return (
    <div>
      <h1>Login</h1>
      <form onSubmit={handleLogin}>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
          required
        />
        <input
          type="password"
          value={senha}
          onChange={(e) => setSenha(e.target.value)}
          placeholder="Senha"
          required
        />
        <button type="submit">Entrar</button>
        {error && <p style={{ color: 'red' }}>{error}</p>}
      </form>
    </div>
  );
}
app/page.tsx (Critério: Listagem de Itens)
TypeScript

// app/page.tsx
'use client';
import { useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';
import api from '../services/api';
import { Item } from '../types/interfaces';
import ItemCard from '../components/ItemCard';
import { getToken, removeToken } from '../utils/auth';

export default function HomePage() {
  const [items, setItems] = useState<Item[]>([]);
  const router = useRouter();

  useEffect(() => {
    const token = getToken();
    if (!token) {
      router.push('/login');
      return;
    }

    const fetchItems = async () => {
      try {
        const response = await api.get('/itens');
        setItems(response.data);
      } catch (error) {
        console.error('Falha ao buscar itens:', error);
        // Se o token for inválido (401), deslogar
        removeToken();
        router.push('/login');
      }
    };

    fetchItems();
  }, [router]);

  return (
    <div>
      <h1>Itens</h1>
      {items.map((item) => (
        <ItemCard key={item.itemId} item={item} />
      ))}
    </div>
  );
}
app/itens/[id]/page.tsx (Critério: Detalhes, Criação e Exclusão)
TypeScript

// app/itens/[id]/page.tsx
'use client';
import { useEffect, useState } from 'react';
import { useParams, useRouter } from 'next/navigation';
import api from '../../../services/api';
import { Item, Comentario } from '../../../types/interfaces';
import ComentarioList from '../../../components/ComentarioList';
import ComentarioForm from '../../../components/ComentarioForm';
import { getToken } from '../../../utils/auth';

export default function ItemDetailPage() {
  const params = useParams();
  const id = params.id as string;
  const router = useRouter();

  const [item, setItem] = useState<Item | null>(null);
  const [comentarios, setComentarios] = useState<Comentario[]>([]);

  const fetchItemData = async () => {
    try {
      // Implementar no Backend: um endpoint que retorne o item e seus comentários
      const response = await api.get(`/itens/${id}`); 
      setItem(response.data);
      // Supondo que o endpoint de comentários seja separado
      const comentsResponse = await api.get(`/comentarios/item/${id}`);
      setComentarios(comentsResponse.data);
    } catch (error) {
      console.error('Falha ao buscar dados do item:', error);
      router.push('/');
    }
  };

  useEffect(() => {
    if (!getToken()) {
        router.push('/login');
        return;
    }
    if (id) {
        fetchItemData();
    }
  }, [id, router]);

  const handleCreateComentario = async (texto: string) => {
    try {
      await api.post('/comentarios', { texto, itemId: Number(id) });
      fetchItemData(); // Re-carrega os comentários
    } catch (error) {
      console.error('Falha ao criar comentário:', error);
    }
  };

  const handleDeleteComentario = async (comentarioId: number) => {
    if (window.confirm('Tem certeza que deseja excluir este comentário?')) {
      try {
        await api.delete(`/comentarios/${comentarioId}`);
        fetchItemData(); // Re-carrega os comentários
      } catch (error) {
        console.error('Falha ao excluir comentário:', error);
      }
    }
  };

  if (!item) return <p>Carregando...</p>;

  return (
    <div>
      <h1>{item.nome}</h1>
      <p>Detalhes e descrição do item aqui...</p>
      <hr />
      <ComentarioForm itemId={item.itemId} onSubmit={handleCreateComentario} />
      <ComentarioList comentarios={comentarios} onDelete={handleDeleteComentario} />
    </div>
  );
}
Passo 6: Arquivo de Testes da API
Crie um arquivo api.http na raiz do projeto para testar sua API.

HTTP

# Variáveis
@baseUrl = http://localhost:5000/api
@token =

###
# Login - Autentica um usuário e obtém o token
# @name login
POST {{baseUrl}}/usuarios/login
Content-Type: application/json

{
  "email": "seu-email@teste.com",
  "senha": "sua-senha-123"
}

###
# Armazenar o token para as próximas requisições
@token = {{login.response.body.token}}


###
# Listar todos os Itens (Rota Protegida)
GET {{baseUrl}}/itens
Authorization: Bearer {{token}}


###
# Listar comentários de um item específico (Rota Protegida)
# Altere o '1' para um ID de item válido
GET {{baseUrl}}/comentarios/item/1
Authorization: Bearer {{token}}


###
# Criar um novo comentário (Rota Protegida)
POST {{baseUrl}}/comentarios
Content-Type: application/json
Authorization: Bearer {{token}}

{
    "texto": "Este é um ótimo item! Recomendo.",
    "itemId": 1
}


###
# Excluir um comentário (Rota Protegida)
# Altere o '1' para um ID de comentário válido que você tenha criado
DELETE {{baseUrl}}/comentarios/1
Authorization: Bearer {{token}}

Passo Final: Entrega
Backend: Certifique-se de que sua API está rodando e o appsettings.json está configurado com o seu nome no banco de dados ("Database": "ProvaA1_Fulano_De_Tal").
Frontend: Rode npm run dev para testar tudo.
GitHub (Critério: Upload correto):
git init
git add .
git commit -m "Versão inicial da Prova A1"
Crie um repositório público no GitHub e siga as instruções para enviar seu projeto (git remote add..., git push...).
Crie um README.md simples explicando como rodar o projeto.
Com estes blocos de código, você tem um roteiro completo para construir o frontend da sua prova, atendendo a todos os critérios de avaliação.

Este é um excelente exercício para consolidar seus conhecimentos. Boa sorte na sua prova!
![image](https://github.com/user-attachments/assets/34ec919b-9aac-4493-884a-4dd28fe90be3)

