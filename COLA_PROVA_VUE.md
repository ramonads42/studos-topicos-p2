# COLA DE PROVA — Vue 3 + Axios (modo Lego)

> Abra este arquivo no VS Code ao lado da prova (Ctrl+Shift+V mostra bonitinho).
> Você **não precisa entender o código**: copie, cole e **troque só o que está marcado com `<-- TROQUE`**.

---

## 0. As 5 regras que evitam 90% dos erros

1. Um arquivo `.vue` tem **um** `<template>` em cima e **um** `<script setup lang="ts">` embaixo. Nunca um dentro do outro.
2. **Nomes têm que ser idênticos** aos do Java/JSON (maiúscula e minúscula importam: `dataHoraEntrega` ≠ `datahoraentrega`).
3. Variável com `ref` → no `<script>` usa `.value`, no `<template>` **não** usa `.value`.
4. A rota do axios é **sem barra no começo**: `axios.get('trabalho')` ✅ — `axios.get('/trabalho')` ❌ (funciona às vezes, mas siga o professor).
5. Backend precisa estar rodando (e o banco criado) **antes** de testar o Vue.

---

## 1. Roteiro da prova (siga na ordem)

| Passo | O que fazer |
|---|---|
| 1 | Clone os repositórios (backend e vue). Crie o **banco de dados** antes de rodar o backend. Se der erro de versão do Java: `mvn clean` |
| 2 | Rode o backend. Abra no navegador `http://localhost:8080/ROTA` (ex.: `trabalho`) e **olhe o JSON**: anote os nomes exatos dos campos |
| 3 | Abra o `*Controller.java` do enunciado e leia a **seção 2 abaixo** (tradução Java → Vue) |
| 4 | No projeto Vue: `npm install` e depois `npm run dev` |
| 5 | **Item 1:** Lego A (rota + link no menu) |
| 6 | Crie o arquivo `src/views/NomeView.vue` e cole o **Lego B (arquivo completo)** |
| 7 | Troque os nomes marcados com `<-- TROQUE` (rota, campos, textos) |
| 8 | Teste cada item (2, 3 e 4) no navegador. Aperte **F12 → Console** para ver erros |
| 9 | Se algo quebrar: seção 7 (erros comuns) |

---

## 2. Lendo o Controller Java (tradução para Vue)

Abra o Controller e procure estas linhas:

| No Java | Significa | No Vue |
|---|---|---|
| `@RequestMapping("/trabalho")` | rota base | `axios.get('trabalho')` |
| `@GetMapping` (sem nada) | lista tudo | `axios.get('trabalho')` |
| `@PostMapping` | cadastra | `axios.post('trabalho', objeto)` |
| `@GetMapping("/buscar")` | consulta | `axios.get('trabalho/buscar', { params: {...} })` |
| `@RequestParam("ra")` | parâmetro na URL | `params: { ra: valor }` ← o nome **tem** que ser igual |
| `@PathVariable Long id` | id dentro da URL | ``axios.get(`trabalho/${id}`)`` |
| `@RequestBody Trabalho t` | recebe JSON | o objeto que você manda no `post`/`put` |
| `@PutMapping("/{id}")` | edita | ``axios.put(`trabalho/${id}`, objeto)`` |
| `@DeleteMapping("/{id}")` | exclui | ``axios.delete(`trabalho/${id}`)`` |

**Tipo do campo na Entity → tipo do `<input>`:**

| Java | HTML |
|---|---|
| `String` | `<input type="text">` |
| `Integer`, `Long` | `<input type="number">` |
| `Double`, `BigDecimal` | `<input type="number" step="any">` |
| `LocalDateTime` | `<input type="datetime-local">` |
| `LocalDate` | `<input type="date">` |
| `LocalTime` | `<input type="time">` |
| `Boolean` | `<input type="checkbox">` |
| outra Entity (ex.: `Aluno aluno`) | `<select>` que guarda só o **id** (ver Lego B) |

> Dica: veja na Entity quais campos têm `@NotNull`, `@NotBlank` ou `nullable = false`. São os **obrigatórios**.

