<script setup>
import { ref } from 'vue';

const firstName = ref('');
const lastName = ref('');
const motherLastName = ref('');
const birthDate = ref('');
const rfc = ref('');

const generateRFC = () => {
  const rfcValue = `${lastName.value.slice(0, 2)}${motherLastName.value.slice(0, 1)}${firstName.value.slice(0, 1)}${birthDate.value.replaceAll('-', '').slice(2, 8)}`;
  rfc.value = rfcValue.toUpperCase();
};
</script>

<template>
  <Fluid>
    <div class="flex flex-col md:flex-row gap-8">
      <div class="md:w-1/2">
        <div class="card flex flex-col gap-4">
          <div class="font-semibold text-xl">Generar el recibo de una persona</div>
          <div class="flex flex-col gap-2">
            <label for="firstName">Nombre(s):</label>
            <InputText id="firstName" v-model="firstName" type="text" />
          </div>
          <div class="flex flex-col gap-2">
            <label for="lastName">Apellido Paterno:</label>
            <InputText id="lastName" v-model="lastName" type="text" />
          </div>
          <div class="flex flex-col gap-2">
            <label for="motherLastName">Apellido Materno:</label>
            <InputText id="motherLastName" v-model="motherLastName" type="text" />
          </div>
          <div class="flex flex-col gap-2">
            <label for="birthDate">Fecha de nacimiento:</label>
            <InputText id="birthDate" v-model="birthDate" type="date" />
          </div>
          <div class="flex flex-col gap-2">
            <label for="rfc">RFC:</label>
            <InputText id="rfc" v-model="rfc" type="text" readonly />
          </div>
          <Button label="Generar RFC" @click="generateRFC" :fluid="false" />
        </div>
      </div>
    </div>
  </Fluid>
</template>
