<!--
  ╔══════════════════════════════════════════════════════════════════╗
  ║  CATÁLOGO DE FUNCIONALIDADES — página Vue 3 (script setup, TS)   ║
  ╚══════════════════════════════════════════════════════════════════╝

  COMO USAR (modo Lego):
  1. Copie este arquivo para src/views/ e registre a rota (veja COLA_PROVA_VUE.md, Lego A).
     Ele FUNCIONA inteiro como está (backend de Trabalho/Aluno rodando).
  2. Cada funcionalidade é um BLOCO NUMERADO. O mesmo número aparece em 3 lugares:
        [T] no <template>   (o que aparece na tela)
        [S] no <script>     (variáveis e funções)
     Para levar um bloco para a sua página da prova: copie [T] e [S] do mesmo número.
  3. Alguns blocos dependem de outros (está escrito "PRECISA DO BLOCO X").
  4. Troque nomes de campos/rotas pelos da prova. Procure por "TROQUE".

  ÍNDICE
   1  Mensagens (erro / sucesso / carregando)
   2  Mostrar / esconder o formulário (v-show)
   3  Cadastro + edição (todos os tipos de input)
   4  Select do banco (aluno)           PRECISA DO BLOCO 3
   5  Pesquisa no back-end: 2 parâmetros (ra + título)
   6  Pesquisa no back-end: por id (PathVariable)
   7  Filtros locais (texto, só pendentes, por aluno) e ordenação
   8  Resumo (total, pendentes, média) — computed
   9  Tabela com colunas calculadas e linha colorida   PRECISA DO BLOCO 7
  10  Mensagem "nenhum registro"
  11  Botões Editar / Excluir (com confirm)
  12  Detalhes de uma linha clicada
  13  Ao abrir a página (onMounted)
-->