---

## 3. LEGO A — Item 1: rota e link no menu

### 3.1 `src/router/index.ts` — adicione um bloco dentro de `routes: [ ... ]`
(lembre da **vírgula** entre os blocos)

```typescript
    {
      path: '/trabalhos',                                   // <-- TROQUE (url da página)
      name: 'trabalhos',                                    // <-- TROQUE
      component: () => import('../views/TrabalhoView.vue') // <-- TROQUE (nome do SEU arquivo)
    },
```

### 3.2 `src/App.vue` — adicione dentro de `<nav> ... </nav>`
O texto tem que ser **exatamente** o do enunciado:

```html
<RouterLink to="/trabalhos">Gerencie Trabalhos</RouterLink>
```
(o `to` precisa ser igual ao `path` da rota)

---

## 4. LEGO B — A PÁGINA COMPLETA (itens 2, 3 e 4 funcionando)

Crie `src/views/TrabalhoView.vue` e cole **tudo**. Está pronto para a prova de exemplo (Trabalho + Aluno).
Confira os 3 pontos `<-- TROQUE`:  rota principal, rota do select, nomes dos campos.

```vue
<template>
  <div>
    <h1>Gerencie Trabalhos</h1>

    <!-- Mensagem de erro (aparece em vermelho) -->
    <p v-if="erro" style="color: red">{{ erro }}</p>

    <!-- ================= ITEM 3: CADASTRO ================= -->
    <h2>Novo trabalho</h2>

    <p>
      <label>Título: </label>
      <input type="text" v-model="novo.titulo" />
    </p>

    <p>
      <label>Data/hora de entrega: </label>
      <input type="datetime-local" v-model="novo.dataHoraEntrega" />
    </p>

    <p>
      <label>Descrição: </label>
      <input type="text" v-model="novo.descricao" />
    </p>

    <p>
      <label>Nota: </label>
      <input type="number" step="any" v-model="novo.nota" />
    </p>

    <p>
      <label>Justificativa: </label>
      <input type="text" v-model="novo.justificativa" />
    </p>

    <p>
      <label>Aluno: </label>
      <select v-model="novo.aluno.id">
        <option :value="null" disabled>Selecione...</option>
        <option v-for="a in alunos" :key="a.id" :value="a.id">
          {{ a.ra }} - {{ a.nome }}
        </option>
      </select>
    </p>

    <button @click="cadastrar">Cadastrar</button>

    <!-- ================= ITEM 4: CONSULTA ================= -->
    <h2>Consultar</h2>

    <p>
      <label>RA do aluno: </label>
      <input type="number" v-model="buscaRa" />
    </p>

    <p>
      <label>Título do trabalho: </label>
      <input type="text" v-model="buscaTitulo" />
    </p>

    <button @click="pesquisar">Pesquisar</button>
    <button @click="listarTodos">Mostrar todos</button>

    <!-- ================= ITEM 2: TABELA ================= -->
    <h2>Trabalhos</h2>

    <table v-if="lista.length > 0">
      <thead>
        <tr>
          <th>Id</th>
          <th>Título</th>
          <th>Status</th>
          <th>Aluno</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="t in lista" :key="t.id">
          <td>{{ t.id }}</td>
          <td>{{ t.titulo }}</td>
          <td>{{ calcularStatus(t.dataHoraEntrega) }}</td>
          <td>{{ t.aluno?.nome }}</td>
        </tr>
      </tbody>
    </table>
    <!-- O v-else TEM que vir colado logo depois da </table> -->
    <p v-else>Nenhum registro foi encontrado para os critérios fornecidos</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

// ---------- VARIÁVEIS ----------
const lista = ref<any[]>([])      // o que aparece na tabela
const alunos = ref<any[]>([])     // o que aparece no select
const erro = ref('')

const buscaRa = ref<any>('')      // campos da consulta
const buscaTitulo = ref('')

// Cria um "formulário vazio". Se o enunciado tiver outros campos, ponha aqui.  <-- TROQUE
function criarVazio() {
  return {
    titulo: '',
    dataHoraEntrega: '',
    descricao: '',
    nota: '',
    justificativa: '',
    aluno: { id: null as any }
  }
}
const novo = ref<any>(criarVazio())

// ---------- MOSTRAR ERRO ----------
function mostrarErro(e: any) {
  erro.value = e?.response?.data?.message || (e as Error).message
}

// ---------- ITEM 2: STATUS ----------
function calcularStatus(dataEntrega: string | null) {
  if (!dataEntrega) {
    return 'Pendente'
  }
  const entrega = new Date(dataEntrega)
  const agora = new Date()
  const umDia = 1000 * 60 * 60 * 24
  const dias = Math.floor(Math.abs(agora.getTime() - entrega.getTime()) / umDia)
  return `Entregue a ${dias} dias`
}

// ---------- LISTAR TUDO ----------
async function listarTodos() {
  try {
    erro.value = ''
    lista.value = (await axios.get('trabalho')).data          // <-- TROQUE a rota
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- ALUNOS PARA O SELECT ----------
async function listarAlunos() {
  try {
    alunos.value = (await axios.get('aluno')).data            // <-- TROQUE a rota do select
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- ITEM 3: CADASTRAR ----------
async function cadastrar() {
  erro.value = ''

  // Validação dos obrigatórios. Acrescente os que o enunciado/Entity pedir.   <-- TROQUE
  if (!novo.value.titulo || !novo.value.aluno.id) {
    erro.value = 'Preencha os campos obrigatórios'
    return
  }

  // Campo vazio ('') vira null antes de enviar (evita erro 400 no Java)
  const corpo: any = { ...novo.value }
  for (const campo in corpo) {
    if (corpo[campo] === '') {
      corpo[campo] = null
    }
  }

  try {
    await axios.post('trabalho', corpo)                       // <-- TROQUE a rota
    novo.value = criarVazio()      // limpa os inputs
    await listarTodos()            // atualiza a tabela
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- ITEM 4: PESQUISAR ----------
async function pesquisar() {
  erro.value = ''
  if (!buscaRa.value || !buscaTitulo.value) {
    erro.value = 'Informe o RA e o título para pesquisar'
    return
  }
  try {
    const resposta = await axios.get('trabalho/buscar', {     // <-- TROQUE a rota
      params: {
        ra: buscaRa.value,          // nome da esquerda = @RequestParam("ra") do Java
        titulo: buscaTitulo.value   // nome da esquerda = @RequestParam("titulo") do Java
      }
    })
    lista.value = resposta.data
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- AO ABRIR A PÁGINA ----------
onMounted(() => {
  listarTodos()
  listarAlunos()
})
</script>
```

