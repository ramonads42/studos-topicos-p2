<template>
  <div>
    <h1>Gerenciamento de Trabalhos 2</h1>
    <p v-if="erro" style="color: red">{{ erro }}</p>

    <!-- AQUI VOCÊ VAI MONTAR O FORMULÁRIO DE CADASTRO (ITEM 3) -->

    <!-- AQUI VOCÊ VAI MONTAR O FORMULÁRIO DE BUSCA (ITEM 4) -->

    <table>
      <thead>
        <tr>
          <th>Id</th>
          <th>Título</th>
          <th>Status</th>
          <th>Aluno</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="trabalho in trabalhos" :key="trabalho.id">
          <td>{{ trabalho.id }}</td>
          <td>{{ trabalho.titulo }}</td>
          <td>{{ formatarStatus(trabalho.dataHoraEntrega) }}</td>
          <td>{{ trabalho.aluno?.nome }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

const trabalhos = ref<any[]>([])
const erro = ref('')

async function buscarTrabalhos() {
  try {
    const resposta = await axios.get('trabalho')
    trabalhos.value = resposta.data
  } catch (ex) {
    erro.value = (ex as Error).message
  }
}

function formatarStatus(dataHoraEntrega: string | null) {
  if (!dataHoraEntrega) {
    return 'Pendente'
  }

  const dataEntrega = new Date(dataHoraEntrega)
  const dataAtual = new Date()

  const diferencaMs = dataAtual.getTime() - dataEntrega.getTime()
  const dias = Math.floor(diferencaMs / (1000 * 60 * 60 * 24))

  return `Entregue a ${dias} dias`
}

onMounted(() => {
  buscarTrabalhos()
})
</script>