<script setup>
import { computed, onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';

const tutores = ref([]);
const pets = ref([]);
const pesquisa = ref('');
const erro = ref('');
const carregando = ref(false);

async function carregarTutores() {
  carregando.value = true;
  erro.value = '';

  try {
    const [respostaTutores, respostaPets] = await Promise.all([
      fetch(`${API_URL}/tutores`),
      fetch(`${API_URL}/pets`),
    ]);

    if (!respostaTutores.ok) {
      throw new Error('Não foi possível carregar os tutores.');
    }

    tutores.value = await respostaTutores.json();

    if (respostaPets.ok) {
      pets.value = await respostaPets.json();
    } else {
      pets.value = [];
    }
  } catch (error) {
    console.error('Erro ao carregar dados:', error);
    erro.value =
      'Não foi possível carregar os tutores. Verifique se o JSON Server está funcionando na porta 3000.';
  } finally {
    carregando.value = false;
  }
}

const tutoresFiltrados = computed(() => {
  const termo = pesquisa.value.trim().toLowerCase();

  if (!termo) return tutores.value;

  return tutores.value.filter((tutor) =>
    [tutor.nome, tutor.cpfcnpj, tutor.email].some((valor) =>
      String(valor ?? '').toLowerCase().includes(termo)
    )
  );
});

function obterPets(tutor) {
  const petsDoTutor = pets.value.filter((pet) => {
    const idVinculado =
      pet.tutorId ??
      pet.tutor_id ??
      pet.idTutor ??
      pet.tutor?.id;

    return String(idVinculado) === String(tutor.id);
  });

  if (petsDoTutor.length) return petsDoTutor;

  if (Array.isArray(tutor.pets)) return tutor.pets;
  if (Array.isArray(tutor.animais)) return tutor.animais;

  return [];
}

function quantidadePets(tutor) {
  return obterPets(tutor).length;
}

function nomesPets(tutor) {
  const lista = obterPets(tutor);

  if (!lista.length) return 'Nenhum pet vinculado';

  return lista
    .map((pet) => typeof pet === 'string' ? pet : pet.nome)
    .filter(Boolean)
    .join(' / ');
}

function mascararDocumento(documento) {
  const numeros = String(documento ?? '').replace(/\D/g, '');

  if (numeros.length === 11) return '***.***.***-**';
  if (numeros.length === 14) return '**.***.***/****-**';

  return documento || '—';
}

async function excluirTutor(id) {
  if (!window.confirm('Tem certeza que deseja excluir este tutor?')) {
    return;
  }

  try {
    const resposta = await fetch(`${API_URL}/tutores/${id}`, {
      method: 'DELETE',
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível excluir o tutor.');
    }

    tutores.value = tutores.value.filter(
      (tutor) => String(tutor.id) !== String(id)
    );
  } catch (error) {
    console.error('Erro ao excluir tutor:', error);
    window.alert('Não foi possível excluir o tutor.');
  }
}

onMounted(carregarTutores);
</script>


<template>
  <main class="tutores-container">
    <header class="pagina-header">
      <div>
        <h1>Listagem de Tutores</h1>
        <p>Consulte os tutores cadastrados e seus pets.</p>
      </div>

      <RouterLink
        class="btn-cadastrar"
        to="/tutores/novo"
      >
        <span class="icone-mais">+</span>
        Cadastrar Tutor
      </RouterLink>
    </header>

    <div class="barra-pesquisa">
      <span class="icone-pesquisa" aria-hidden="true">
        <svg
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
        >
          <circle cx="10.8" cy="10.8" r="6.8" />
          <path d="m16 16 5 5" />
        </svg>
      </span>

      <input
        v-model="pesquisa"
        type="search"
        placeholder="Pesquisar tutor por nome, CPF/CNPJ ou e-mail"
        aria-label="Pesquisar tutor por nome, CPF/CNPJ ou e-mail"
      />
    </div>

    <p v-if="carregando" class="mensagem">
      Carregando tutores...
    </p>

    <div v-else-if="erro" class="mensagem erro">
      {{ erro }}

      <button class="btn-tentar" @click="carregarTutores">
        Tentar novamente
      </button>
    </div>

    <template v-else>
      <div class="tabela-container">
        <table class="tabela-tutores">
          <thead>
            <tr>
              <th class="coluna-id">ID</th>
              <th>Nome Completo</th>
              <th>CPF / CNPJ</th>
              <th>Telefone / Celular / WhatsApp</th>
              <th>E-mail</th>
              <th class="coluna-quantidade">Pets (Quantidade)</th>
              <th>Nomes dos Pets</th>
              <th class="coluna-acoes">Ações</th>
            </tr>
          </thead>

          <tbody>
            <tr
              v-for="tutor in tutoresFiltrados"
              :key="tutor.id"
            >
              <td class="coluna-id">{{ tutor.id }}</td>

              <td class="nome-tutor">
                {{ tutor.nome }}
              </td>

              <td class="documento">
                {{ mascararDocumento(tutor.cpfcnpj) }}
              </td>

              <td>{{ tutor.telefone || '—' }}</td>

              <td>{{ tutor.email || '—' }}</td>

              <td class="coluna-quantidade">
                {{ quantidadePets(tutor) }}
              </td>

              <td class="nomes-pets">
                {{ nomesPets(tutor) }}
              </td>

              <td class="coluna-acoes">
                <div class="botoes-acoes">
                  <RouterLink
                      class="btn-visualizar"
                      :to="`/tutores/${tutor.id}`"
                  >
                  <svg
                      viewBox="0 0 24 24"
                      fill="none"
                      stroke="currentColor"
                      stroke-width="2"
                      aria-hidden="true"
                  >
                  <path
                  d="M2 12s3.6-7 10-7 10 7 10 7-3.6 7-10 7-10-7-10-7Z"
                  />
                  <circle cx="12" cy="12" r="3" />
                  </svg>
                  Visualizar
                  </RouterLink>
                  <button
                    class="btn-remover"
                    type="button"
                    title="Remover tutor"
                    @click="excluirTutor(tutor.id)"
                  >
                    <svg
                      viewBox="0 0 24 24"
                      fill="none"
                      stroke="currentColor"
                      stroke-width="2"
                      aria-hidden="true"
                    >
                      <path d="M3 6h18" />
                      <path d="M8 6V4h8v2" />
                      <path d="m19 6-1 14H6L5 6" />
                      <path d="M10 10v6M14 10v6" />
                    </svg>
                    Remover
                  </button>
                </div>
              </td>
            </tr>

            <tr v-if="tutoresFiltrados.length === 0">
              <td colspan="8" class="sem-resultados">
                {{
                  pesquisa
                    ? 'Nenhum tutor encontrado para essa pesquisa.'
                    : 'Nenhum tutor cadastrado.'
                }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <footer class="tabela-footer">
        Exibindo {{ tutoresFiltrados.length }}
        {{ tutoresFiltrados.length === 1 ? 'tutor' : 'tutores' }}
      </footer>
    </template>
  </main>
</template>

<style scoped>
.tutores-container {
  min-height: 100%;
  width: 100%;
  padding: 26px 24px;
  box-sizing: border-box;
  background-color: #fff;
  color: #172033;
  font-family: Arial, Helvetica, sans-serif;
}

.pagina-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  margin-bottom: 32px;
}

.pagina-header h1 {
  margin: 0 0 5px;
  color: #151d2b;
  font-size: 38px;
  line-height: 1.2;
  font-weight: 700;
  letter-spacing: -0.8px;
}

.pagina-header p {
  margin: 0;
  color: #667085;
  font-size: 19px;
  line-height: 1.5;
}

.btn-cadastrar {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  flex-shrink: 0;
  gap: 10px;
  min-height: 52px;
  padding: 10px 20px;
  border: 1px solid #0d6efd;
  border-radius: 6px;
  background-color: #0d6efd;
  color: #fff;
  font-size: 16px;
  text-decoration: none;
  transition: background-color 0.2s;
}

.btn-cadastrar:hover {
  background-color: #0b5ed7;
}

.icone-mais {
  font-size: 29px;
  font-weight: 300;
  line-height: 1;
}

.barra-pesquisa {
  display: flex;
  align-items: center;
  gap: 12px;
  width: min(100%, 960px);
  height: 54px;
  margin-bottom: 20px;
  padding: 0 16px;
  box-sizing: border-box;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  background: #fff;
}

.icone-pesquisa {
  display: flex;
  width: 24px;
  height: 24px;
  flex-shrink: 0;
  color: #586577;
}

.icone-pesquisa svg {
  width: 100%;
  height: 100%;
}

.barra-pesquisa input {
  width: 100%;
  height: 100%;
  padding: 0;
  border: none;
  outline: none;
  background: transparent;
  color: #172033;
  font-family: inherit;
  font-size: 16px;
}

.barra-pesquisa:focus-within {
  border-color: #0d6efd;
  box-shadow: 0 0 0 3px rgb(13 110 253 / 10%);
}

.barra-pesquisa input::placeholder {
  color: #8792a5;
}

.tabela-container {
  width: 100%;
  overflow-x: auto;
  border: 1px solid #e1e6ec;
  border-radius: 5px;
}

.tabela-tutores {
  width: 100%;
  min-width: 1100px;
  border-collapse: collapse;
  font-size: 15px;
}

.tabela-tutores th,
.tabela-tutores td {
  padding: 15px 14px;
  border-right: 1px solid #e1e6ec;
  border-bottom: 1px solid #e1e6ec;
  text-align: left;
  vertical-align: middle;
}

.tabela-tutores th:last-child,
.tabela-tutores td:last-child {
  border-right: none;
}

.tabela-tutores tbody tr:last-child td {
  border-bottom: none;
}

.tabela-tutores thead {
  background-color: #f1f3f5;
}

.tabela-tutores th {
  color: #172033;
  font-weight: 700;
  line-height: 1.5;
  white-space: normal;
}

.tabela-tutores tbody tr:nth-child(even) {
  background-color: #f7f7f7;
}

.tabela-tutores tbody tr:hover {
  background-color: #f0f6ff;
}

.coluna-id {
  width: 48px;
  text-align: center !important;
}

.nome-tutor {
  min-width: 150px;
}

.documento {
  white-space: nowrap;
}

.coluna-quantidade {
  min-width: 125px;
  text-align: center !important;
}

.nomes-pets {
  min-width: 160px;
}

.coluna-acoes {
  min-width: 245px;
}

.botoes-acoes {
  display: flex;
  align-items: center;
  gap: 8px;
  white-space: nowrap;
}

.btn-visualizar,
.btn-remover {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  gap: 7px;
  min-height: 40px;
  padding: 8px 11px;
  border: 1px solid;
  border-radius: 5px;
  background: #fff;
  font-family: inherit;
  font-size: 14px;
  text-decoration: none;
  cursor: pointer;
  transition: background-color 0.2s;
}

.btn-visualizar {
  border-color: #0d6efd;
  color: #0d6efd;
}

.btn-visualizar:hover {
  background-color: #eaf2ff;
}

.btn-remover {
  border-color: #ff4545;
  color: #f5222d;
}

.btn-remover:hover {
  background-color: #fff0f0;
}

.btn-visualizar svg,
.btn-remover svg {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

.tabela-footer {
  margin-top: 18px;
  padding-top: 16px;
  border-top: 1px solid #e1e6ec;
  color: #667085;
  font-size: 16px;
}

.mensagem {
  padding: 20px 0;
  color: #667085;
}

.erro {
  color: #c62828;
}

.btn-tentar {
  margin-left: 10px;
  padding: 7px 12px;
  border: 1px solid #0d6efd;
  border-radius: 4px;
  background: #fff;
  color: #0d6efd;
  cursor: pointer;
}

.sem-resultados {
  padding: 28px !important;
  color: #667085;
  text-align: center !important;
}

@media (max-width: 768px) {
  .tutores-container {
    padding: 20px 14px;
  }

  .pagina-header {
    flex-direction: column;
    gap: 16px;
    margin-bottom: 24px;
  }

  .pagina-header h1 {
    font-size: 29px;
  }

  .pagina-header p {
    font-size: 16px;
  }

  .btn-cadastrar {
    min-height: 44px;
  }

  .barra-pesquisa {
    height: 48px;
  }

  .tabela-tutores {
    font-size: 14px;
  }
}
</style>