**O que cada pedaço do enunciado vira:**

| Pediu o enunciado | Onde está no código |
|---|---|
| Tabela ao abrir a página | `onMounted` + `listarTodos` |
| Só id, título, status e nome do aluno | `<th>` e `<td>` da tabela |
| Status "Pendente" / "Entregue a X dias" | `calcularStatus` |
| Erro se faltar campo obrigatório | `if (!novo.value.titulo ...)` + `<p v-if="erro">` |
| Lista atualizada e inputs limpos | `novo.value = criarVazio()` e `listarTodos()` |
| Consulta na mesma lista | `pesquisar` troca o `lista.value` |
| Mensagem de nenhum registro | `<p v-else>` |

---

## 5. PEÇAS EXTRAS — casos que podem cair

### 5.1 Outros tipos de campo no cadastro

```html
<!-- Texto grande -->
<textarea v-model="novo.descricao"></textarea>

<!-- Sim/Não (Boolean no Java) -->
<label><input type="checkbox" v-model="novo.ativo" /> Ativo</label>

<!-- Só data -->
<input type="date" v-model="novo.data" />

<!-- Só hora -->
<input type="time" v-model="novo.hora" />

<!-- Senha / e-mail -->
<input type="password" v-model="novo.senha" />
<input type="email" v-model="novo.email" />

<!-- Opções fixas (ex.: Entrada/Saída). O texto do value é o que vai pro Java -->
<select v-model="novo.sentido">
  <option value="Entrada">Entrada</option>
  <option value="Saida">Saída</option>
</select>

<!-- Botões de escolha (radio) -->
<label><input type="radio" value="A" v-model="novo.tipo" /> A</label>
<label><input type="radio" value="B" v-model="novo.tipo" /> B</label>
```
Se criar campo novo: ponha também em `criarVazio()` (no checkbox use `false`).

