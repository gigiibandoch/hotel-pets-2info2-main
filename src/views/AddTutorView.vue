<script>
import { ref } from 'vue';
import { RouterLink, useRouter } from 'vue-router';

const router = useRouter();
const API_URL = 'http://localhost:3000';

const novoTutor = ref({
  nome: '',
  telefone: '',
});

async function salvarTutor() {
  await fetch(`${API_URL}/tutores`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(novoTutor.value),
  });

  router.push('/tutores');
}
</script>
<template>
<div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Cadastrar Tutores</h1>
      <p class="text-body-secondary mb-0">Cadastro de Tutores no sistema.</p>
    </header>

    <RouterLink
      class="btn btn-outline-primary"
      :to="{ name: 'tutores' }"
    >
      Lista de Tutores
    </RouterLink>

    <form
      class="row g-3"
      @submit.prevent="salvarTutor"
    >
      <div class="col-md-6">
        <label
          for="nome"
          class="form-label"
        >
          Nome do Tutor:
        </label>
        <input
          type="text"
          class="form-control"
          id="nome"
          v-model="novoTutor.nome"
          required
        />
      </div>

      <div class="col-md-6">
        <label
          class="form-label"
          for="telefone"
          >Telefone</label
        >
        <input
          type="tel"
          class="form-control"
          id="telefone"
          v-model="novoTutor.telefone"
          placeholder="(xx) xxxxx-xxxx"
          required
        />
      </div>

      <div class="col-12 d-flex gap-2">
        <button
          class="btn btn-success"
          type="submit"
        >
          Salvar Tutor
        </button>
      </div>
    </form>
  </div>
</template>
