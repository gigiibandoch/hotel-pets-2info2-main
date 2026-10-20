<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const API_URL = 'http://localhost:3000';
const salvando = ref(false);

const novoTutor = ref({
  nome: '',
  cpfcnpj: '',
  telefone: '',
  email: '',
  dataNascimento: '',
  rua: '',
  numero: '',
  bairro: '',
  cidade: '',
  estado: '',
  cep: '',
});

const estados = [
  { sigla: 'AC', nome: 'Acre' },
  { sigla: 'AL', nome: 'Alagoas' },
  { sigla: 'AP', nome: 'Amapá' },
  { sigla: 'AM', nome: 'Amazonas' },
  { sigla: 'BA', nome: 'Bahia' },
  { sigla: 'CE', nome: 'Ceará' },
  { sigla: 'DF', nome: 'Distrito Federal' },
  { sigla: 'ES', nome: 'Espírito Santo' },
  { sigla: 'GO', nome: 'Goiás' },
  { sigla: 'MA', nome: 'Maranhão' },
  { sigla: 'MT', nome: 'Mato Grosso' },
  { sigla: 'MS', nome: 'Mato Grosso do Sul' },
  { sigla: 'MG', nome: 'Minas Gerais' },
  { sigla: 'PA', nome: 'Pará' },
  { sigla: 'PB', nome: 'Paraíba' },
  { sigla: 'PR', nome: 'Paraná' },
  { sigla: 'PE', nome: 'Pernambuco' },
  { sigla: 'PI', nome: 'Piauí' },
  { sigla: 'RJ', nome: 'Rio de Janeiro' },
  { sigla: 'RN', nome: 'Rio Grande do Norte' },
  { sigla: 'RS', nome: 'Rio Grande do Sul' },
  { sigla: 'RO', nome: 'Rondônia' },
  { sigla: 'RR', nome: 'Roraima' },
  { sigla: 'SC', nome: 'Santa Catarina' },
  { sigla: 'SP', nome: 'São Paulo' },
  { sigla: 'SE', nome: 'Sergipe' },
  { sigla: 'TO', nome: 'Tocantins' },
];

async function salvarTutor() {
  if (salvando.value) return;

  salvando.value = true;

  try {
    const dadosTutor = {
      nome: novoTutor.value.nome.trim(),
      cpfcnpj: novoTutor.value.cpfcnpj.trim(),
      telefone: novoTutor.value.telefone.trim(),
      email: novoTutor.value.email.trim(),
      dataNascimento: novoTutor.value.dataNascimento,
      rua: novoTutor.value.rua.trim(),
      numero: novoTutor.value.numero.trim(),
      bairro: novoTutor.value.bairro.trim(),
      cidade: novoTutor.value.cidade.trim(),
      estado: novoTutor.value.estado,
      cep: novoTutor.value.cep.trim(),
    };

    const resposta = await fetch(`${API_URL}/tutores`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(dadosTutor),
    });

    if (!resposta.ok) {
      const mensagem = await resposta.text();
      throw new Error(`Erro ${resposta.status}: ${mensagem}`);
    }

    const tutorSalvo = await resposta.json();

    if (tutorSalvo.id === undefined || tutorSalvo.id === null) {
      throw new Error('A API salvou o tutor sem retornar um ID.');
    }

    alert('Tutor cadastrado com sucesso!');
    await router.push('/tutores');
  } catch (erro) {
    console.error('Erro ao cadastrar tutor:', erro);

    alert(
      'Não foi possível cadastrar o tutor. Confira se o JSON Server está funcionando na porta 3000.'
    );
  } finally {
    salvando.value = false;
  }
}

function cancelarCadastro() {
  router.push('/tutores');
}
</script>


<template>
  <main class="cadastro-container">
    <header class="pagina-header">
      <h1>Cadastrar Tutor</h1>
      <p>Preencha os dados para cadastrar um novo tutor.</p>
    </header>

    <form @submit.prevent="salvarTutor">
      <section class="form-section">
        <h2>Dados do Tutor</h2>

        <div class="form-grid dados-grid">
          <div class="campo campo-id">
            <label for="idTutor">ID</label>
            <input
              id="idTutor"
              type="text"
              value="Gerado automaticamente"
              disabled
            />
          </div>

          <div class="campo campo-nome">
            <label for="nome">Nome Completo</label>
            <input
              id="nome"
              v-model.trim="novoTutor.nome"
              type="text"
              placeholder="Digite o nome completo"
              autocomplete="name"
              required
            />
          </div>

          <div class="campo">
            <label for="cpfcnpj">CPF / CNPJ</label>
            <input
              id="cpfcnpj"
              v-model.trim="novoTutor.cpfcnpj"
              type="text"
              placeholder="Digite o CPF ou CNPJ"
              required
            />
          </div>

          <div class="campo">
            <label for="telefone">
              Telefone / Celular / WhatsApp
            </label>
            <input
              id="telefone"
              v-model.trim="novoTutor.telefone"
              type="tel"
              placeholder="(00) 00000-0000"
              autocomplete="tel"
              required
            />
          </div>

          <div class="campo">
            <label for="email">E-mail</label>
            <input
              id="email"
              v-model.trim="novoTutor.email"
              type="email"
              placeholder="nome@example.com"
              autocomplete="email"
              required
            />
          </div>

          <div class="campo">
            <label for="dataNascimento">Data de Nascimento</label>
            <input
              id="dataNascimento"
              v-model="novoTutor.dataNascimento"
              type="date"
              autocomplete="bday"
              required
            />
          </div>
        </div>
      </section>

      <section class="form-section endereco-section">
        <h2>Endereço</h2>

        <div class="form-grid endereco-grid">
          <div class="campo campo-rua">
            <label for="rua">Rua</label>
            <input
              id="rua"
              v-model.trim="novoTutor.rua"
              type="text"
              placeholder="Digite o nome da rua"
              autocomplete="address-line1"
              required
            />
          </div>

          <div class="campo campo-numero">
            <label for="numero">Número</label>
            <input
              id="numero"
              v-model.trim="novoTutor.numero"
              type="text"
              placeholder="Número"
              required
            />
          </div>

          <div class="campo">
            <label for="bairro">Bairro</label>
            <input
              id="bairro"
              v-model.trim="novoTutor.bairro"
              type="text"
              placeholder="Digite o bairro"
              required
            />
          </div>

          <div class="campo">
            <label for="cidade">Cidade</label>
            <input
              id="cidade"
              v-model.trim="novoTutor.cidade"
              type="text"
              placeholder="Digite a cidade"
              autocomplete="address-level2"
              required
            />
          </div>

          <div class="campo">
            <label for="estado">Estado</label>
            <select
              id="estado"
              v-model="novoTutor.estado"
              autocomplete="address-level1"
              required
            >
              <option value="" disabled>Selecione</option>
              <option
                v-for="estado in estados"
                :key="estado.sigla"
                :value="estado.sigla"
              >
                {{ estado.nome }}
              </option>
            </select>
          </div>

          <div class="campo">
            <label for="cep">CEP</label>
            <input
              id="cep"
              v-model.trim="novoTutor.cep"
              type="text"
              placeholder="00000-000"
              autocomplete="postal-code"
              required
            />
          </div>
        </div>
      </section>

      <div class="form-actions">
        <button
          class="btn-cancelar"
          type="button"
          @click="cancelarCadastro"
        >
          Cancelar
        </button>

        <button class="btn-cadastrar" type="submit">
          Cadastrar Tutor
        </button>
      </div>
    </form>
  </main>