### 5.2 Select de outra tabela (curso, projeto, produto...)
Troque `aluno` pelo nome do campo na Entity (ex.: `curso`):

```html
<select v-model="novo.curso.id">
  <option :value="null" disabled>Selecione...</option>
  <option v-for="c in cursos" :key="c.id" :value="c.id">{{ c.codigo }} - {{ c.nome }}</option>
</select>
```
```typescript
const cursos = ref<any[]>([])
// em criarVazio():  curso: { id: null as any }
// função: cursos.value = (await axios.get('curso')).data
// e chame no onMounted
```
Texto do option "RA acompanhado do nome" = `{{ a.ra }} - {{ a.nome }}`.

### 5.3 Colunas calculadas (cole no `<script>`, chame no `<td>`)

```typescript
// A) Dias entre DUAS datas do mesmo registro (null = "Em andamento")
function duracaoDias(inicio: string | null, fim: string | null) {
  if (!inicio || !fim) return 'Em andamento'
  const dias = Math.floor(Math.abs(new Date(fim).getTime() - new Date(inicio).getTime()) / 86400000)
  return `${dias} dias`
}

// B) Situação pela nota (null = "Não avaliado")
function situacao(nota: number | null) {
  if (nota === null || nota === undefined) return 'Não avaliado'
  return nota >= 6 ? 'Aprovado' : 'Reprovado'
}

// C) Booleano -> Sim/Não
function simNao(valor: boolean) {
  return valor ? 'Sim' : 'Não'
}

// D) Atrasado ou no prazo (compara data de entrega com prazo)
function prazo(entrega: string | null, limite: string) {
  if (!entrega) return 'Pendente'
  return new Date(entrega) > new Date(limite) ? 'Atrasado' : 'No prazo'
}

// E) Horas desde uma data
function horasDesde(data: string | null) {
  if (!data) return '-'
  return Math.floor(Math.abs(Date.now() - new Date(data).getTime()) / 3600000) + ' horas'
}

// F) Data em formato brasileiro (dd/mm/aaaa hh:mm)
function dataBR(data: string | null) {
  return data ? new Date(data).toLocaleString('pt-BR') : '-'
}

// G) Dinheiro em reais
function dinheiro(valor: number) {
  return valor.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' })
}
```
Uso: `<td>{{ situacao(t.nota) }}</td>` · `<td>{{ duracaoDias(t.dataInicio, t.dataFim) }}</td>`

Outras contas rápidas: **meses** = dias ÷ 30 → `Math.floor(dias / 30)` · **anos** = dias ÷ 365 · **dentro de objeto** = `t.aluno?.nome` (o `?.` evita erro se for null) · **texto em maiúsculo** = `t.titulo.toUpperCase()`.

### 5.4 Consultas diferentes

```typescript
// Um parâmetro só (ex.: @RequestParam("nome"))
const r = await axios.get('trabalho/buscar', { params: { nome: filtro.value } })

// Data (input type="date" já manda no formato certo)
const r = await axios.get('trabalho/buscar', { params: { data: dataBusca.value } })

// Intervalo (dois parâmetros)
const r = await axios.get('trabalho/buscar', { params: { inicio: ini.value, fim: fim.value } })

// @PathVariable  (ex.: GetMapping("/{id}") ou "/aluno/{ra}")
const r = await axios.get(`trabalho/aluno/${buscaRa.value}`)
```
⚠️ Se o back-end devolve **um objeto só** (não lista), embrulhe em lista para a tabela:
```typescript
lista.value = [r.data]
```
⚠️ Se devolver `null`/vazio: `lista.value = r.data ? [r.data] : []`

