<script setup>
import { computed, onMounted, ref } from 'vue';
import { RouterLink, useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

const API_URL = 'http://localhost:3000';

const tutor = ref(null);
const pets = ref([]);
const carregando = ref(true);
const erro = ref('');
const salvandoPet = ref(false);

const novoPet = ref({
  nome: '',
  especie: '',
});

const idTutor = computed(() => route.params.id);

const petsDoTutor = computed(() => {
  return pets.value.filter((pet) => {
    const idVinculado =
      pet.tutorId ??
      pet.tutor_id ??
      pet.idTutor ??
      pet.tutor?.id;

    return String(idVinculado) === String(idTutor.value);
  });
});

function formatarData(data) {
  if (!data) return '—';

  // Evita deslocamento de dia ao formatar uma data YYYY-MM-DD.
  const partes = String(data).split('-');

  if (
    partes.length === 3 &&
    partes[0].length === 4
  ) {
    return `${partes[2]}/${partes[1]}/${partes[0]}`;
  }

  return data;
}

function mascararDocumento(documento) {
  const numeros = String(documento ?? '').replace(/\D/g, '');

  if (numeros.length === 11) {
    return '***.***.***-**';
  }

  if (numeros.length === 14) {
    return '**.***.***/****-**';
  }

  return documento || '—';
}

async function carregarTutor() {
  carregando.value = true;
  erro.value = '';

  try {
    const resposta = await fetch(
      `${API_URL}/tutores/${idTutor.value}`
    );

    if (!resposta.ok) {
      throw new Error('Não foi possível encontrar o tutor.');
    }

    tutor.value = await resposta.json();

    try {
      const respostaPets = await fetch(`${API_URL}/pets`);

      if (respostaPets.ok) {
        pets.value = await respostaPets.json();
      }
    } catch (error) {
      console.error('Erro ao carregar pets:', error);
    }
  } catch (error) {
    console.error('Erro ao carregar tutor:', error);
    erro.value =
      'Não foi possível carregar os dados do tutor. Verifique a API.';
  } finally {
    carregando.value = false;
  }
}

async function cadastrarPet() {
  if (!novoPet.value.nome.trim() || !novoPet.value.especie) {
    alert('Preencha o nome e a espécie do pet.');
    return;
  }

  salvandoPet.value = true;

  try {
    const pet = {
      nome: novoPet.value.nome.trim(),
      especie: novoPet.value.especie,
      tutorId: tutor.value.id,
    };

    const resposta = await fetch(`${API_URL}/pets`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(pet),
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível cadastrar o pet.');
    }

    const petCadastrado = await resposta.json();

    pets.value.push(petCadastrado);

    novoPet.value = {
      nome: '',
      especie: '',
    };

    alert('Pet cadastrado com sucesso!');
  } catch (error) {
    console.error('Erro ao cadastrar pet:', error);
    alert(
      'Não foi possível cadastrar o pet. Verifique se a rota /pets existe na API.'
    );
  } finally {
    salvandoPet.value = false;
  }
}

onMounted(carregarTutor);
</script>

<template>
  <main class="detalhes-container">
    <section v-if="carregando" class="mensagem">
      Carregando dados do tutor...
    </section>

    <section v-else-if="erro" class="mensagem erro">
      {{ erro }}

      <button class="btn-tentar" @click="carregarTutor">
        Tentar novamente
      </button>
    </section>

    <template v-else-if="tutor">
      <header class="pagina-header">
        <div>
          <h1>Detalhes do Tutor</h1>
          <p>
            Visualize os dados do tutor e cadastre pets vinculados.
          </p>
        </div>

        <RouterLink
          class="btn-voltar"
          to="/tutores"
        >
          <span aria-hidden="true">←</span>
          Voltar para Tutores
        </RouterLink>
      </header>

      <section class="painel">
        <h2 class="painel-titulo">Informações do Tutor</h2>

        <div class="painel-conteudo">
          <section class="subpainel">
            <h3 class="subpainel-titulo">Dados do Tutor</h3>

            <div class="dados-grid">
              <div class="dado">
                <strong>ID</strong>
                <span>{{ tutor.id }}</span>
              </div>

              <div class="dado">
                <strong>Nome Completo</strong>
                <span>{{ tutor.nome || '—' }}</span>
              </div>

              <div class="dado">
                <strong>CPF / CNPJ</strong>
                <span>{{ mascararDocumento(tutor.cpfcnpj) }}</span>
              </div>

              <div class="dado">
                <strong>Telefone / Celular / WhatsApp</strong>
                <span>{{ tutor.telefone || '—' }}</span>
              </div>

              <div class="dado">
                <strong>E-mail</strong>
                <span>{{ tutor.email || '—' }}</span>
              </div>

              <div class="dado">
                <strong>Data de Nascimento</strong>
                <span>{{ formatarData(tutor.dataNascimento) }}</span>
              </div>
            </div>
          </section>

          <section class="subpainel">
            <h3 class="subpainel-titulo">Endereço</h3>

            <div class="dados-grid">
              <div class="dado">
                <strong>Rua</strong>
                <span>{{ tutor.rua || '—' }}</span>
              </div>

              <div class="dado">
                <strong>Número</strong>
                <span>{{ tutor.numero || '—' }}</span>
              </div>

              <div class="dado">
                <strong>Bairro</strong>
                <span>{{ tutor.bairro || '—' }}</span>
              </div>

              <div class="dado">
                <strong>Cidade</strong>
                <span>{{ tutor.cidade || '—' }}</span>
              </div>

              <div class="dado">
                <strong>Estado</strong>
                <span>{{ tutor.estado || '—' }}</span>
              </div>

              <div class="dado">
                <strong>CEP</strong>
                <span>{{ tutor.cep || '—' }}</span>
              </div>
            </div>
          </section>
        </div>
      </section>

      <section class="painel painel-pets">
        <h2 class="painel-titulo">Pets do Tutor</h2>

        <div class="pets-layout">
          <section class="subpainel lista-pets">
            <h3 class="subpainel-titulo titulo-pets">
              <span>Pets vinculados</span>
              <span class="contador">
                {{ petsDoTutor.length }}
                {{ petsDoTutor.length === 1 ? 'pet' : 'pets' }}
              </span>
            </h3>

            <div class="tabela-container">
              <table class="tabela-pets">
                <thead>
                  <tr>
                    <th>ID</th>
                    <th>Nome</th>
                    <th>Espécie</th>
                    <th>Ações</th>
                  </tr>
                </thead>

                <tbody>
                  <tr
                    v-for="pet in petsDoTutor"
                    :key="pet.id"
                  >
                    <td>{{ pet.id }}</td>
                    <td>{{ pet.nome }}</td>
                    <td>{{ pet.especie || '—' }}</td>
                    <td>
                      <RouterLink
                        class="btn-visualizar-pet"
                        :to="`/pets/${pet.id}`"
                      >
                        Visualizar
                      </RouterLink>
                    </td>
                  </tr>

                  <tr v-if="petsDoTutor.length === 0">
                    <td colspan="4" class="sem-pets">
                      Nenhum pet vinculado a este tutor.
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </section>

          <section class="subpainel formulario-pet">
            <h3 class="subpainel-titulo">
              Cadastrar novo pet
            </h3>

            <p class="descricao-pet">
              O pet será vinculado automaticamente a
              <strong>{{ tutor.nome }}</strong>.
            </p>

            <form @submit.prevent="cadastrarPet">
              <div class="campo">
                <label for="nomePet">Nome do pet</label>
                <input
                  id="nomePet"
                  v-model.trim="novoPet.nome"
                  type="text"
                  placeholder="Digite o nome do pet"
                  required
                />
              </div>

              <div class="campo">
                <label for="especiePet">Espécie</label>
                <select
                  id="especiePet"
                  v-model="novoPet.especie"
                  required
                >
                  <option value="" disabled>Selecione</option>
                  <option value="Cachorro">Cachorro</option>
                  <option value="Gato">Gato</option>
                  <option value="Ave">Ave</option>
                  <option value="Coelho">Coelho</option>
                  <option value="Roedor">Roedor</option>
                  <option value="Réptil">Réptil</option>
                  <option value="Outro">Outro</option>
                </select>
              </div>

              <button
                class="btn-cadastrar-pet"
                type="submit"
                :disabled="salvandoPet"
              >
                {{
                  salvandoPet
                    ? 'Cadastrando...'
                    : 'Cadastrar pet'
                }}
              </button>
            </form>
          </section>
        </div>
      </section>
    </template>
  </main>
</template>

<style scoped>
.detalhes-container {
  width: 100%;
  min-height: 100%;
  padding: 16px 22px;
  box-sizing: border-box;
  background: #fff;
  color: #172033;
  font-family: Arial, Helvetica, sans-serif;
}

.pagina-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 12px;
}