<template>
  <div>
    <h1>Gerencie Trabalhos</h1>

    <!-- ===== [T] BLOCO 1: MENSAGENS ===== -->
    <p v-if="erro" style="color: red">{{ erro }}</p>
    <p v-if="sucesso" style="color: green">{{ sucesso }}</p>
    <p v-if="carregando">Carregando...</p>

    <!-- ===== [T] BLOCO 2: MOSTRAR / ESCONDER FORMULÁRIO ===== -->
    <button @click="mostrarForm = !mostrarForm">
      {{ mostrarForm ? 'Esconder formulário' : 'Novo trabalho' }}
    </button>

    <!-- ===== [T] BLOCO 3: CADASTRO + EDIÇÃO ===== -->
    <!-- Sem <form> e sem "required": a validação é feita na função salvar() e mostra o erro na tela -->
    <div v-show="mostrarForm">
      <h2>{{ novo.id ? 'Editar trabalho #' + novo.id : 'Novo trabalho' }}</h2>

      <!-- Texto (obrigatório: asterisco vermelho) -->
      <p>
        <label>Título <span style="color: red">*</span>: </label>
        <input type="text" v-model="novo.titulo" />
      </p>

      <!-- Texto grande -->
      <p>
        <label>Descrição: </label><br />
        <textarea v-model="novo.descricao" rows="3" cols="40"></textarea>
      </p>

      <p>
        <label>Justificativa: </label><br />
        <textarea v-model="novo.justificativa" rows="3" cols="40"></textarea>
      </p>

      <!-- Número decimal (v-model.number = já vira número) -->
      <p>
        <label>Nota: </label>
        <input type="number" v-model.number="novo.nota" min="0" max="10" step="0.1" />
      </p>

      <!-- Data e hora (LocalDateTime) -->
      <p>
        <label>Data/hora de entrega: </label>
        <input type="datetime-local" v-model="novo.dataHoraEntrega" />
      </p>

      <!-- Só data (LocalDate) -->
      <p>
        <label>Prazo: </label>
        <input type="date" v-model="novo.dataPrazo" />
      </p>

      <!-- Opções fixas (select) -->
      <p>
        <label>Tipo: </label>
        <select v-model="novo.tipo">
          <option value="">Selecione...</option>
          <option value="Individual">Individual</option>
          <option value="Grupo">Grupo</option>
        </select>
      </p>

      <!-- Radio -->
      <p>
        <label>Prioridade: </label>
        <label><input type="radio" value="Baixa" v-model="novo.prioridade" /> Baixa</label>
        <label><input type="radio" value="Alta" v-model="novo.prioridade" /> Alta</label>
      </p>

      <!-- Checkbox (Boolean) -->
      <p>
        <label><input type="checkbox" v-model="novo.ativo" /> Ativo</label>
      </p>

      <!-- ===== [T] BLOCO 4: SELECT DO BANCO (ALUNO) ===== -->
      <p>
        <label>Aluno <span style="color: red">*</span>: </label>
        <select v-model="novo.aluno.id">
          <option :value="null" disabled>Selecione...</option>
          <option v-for="a in alunos" :key="a.id" :value="a.id">{{ a.ra }} - {{ a.nome }}</option>
        </select>
      </p>

      <!-- Botão muda de texto se estiver editando -->
      <button @click="salvar">{{ novo.id ? 'Salvar alterações' : 'Cadastrar' }}</button>
      <button @click="cancelar">Limpar / Cancelar</button>
    </div>

    <!-- ===== [T] BLOCO 5: PESQUISA NO BACK-END (RA + TÍTULO) ===== -->
    <h2>Consultar</h2>
    <p>
      <label>RA: </label>
      <input type="number" v-model.number="buscaRa" />
      <label> Título: </label>
      <input type="text" v-model="buscaTitulo" />
      <button @click="pesquisarRaTitulo">Pesquisar</button>
      <button @click="buscarTrabalhos">Mostrar todos</button>
    </p>

    <!-- ===== [T] BLOCO 6: PESQUISA POR ID (PATHVARIABLE) ===== -->
    <p>
      <label>Buscar por id: </label>
      <input type="number" v-model.number="buscaId" />
      <button @click="pesquisarPorId">Buscar</button>
    </p>

    <!-- ===== [T] BLOCO 7: FILTROS LOCAIS E ORDENAÇÃO (não chamam o back-end) ===== -->
    <h2>Filtrar a lista na tela</h2>
    <p>
      <label>Título contém: </label>
      <input type="text" v-model="filtroTexto" />

      <label><input type="checkbox" v-model="apenasPendentes" /> Só pendentes</label>

      <label> Aluno: </label>
      <select v-model="filtroAluno">
        <option :value="null">Todos</option>
        <option v-for="a in alunos" :key="a.id" :value="a.id">{{ a.nome }}</option>
      </select>

      <label> Ordenar por: </label>
      <select v-model="ordenarPor">
        <option value="id">Id</option>
        <option value="titulo">Título</option>
        <option value="nota">Nota</option>
      </select>
      <select v-model="ordem">
        <option value="asc">Crescente</option>
        <option value="desc">Decrescente</option>
      </select>
    </p>

    <!-- ===== [T] BLOCO 8: RESUMO ===== -->
    <p>
      Total na tela: <strong>{{ listaVisivel.length }}</strong> |
      Pendentes: <strong>{{ qtdPendentes }}</strong> |
      Média das notas: <strong>{{ mediaNotas.toFixed(2) }}</strong>
    </p>

    <!-- ===== [T] BLOCO 9: TABELA ===== -->
    <table v-if="listaVisivel.length > 0" border="1" cellpadding="4">
      <thead>
        <tr>
          <th>Id</th>
          <th>Título</th>
          <th>Status</th>
          <th>Situação</th>
          <th>Entrega</th>
          <th>Prazo</th>
          <th>Ativo</th>
          <th>Aluno</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        <!-- :style = linha colorida conforme a função corLinha -->
        <tr v-for="t in listaVisivel" :key="t.id" :style="{ backgroundColor: corLinha(t) }">
          <td>{{ t.id }}</td>
          <td>{{ t.titulo }}</td>
          <td>{{ calcularStatus(t.dataHoraEntrega) }}</td>
          <td>{{ situacaoNota(t.nota) }}</td>
          <td>{{ dataBR(t.dataHoraEntrega) }}</td>
          <td>{{ dataBR(t.dataPrazo) }}</td>
          <td>{{ simNao(t.ativo) }}</td>
          <!-- ?. evita quebrar se o aluno vier nulo -->
          <td>{{ t.aluno?.nome }}</td>

          <!-- ===== [T] BLOCO 11: BOTÕES EDITAR / EXCLUIR ===== -->
          <td>
            <button @click="editar(t)">Editar</button>
            <button @click="excluir(t.id)">Excluir</button>
            <!-- ===== [T] BLOCO 12: BOTÃO DETALHES ===== -->
            <button @click="selecionado = t">Detalhes</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- ===== [T] BLOCO 10: NENHUM REGISTRO (v-else COLADO NA </table>) ===== -->
    <p v-else>Nenhum registro foi encontrado para os critérios fornecidos</p>

    <!-- ===== [T] BLOCO 12: DETALHES ===== -->
    <div v-if="selecionado" style="border: 1px solid gray; padding: 8px; margin-top: 8px">
      <h3>Detalhes de {{ selecionado.titulo }}</h3>
      <p>Descrição: {{ selecionado.descricao || '-' }}</p>
      <p>Justificativa: {{ selecionado.justificativa || '-' }}</p>
      <p>Tipo: {{ selecionado.tipo || '-' }} | Prioridade: {{ selecionado.prioridade || '-' }}</p>
      <p>Aluno: {{ selecionado.aluno?.ra }} - {{ selecionado.aluno?.nome }}</p>
      <button @click="selecionado = null">Fechar</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