Filtro **só no Vue** (sem rota do back-end), se a prova pedir filtrar a lista que já está na tela:
```typescript
const filtrados = lista.value.filter((t: any) => t.aluno?.ra == buscaRa.value && t.titulo.includes(buscaTitulo.value))
lista.value = filtrados
```

### 5.5 Botão Excluir (se tiver `@DeleteMapping`)
Na tabela, uma coluna a mais: `<th>Ações</th>` e `<td>`:
```html
<td><button @click="excluir(t.id)">Excluir</button></td>
```
```typescript
async function excluir(id: number) {
  try {
    await axios.delete(`trabalho/${id}`)   // <-- TROQUE a rota
    await listarTodos()
  } catch (e) {
    mostrarErro(e)
  }
}
```

### 5.6 Botão Editar (se tiver `@PutMapping`)
```html
<td><button @click="editar(t)">Editar</button></td>
```
```typescript
function editar(t: any) {
  novo.value = JSON.parse(JSON.stringify(t))   // copia a linha para o formulário
}
```
E no `cadastrar`, troque a linha do `post` por:
```typescript
if (novo.value.id) {
  await axios.put(`trabalho/${novo.value.id}`, corpo)
} else {
  await axios.post('trabalho', corpo)
}
```

### 5.7 Duas listas na mesma página
Crie uma segunda `const lista2 = ref<any[]>([])`, uma função igual à `listarTodos` apontando para a outra rota, chame no `onMounted` e copie a `<table>` trocando `lista` por `lista2` e as colunas.

### 5.8 Mostrar mensagem "nenhum registro" só depois de pesquisar
```typescript
const pesquisou = ref(false)     // no pesquisar(), após o axios: pesquisou.value = true
```
```html
<p v-else-if="pesquisou">Nenhum registro foi encontrado para os critérios fornecidos</p>
```

### 5.9 Mensagem de sucesso
```typescript
const sucesso = ref('')          // após cadastrar: sucesso.value = 'Cadastrado com sucesso!'
```
```html
<p v-if="sucesso" style="color: green">{{ sucesso }}</p>
```

### 5.10 Confirmar antes de excluir
```typescript
if (!confirm('Tem certeza?')) return
```

---

## 6. Login com token (Parte 2 do guia) — só se a prova pedir

**`src/stores/usuario.ts`**
```typescript
import { ref } from 'vue'
import { defineStore } from 'pinia'
import axios from 'axios'

export const usuarioStore = defineStore('usuario', () => {
  const nomeUsuario = ref<string>()
  const token = ref<string>()
  const erro = ref<string>()

  async function login(nome: string, senha: string) {
    try {
      const response = await axios.post('login', {
        username: nome,     // <-- CONFIRA no back-end: "username/password" ou "nome/senha"
        password: senha
      })
      nomeUsuario.value = nome
      erro.value = ''
      token.value = response.data.token
    } catch (ex) {
      erro.value = (ex as Error).message
    }
  }
  return { nomeUsuario, token, erro, login }
},
{
  persist: true     // guarda no localStorage (precisa do plugin que já está no seu main.ts)
})
```

**`src/main.ts`** — adicione o interceptor (envia o token em toda requisição). Coloque depois de `axios.defaults.baseURL = ...` e adicione o import `import { usuarioStore } from '@/stores/usuario'`:
```typescript
axios.interceptors.request.use(config => {
  const store = usuarioStore()
  if (store.token) {
    config.headers.Authorization = store.token
  }
  return config
})
```

**`src/views/LoginView.vue`**
```vue
<template>
  <div>
    <h1>Login</h1>
    <p><label>Nome: </label><input type="text" v-model="nome" /></p>
    <p><label>Senha: </label><input type="password" v-model="senha" /></p>
    <button @click="store.login(nome, senha)">Login</button>
    <p v-if="store.erro">{{ store.erro }}</p>
    <p v-if="store.token">Logado como {{ store.nomeUsuario }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { usuarioStore } from '@/stores/usuario'

const nome = ref('')
const senha = ref('')
const store = usuarioStore()
</script>
```
Depois registre a rota (Lego A) e o link no menu.