.pagina-header h1 {
  margin: 0 0 3px;
  color: #202733;
  font-size: 36px;
  line-height: 1.2;
  font-weight: 700;
  letter-spacing: -0.7px;
}

.pagina-header p {
  margin: 0;
  color: #667085;
  font-size: 15px;
}

.btn-voltar {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  flex-shrink: 0;
  padding: 9px 14px;
  border: 1px solid #0d6efd;
  border-radius: 5px;
  background: #fff;
  color: #0d6efd;
  font-size: 14px;
  text-decoration: none;
}

.btn-voltar:hover,
.btn-visualizar-pet:hover {
  background: #edf5ff;
}

.btn-voltar span {
  font-size: 20px;
  line-height: 1;
}

.painel {
  margin-bottom: 12px;
  overflow: hidden;
  border: 1px solid #dfe5ec;
  border-radius: 5px;
}

.painel-titulo,
.subpainel-titulo {
  margin: 0;
  padding: 10px 12px;
  border-bottom: 1px solid #dfe5ec;
  background: #f1f3f5;
  color: #202733;
  font-size: 20px;
  font-weight: 700;
}

.painel-conteudo {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 10px 12px;
}

.subpainel {
  min-width: 0;
  overflow: hidden;
  border: 1px solid #dfe5ec;
  border-radius: 5px;
}