// ===== [S] TIPOS (interfaces) — descreva aqui os campos da Entity =====  TROQUE
interface Aluno {
  id: number
  nome: string
  ra?: number
}

interface Trabalho {
  id?: number
  titulo: string
  descricao?: string | null
  justificativa?: string | null
  nota?: number | string | null
  dataHoraEntrega?: string | null
  dataPrazo?: string | null
  tipo?: string | null
  prioridade?: string | null
  ativo?: boolean
  aluno: { id: number | null; nome?: string; ra?: number }
}

// ===== [S] BLOCO 1: MENSAGENS =====
const erro = ref('')
const sucesso = ref('')
const carregando = ref(false)

// Mostra o erro do back-end (se tiver) ou a mensagem padrão do axios
function mostrarErro(e: any) {
  sucesso.value = ''
  erro.value = e?.response?.data?.message || (e as Error).message
}

// ===== [S] BLOCO 2: MOSTRAR / ESCONDER =====
const mostrarForm = ref(true)

// ===== [S] LISTA PRINCIPAL (usada pelos blocos 5, 6, 7, 9) =====
const trabalhos = ref<Trabalho[]>([])

async function buscarTrabalhos() {
  try {
    carregando.value = true
    erro.value = ''
    trabalhos.value = (await axios.get<Trabalho[]>('trabalho')).data // TROQUE a rota
  } catch (e) {
    mostrarErro(e)
  } finally {
    carregando.value = false
  }
}

// ===== [S] BLOCO 3: CADASTRO + EDIÇÃO =====
// Formulário vazio. Todo campo novo do formulário também entra aqui.   TROQUE
function criarVazio(): Trabalho {
  return {
    titulo: '',
    descricao: '',
    justificativa: '',
    nota: null,
    dataHoraEntrega: '',
    dataPrazo: '',
    tipo: '',
    prioridade: 'Baixa',
    ativo: true,
    aluno: { id: null },
  }
}
const novo = ref<Trabalho>(criarVazio())

async function salvar() {
  erro.value = ''
  sucesso.value = ''

  // Validação dos obrigatórios: acrescente os que o enunciado pedir        TROQUE
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
    if (novo.value.id) {
      await axios.put(`trabalho/${novo.value.id}`, corpo) // EDITAR  (precisa de @PutMapping)
      sucesso.value = 'Trabalho alterado!'
    } else {
      await axios.post('trabalho', corpo) // CADASTRAR
      sucesso.value = 'Trabalho cadastrado!'
    }
    novo.value = criarVazio() // limpa os inputs
    await buscarTrabalhos() // atualiza a tabela
  } catch (e) {
    mostrarErro(e)
  }
}

function cancelar() {
  novo.value = criarVazio()
  erro.value = ''
  sucesso.value = ''
}

// ===== [S] BLOCO 4: SELECT DO BANCO (ALUNO) =====
const alunos = ref<Aluno[]>([])

async function buscarAlunos() {
  try {
    alunos.value = (await axios.get<Aluno[]>('aluno')).data // TROQUE a rota do select
  } catch (e) {
    mostrarErro(e)
  }
}

// ===== [S] BLOCO 5: PESQUISA (RA + TÍTULO) =====
const buscaRa = ref<number | string>('')
const buscaTitulo = ref('')

async function pesquisarRaTitulo() {
  erro.value = ''
  sucesso.value = ''
  if (!buscaRa.value || !buscaTitulo.value) {
    erro.value = 'Informe o RA e o título para pesquisar'
    return
  }
  try {
    const resposta = await axios.get<Trabalho[]>('trabalho/buscar', {
      params: {
        ra: buscaRa.value, // nome da esquerda = @RequestParam("ra") do Java
        titulo: buscaTitulo.value, // nome da esquerda = @RequestParam("titulo") do Java
      },
    })
    trabalhos.value = resposta.data
  } catch (e) {
    mostrarErro(e)
  }
}

// ===== [S] BLOCO 6: PESQUISA POR ID (PATHVARIABLE) =====
const buscaId = ref<number | string>('')

async function pesquisarPorId() {
  erro.value = ''
  if (!buscaId.value) {
    erro.value = 'Informe o id'
    return
  }
  try {
    const resposta = await axios.get<Trabalho>(`trabalho/${buscaId.value}`)
    // O back-end devolve UM objeto; a tabela precisa de lista -> embrulha com [ ]
    trabalhos.value = resposta.data ? [resposta.data] : []
  } catch (e) {
    mostrarErro(e)
  }
}