---

## 7. Deu erro? Tabela de socorro

| Sintoma | Causa provável | Solução |
|---|---|---|
| Tabela vazia, console mostra **Network Error** | Backend desligado / `baseURL` errada | Rode o backend. Confira `axios.defaults.baseURL = 'http://localhost:8080/'` no `main.ts` |
| Erro de **CORS** no console | Falta `@CrossOrigin` no Controller | O Controller deve ter `@CrossOrigin` (o do exemplo tem) |
| **404** | Rota errada | Teste a URL no navegador. Rota **sem** barra inicial; confira `@RequestMapping` |
| **400 Bad Request** | JSON enviado errado (campo com nome errado, `''` em número/data, aluno sem id) | Confira nomes com o JSON do GET. Use o `corpo` do Lego B (troca `''` por `null`) |
| **500** | Erro no Java: campo obrigatório nulo, duplicado, banco | Veja o terminal do backend: a causa aparece lá |
| **405** | Método errado (`get` no lugar de `post`) | Confira `@GetMapping`/`@PostMapping` |
| Consulta sempre vazia | Nome do parâmetro diferente do `@RequestParam` | Igualar `ra`/`titulo` (ou os que o Java tiver) |
| `Cannot read properties of undefined` | Tentou ler campo de objeto vazio | Use `?.` (`t.aluno?.nome`) |
| Select não mostra opções | Lista de alunos vazia / rota do select errada | Abra `localhost:8080/aluno` no navegador |
| Página em branco | Erro de sintaxe no `.vue` (falta `</div>`, `}` ou `)`) | Veja o terminal do `npm run dev` e o Console (F12) |
| Link do menu não abre nada | `to` diferente do `path` da rota | Devem ser idênticos (`/trabalhos`) |
| Campos não limpam | Esqueceu `novo.value = criarVazio()` | Está no `cadastrar` do Lego B |
| Número aparece "X dias" negativo | Falta `Math.abs` | Já está no Lego B |
| Tela não atualiza depois de cadastrar | Faltou `await listarTodos()` | Está no `cadastrar` |
| `ref` não funciona no script | Esqueceu `.value` | Script: `lista.value`. Template: `lista` |

---

## 8. Mini-dicionário (para entender o que você está colando)

| Termo | O que é |
|---|---|
| `ref(...)` | Variável que, ao mudar, atualiza a tela sozinha |
| `v-model="x"` | Liga um campo da tela a uma variável (digitou → variável muda) |
| `v-for="t in lista"` | Repete a linha para cada item da lista |
| `:key="t.id"` | Identificador único de cada linha (sempre use o id) |
| `v-if` / `v-else` | Mostra uma coisa ou outra |
| `@click="funcao"` | Chama a função quando clica |
| `:value="x"` | Valor vindo de variável (com `:` é variável; sem `:` é texto fixo) |
| `async` / `await` | "Espere a resposta do back-end antes de continuar" |
| `try { } catch (e) { }` | Tenta executar; se der erro, cai no `catch` |
| `onMounted` | Roda quando a página abre |
| `axios.get/post` | Chamar o back-end (buscar / enviar) |
| `params: {...}` | Parâmetros da URL (`?ra=1&titulo=abc`) |
| `?.` | "Se existir, pegue; se for nulo, não quebre" |
| `.data` | O conteúdo da resposta do axios |

---

## 9. Checklist final antes de entregar

- [ ] Link do menu com o **texto exato** do enunciado
- [ ] Tabela abre preenchida ao entrar na página (item 2)
- [ ] Tabela mostra **somente** as colunas pedidas
- [ ] Status "Pendente" aparece para entrega nula
- [ ] Cadastro: select mostra `RA - Nome`
- [ ] Cadastro sem preencher obrigatório → **mensagem de erro na tela**
- [ ] Cadastro correto → tabela atualiza e campos ficam vazios
- [ ] Consulta com RA + título → resultado na **mesma** tabela
- [ ] Consulta sem resultado → **mensagem** no lugar da tabela
- [ ] Nenhum erro vermelho no Console (F12)