</template>

<style scoped>
.cadastro-container {
  width: 100%;
  min-height: 100%;
  padding: 22px 24px 10px;
  color: #172033;
  background-color: #fff;
  box-sizing: border-box;
  font-family: Arial, Helvetica, sans-serif;
}

.pagina-header {
  margin-bottom: 20px;
}

.pagina-header h1 {
  margin: 0 0 4px;
  color: #151d2b;
  font-size: 36px;
  line-height: 1.2;
  font-weight: 700;
  letter-spacing: -0.8px;
}

.pagina-header p {
  margin: 0;
  color: #667085;
  font-size: 16px;
  line-height: 1.5;
}

.form-section {
  margin-bottom: 24px;
}

.form-section h2 {
  margin: 0 0 14px;
  padding: 0 0 8px;
  border-bottom: 1px solid #dce2e9;
  color: #172033;
  font-size: 20px;
  font-weight: 700;
  line-height: 1.4;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  column-gap: 22px;
  row-gap: 16px;
}

.campo {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 6px;
}

.campo label {
  color: #172033;
  font-size: 14px;
  font-weight: 600;
  line-height: 1.4;
}

.campo input,
.campo select {
  width: 100%;
  height: 39px;
  padding: 8px 14px;
  border: 1px solid #cbd5e1;
  border-radius: 4px;
  outline: none;
  box-sizing: border-box;
  background-color: #fff;
  color: #172033;
  font-family: inherit;
  font-size: 14px;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.campo input::placeholder {
  color: #8792a5;
  opacity: 1;
}

.campo input:focus,
.campo select:focus {
  border-color: #0d6efd;
  box-shadow: 0 0 0 3px rgb(13 110 253 / 12%);
}

.campo input:disabled {
  background-color: #edf0f4;
  color: #7b8798;
  cursor: not-allowed;
}

.dados-grid {
  grid-template-areas:
    "id nome"
    "cpf telefone"
    "email nascimento";
}

.campo-id {
  grid-area: id;
}

.campo-nome {
  grid-area: nome;
}

.endereco-grid {
  grid-template-areas:
    "rua numero"
    "bairro cidade"
    "estado cep";
}

.campo-rua {
  grid-area: rua;
}

.campo-numero {
  grid-area: numero;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 9px;
  margin-top: -16px;
  padding-bottom: 0;
}

.form-actions button {
  min-height: 38px;
  padding: 8px 17px;
  border-radius: 4px;
  font-family: inherit;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s, border-color 0.2s;
}

.btn-cancelar {
  border: 1px solid #cbd5e1;
  background-color: #fff;
  color: #172033;
}

.btn-cancelar:hover {
  background-color: #f1f5f9;
  border-color: #aab5c4;
}

.btn-cadastrar {
  border: 1px solid #0d6efd;
  background-color: #0d6efd;
  color: #fff;
}

.btn-cadastrar:hover {
  background-color: #0b5ed7;
  border-color: #0a58ca;
}

@media (max-width: 700px) {
  .cadastro-container {
    padding: 20px 16px;
  }

  .pagina-header h1 {
    font-size: 28px;
  }

  .form-grid,
  .dados-grid,
  .endereco-grid {
    grid-template-columns: minmax(0, 1fr);
    grid-template-areas: none;
  }

  .campo-id,
  .campo-nome,
  .campo-rua,
  .campo-numero {
    grid-area: auto;
  }

  .form-section {
    margin-bottom: 24px;
  }

  .form-actions {
    margin-top: 0;
    flex-wrap: wrap;
  }
}

@media (max-width: 380px) {
  .form-actions {
    align-items: stretch;
    flex-direction: column-reverse;
  }

  .form-actions button {
    width: 100%;
  }
}
</style>