// ===== [S] BLOCO 7: FILTROS LOCAIS E ORDENAÇÃO =====
const filtroTexto = ref('')
const apenasPendentes = ref(false)
const filtroAluno = ref<number | null>(null)
const ordenarPor = ref<'id' | 'titulo' | 'nota'>('id')
const ordem = ref<'asc' | 'desc'>('asc')

// "computed" = lista recalculada sozinha sempre que algum filtro muda
const listaVisivel = computed(() => {
  let resultado = [...trabalhos.value]

  if (filtroTexto.value) {
    const texto = filtroTexto.value.toLowerCase()
    resultado = resultado.filter((t) => t.titulo?.toLowerCase().includes(texto))
  }
  if (apenasPendentes.value) {
    resultado = resultado.filter((t) => !t.dataHoraEntrega)
  }
  if (filtroAluno.value) {
    resultado = resultado.filter((t) => t.aluno?.id === filtroAluno.value)
  }

  resultado.sort((a: any, b: any) => {
    const va = a[ordenarPor.value] ?? ''
    const vb = b[ordenarPor.value] ?? ''
    const r = va > vb ? 1 : va < vb ? -1 : 0
    return ordem.value === 'asc' ? r : -r
  })

  return resultado
})

// ===== [S] BLOCO 8: RESUMO =====
const qtdPendentes = computed(() => listaVisivel.value.filter((t) => !t.dataHoraEntrega).length)

const mediaNotas = computed(() => {
  const notas = listaVisivel.value
    .filter((t) => t.nota !== null && t.nota !== undefined && t.nota !== '')
    .map((t) => Number(t.nota))
  if (notas.length === 0) return 0
  return notas.reduce((soma, n) => soma + n, 0) / notas.length
})

// ===== [S] BLOCO 9: FUNÇÕES DAS COLUNAS CALCULADAS =====

// Status pela data de entrega (nula = Pendente, senão "Entregue a X dias")
function calcularStatus(dataEntrega?: string | null) {
  if (!dataEntrega) {
    return 'Pendente'
  }
  const umDia = 1000 * 60 * 60 * 24
  const dias = Math.floor(
    Math.abs(new Date().getTime() - new Date(dataEntrega).getTime()) / umDia,
  )
  return `Entregue a ${dias} dias`
}

// Situação pela nota (nula = Não avaliado, >= 6 Aprovado, senão Reprovado)
function situacaoNota(nota?: number | string | null) {
  if (nota === null || nota === undefined || nota === '') {
    return 'Não avaliado'
  }
  return Number(nota) >= 6 ? 'Aprovado' : 'Reprovado'
}

// Data no formato brasileiro (dd/mm/aaaa hh:mm:ss)
function dataBR(data?: string | null) {
  return data ? new Date(data).toLocaleString('pt-BR') : '-'
}

// Boolean -> Sim/Não
function simNao(valor?: boolean) {
  return valor ? 'Sim' : 'Não'
}

// Cor da linha: amarelo = pendente, vermelho claro = nota abaixo de 6
function corLinha(t: Trabalho) {
  if (!t.dataHoraEntrega) return '#fff3cd'
  if (t.nota !== null && t.nota !== undefined && t.nota !== '' && Number(t.nota) < 6) {
    return '#f8d7da'
  }
  return ''
}

// ===== [S] BLOCO 11: EDITAR / EXCLUIR =====
function editar(t: Trabalho) {
  const copia: Trabalho = JSON.parse(JSON.stringify(t)) // copia a linha para o formulário
  // datetime-local só aceita "aaaa-mm-ddThh:mm": corta o excesso
  if (copia.dataHoraEntrega) {
    copia.dataHoraEntrega = copia.dataHoraEntrega.substring(0, 16)
  }
  novo.value = copia
  mostrarForm.value = true
}

async function excluir(id?: number) {
  if (!confirm('Tem certeza que deseja excluir?')) {
    return
  }
  try {
    erro.value = ''
    await axios.delete(`trabalho/${id}`) // precisa de @DeleteMapping
    sucesso.value = 'Trabalho excluído!'
    await buscarTrabalhos()
  } catch (e) {
    mostrarErro(e)
  }
}

// ===== [S] BLOCO 12: DETALHES =====
const selecionado = ref<Trabalho | null>(null)

// ===== [S] BLOCO 13: AO ABRIR A PÁGINA =====
onMounted(() => {
  buscarTrabalhos()
  buscarAlunos()
})
</script>