.subpainel-titulo {
  padding: 7px 12px;
  font-size: 16px;
}

.dados-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px 20px;
  padding: 9px 12px;
}

.dado {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 3px;
  font-size: 14px;
  overflow-wrap: anywhere;
}

.dado strong {
  font-weight: 700;
}

.dado span {
  color: #303846;
}

.painel-pets > .painel-titulo {
  padding: 9px 12px;
}

.pets-layout {
  display: grid;
  grid-template-columns: minmax(0, 1.15fr) minmax(0, 1fr);
  gap: 12px;
  padding: 10px 12px;
}

.titulo-pets {
  display: flex;
  align-items: center;
  gap: 10px;
}

.contador {
  padding: 4px 10px;
  border-radius: 14px;
  background: #0d6efd;
  color: #fff;
  font-size: 12px;
  font-weight: 600;
  white-space: nowrap;
}

.tabela-container {
  width: 100%;
  overflow-x: auto;
}

.tabela-pets {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}

.tabela-pets th,
.tabela-pets td {
  padding: 7px 10px;
  border-bottom: 1px solid #e5e9ef;
  text-align: left;
  vertical-align: middle;
}

.tabela-pets th {
  background: #f1f3f5;
  font-weight: 700;
}

.tabela-pets tbody tr:nth-child(even) {
  background: #f7f7f7;
}

.tabela-pets tbody tr:last-child td {
  border-bottom: none;
}

.btn-visualizar-pet {
  display: inline-block;
  padding: 5px 12px;
  border: 1px solid #0d6efd;
  border-radius: 4px;
  background: #fff;
  color: #0d6efd;
  text-decoration: none;
  white-space: nowrap;
}

.sem-pets {
  padding: 18px 10px !important;
  color: #667085;
  text-align: center !important;
}

.formulario-pet {
  padding-bottom: 8px;
}

.formulario-pet > .subpainel-titulo {
  border-bottom: none;
  background: #fff;
  padding-bottom: 2px;
}

.descricao-pet {
  margin: 0 12px 8px;
  color: #667085;
  font-size: 13px;
  line-height: 1.4;
}

.formulario-pet form {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 0 12px;
}

.campo {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.campo label {
  font-size: 13px;
  font-weight: 600;
}

.campo input,
.campo select {
  width: 100%;
  height: 34px;
  padding: 6px 10px;
  box-sizing: border-box;
  border: 1px solid #cbd5e1;
  border-radius: 4px;
  background: #fff;
  color: #172033;
  font-family: inherit;
  font-size: 13px;
}

.campo input:focus,
.campo select:focus {
  border-color: #0d6efd;
  outline: 2px solid rgb(13 110 253 / 12%);
}

.campo input::placeholder {
  color: #929baa;
}

.btn-cadastrar-pet {
  width: 100%;
  min-height: 34px;
  padding: 7px 12px;
  border: 1px solid #198754;
  border-radius: 4px;
  background: #198754;
  color: #fff;
  font-family: inherit;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}

.btn-cadastrar-pet:hover {
  background: #157347;
}

.btn-cadastrar-pet:disabled {
  opacity: 0.65;
  cursor: wait;
}

.mensagem {
  padding: 20px 0;
  color: #667085;
}

.erro {
  color: #c62828;
}

.btn-tentar {
  margin-left: 8px;
  padding: 6px 10px;
  border: 1px solid #0d6efd;
  border-radius: 4px;
  background: #fff;
  color: #0d6efd;
  cursor: pointer;
}

@media (max-width: 850px) {
  .pets-layout {
    grid-template-columns: 1fr;
  }

  .pagina-header h1 {
    font-size: 29px;
  }
}

@media (max-width: 600px) {
  .detalhes-container {
    padding: 16px 12px;
  }

  .pagina-header {
    flex-direction: column;
  }

  .dados-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 14px;
  }

  .painel-titulo {
    font-size: 18px;
  }
}

@media (max-width: 380px) {
  .dados-grid {
    grid-template-columns: 1fr;
  }
}
</style>
