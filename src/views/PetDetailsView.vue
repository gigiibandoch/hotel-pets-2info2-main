<script setup>
import { onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute(); // chamando meu router
// chamando a minha API
const API_URL = 'http://localhost:3000';

const pet = ref(null); // lista de pets vazia
const tutor = ref(null); // lista de tutor vazia;

async function carregarPet(){
const idPet = route.params.id; // pegando o id do pet que está na rota
const respostaPet = await fetch(`${API_URL}/pets/${idPet}`);
console.log(respostaPet);
pet.value = await respostaPet.json();
console.log('pet',pet.value);


const respostaTutores = await fetch(`${API_URL}/tutor/${pet.value.tutorId}`);
tutor.value = await respostaTutores.json();
console.log('tutor',tutor.value);
}

onMounted(carregarPet);
</script>

<template>

<section v-if="pet == null">
  <h1>Carregando...</h1>
</section>


<section v-else>
<h1>Nome do Pet: {{ pet.nome }}</h1>
<p>Tutor: {{ tutor?.nome }}</p>
<h5>Espécie: {{ pet.especie }}</h5>
</section>

</template>
