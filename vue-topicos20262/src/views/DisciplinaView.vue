<template>
  <div>
    <h1>Gerencie Disciplinas</h1>

    <!-- Mensagem de erro (aparece em vermelho) -->
    <p v-if="erro" style="color: red">{{ erro }}</p>

    <!-- ================= ITEM 3: CADASTRO ================= -->
  <h2>Nova Disciplina</h2>

    <p>
      <label>Código: </label>
      <input type="text" v-model="novo.codigo" />
    </p>

    <p>
      <label>Nome: </label>
      <input type="text" v-model="novo.nome" />
    </p>

    <p>
      <label>Carga Horária: </label>
      <input type="number" v-model="novo.cargaHoraria" />
    </p>

    <p>
      <label>Curso: </label>
      <select v-model="novo.curso.id">
        <option :value="null" disabled>Selecione...</option>
        <option v-for="c in cursos" :key="c.id" :value="c.id">
          {{ c.sigla }} - {{ c.nome }}
        </option>
      </select>
    </p>

    <button @click="cadastrar">Cadastrar</button>

    <!-- ================= ITEM 4: CONSULTA ================= -->
  <h2>Consultar</h2>

    <p>
      <label>Id da disciplina: </label>
      <input type="number" v-model="buscaId" />
    </p>

    <button @click="pesquisar">Pesquisar</button>
    <button @click="listarTodos">Mostrar todos</button>

    <!-- ================= ITEM 2: TABELA ================= -->
    <h2>Disciplinas</h2>

    <table v-if="lista.length > 0">
      <thead>
        <tr>
          <th>id</th>
          <th>codigo</th>
          <th>nome</th>
          <th>cargaHoraria</th>
          <th>Curso</th>

        </tr>
      </thead>
      <tbody>
        <tr v-for="t in lista" :key="t.id">
          <td>{{ t.id }}</td>
          <td>{{ t.codigo }}</td>
          <td>{{ t.nome }}</td>
          <td>{{ calcularDuracao(t.cargaHoraria) }}</td>
          <td>{{ t.curso?.sigla }}</td>
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
const cursos = ref<any[]>([])     // o que aparece no select
const erro = ref('')

const buscaId = ref<any>('')      // campos da consulta
const buscaTitulo = ref('')

// Cria um "formulário vazio". Se o enunciado tiver outros campos, ponha aqui.  <-- TROQUE
function criarVazio() {
  return {
    codigo: '',
    nome: '',
    cargaHoraria: '',
    curso: { id: null as any }
  }
}
const novo = ref<any>(criarVazio())

// ---------- MOSTRAR ERRO ----------
function mostrarErro(e: any) {
  erro.value = e?.response?.data?.message || (e as Error).message
}

// ---------- ITEM 2: STATUS ----------
function calcularDuracao(cargaHoraria: number | null) {
  if (cargaHoraria === null || cargaHoraria === undefined) {
    return 'Não definida'
  }
  if (cargaHoraria >= 80) {
    return `Longa (${cargaHoraria} horas)`
  }
  return `Curta (${cargaHoraria} horas)`
}

// ---------- LISTAR TUDO ----------
async function listarTodos() {
  try {
    erro.value = ''
    lista.value = (await axios.get('disciplina')).data          // <-- TROQUE a rota
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- ALUNOS PARA O SELECT ----------
async function listarCursos() {
  try {
    cursos.value = (await axios.get('curso')).data
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- ITEM 3: CADASTRAR ----------
async function cadastrar() {
  erro.value = ''

  // 1. Validação dos obrigatórios (Item 3 da prova)
  if (!novo.value.codigo || !novo.value.nome || !novo.value.curso.id) {
    erro.value = 'Preencha os campos obrigatórios'
    return
  }

  // 2. Transforma campos vazios ('') em null antes de mandar pro Java
  const corpo: any = { ...novo.value }
  for (const campo in corpo) {
    if (corpo[campo] === '') {
      corpo[campo] = null
    }
  }

  try {
    await axios.post('disciplina', corpo)
    novo.value = criarVazio() // Limpa os inputs mantendo curso: { id: null }
    await listarTodos()       // Atualiza a tabela do Item 2
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- ITEM 4: PESQUISAR ----------
async function pesquisar() {
  erro.value = ''
  if (!buscaId.value) {
    erro.value = 'Informe o id para pesquisar'
    return
  }
  try {
    const todas = (await axios.get('disciplina')).data
    lista.value = todas.filter((d: any) => d.id === Number(buscaId.value))
  } catch (e) {
    mostrarErro(e)
  }
}

// ---------- AO ABRIR A PÁGINA ----------
onMounted(() => {
  listarTodos()
  listarCursos()
})
</script>