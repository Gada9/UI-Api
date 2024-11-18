<script>
import axios from 'axios'; // Asegúrate de importar axios

export default {
  data() {
    return {
      customerData: null, // Para almacenar los datos del cliente que buscamos
      customerId: '', // Para almacenar el ID ingresado por el usuario
      customers1: [], // Aquí debes cargar los datos de la API
      filters1: {
        global: { value: null } // Aseguramos que 'global' esté inicializado
      },
      loading: false, // Para manejar el estado de carga de la búsqueda
      loading1: true, // Para manejar el estado de carga de la tabla
    };
  },
  methods: {
    // Método para buscar un teléfono específico
    searchCustomer() {
      this.loading = true;
      axios
        .get(`https://api.restful-api.dev/objects/${this.customerId}`) // URL de la API para buscar un solo cliente por ID
        .then((response) => {
          if (response.data) {
            this.customerData = {
              id: response.data.id,
              name: response.data.name || '',
              color: response.data.data?.color || '',
              capacity: response.data.data?.capacity || '',
            };
          } else {
            this.customerData = null; // Si no se encuentra el cliente
          }
          this.loading = false;
        })
        .catch((error) => {
          console.error('Error loading data:', error);
          this.loading = false;
        });
    },

    // Método para cargar todos los teléfonos
    loadData() {
      this.loading1 = true;
      axios
        .get('https://api.restful-api.dev/objects') // URL para obtener todos los objetos
        .then((response) => {
          if (Array.isArray(response.data)) {
            this.customers1 = response.data.map((item) => ({
              id: item.id,
              name: item.name || 'N/A',
              color: item.data?.color || 'N/A',
              capacity: item.data?.capacity || 'N/A',
            }));
          } else {
            this.customers1 = [response.data]; // Si es un solo objeto
          }
          this.loading1 = false;
        })
        .catch((error) => {
          console.error('Error loading data:', error);
          this.loading1 = false;
        });
    },

    // Limpiar el filtro global
    clearFilter() {
      this.filters1 = { global: { value: null } }; // Limpiar el filtro global
    },
  },
  mounted() {
    this.loadData(); // Llamar al método cuando el componente esté montado
  },
};
</script>

<template>
  <div class="flex justify-center p-4">
    <div class="bg-white p-6 rounded-lg shadow-md w-full max-w-lg">
      <!-- Búsqueda de Teléfono -->
      <div class="font-semibold text-xl mb-4">Buscar Telefono</div>
      <div class="flex flex-col gap-2 mb-4">
        <label for="customerId">ID del Telefono:</label>
        <InputText id="customerId" v-model="customerId" type="text" />
        <Button label="Buscar" @click="searchCustomer" />
      </div>

      <!-- Detalles del Teléfono -->
      <div v-if="customerData" class="mt-6">
        <div class="font-semibold text-xl mb-4">Detalles del Telefono</div>

        <div class="flex flex-col gap-4">
          <div class="flex items-center">
            <label for="name" class="mr-4 w-32">Nombre:</label>
            <InputText id="name" v-model="customerData.name" type="text" class="flex-grow" />
          </div>

          <div class="flex items-center">
            <label for="color" class="mr-4 w-32">Color:</label>
            <InputText id="color" v-model="customerData.color" type="text" class="flex-grow" />
          </div>

          <div class="flex items-center">
            <label for="capacity" class="mr-4 w-32">Capacidad:</label>
            <InputText id="capacity" v-model="customerData.capacity" type="text" class="flex-grow" />
          </div>
        </div>
      </div>

      <div v-else-if="customerId && !loading">
        <p>No se encontraron datos para este ID.</p>
      </div>

      <div v-if="loading">
        <p>Cargando datos...</p>
      </div>
    </div>
  </div>

  <!-- Tabla de Todos los Teléfonos -->
  <div class="card mt-6">
    <div class="font-semibold text-xl mb-4">Datos De Los Telefonos</div>
    <DataTable
      :value="customers1"
      :paginator="true"
      :rows="10"
      dataKey="id"
      :rowHover="true"
      v-model:filters="filters1"
      filterDisplay="menu"
      :loading="loading1"
      :globalFilterFields="['name', 'color', 'capacity']"
      showGridlines
    >
      <template #header>
        <div class="flex justify-between">
          <InputText v-model="filters1.global.value" placeholder="Keyword Search" /> <!-- Filtrar globalmente -->
        </div>
      </template>

      <template #empty>
        No se encontraron teléfonos.
      </template>

      <template #loading>
        Cargando datos de los teléfonos. Por favor espere.
      </template>

      <!-- Nueva columna para el ID -->
      <Column field="id" header="CNS" style="min-width: 6rem" sortable>
        <template #body="{ data }">
          {{ data.id }}
        </template>
      </Column>

      <Column field="name" header="Nombre" style="min-width: 12rem" sortable>
        <template #body="{ data }">
          {{ data.name }}
        </template>
      </Column>

      <Column field="color" header="Color" style="min-width: 12rem" sortable>
        <template #body="{ data }">
          {{ data.color }}
        </template>
      </Column>

      <Column field="capacity" header="Capacidad" style="min-width: 12rem" sortable>
        <template #body="{ data }">
          {{ data.capacity }}
        </template>
      </Column>
    </DataTable>
  </div>
</template>

<style scoped>
/* Estilos generales */
.card {
  background-color: white;
  border-radius: 15px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
  padding: 1rem;
}

.bg-white {
  background-color: #ffffff;
}

.max-w-lg {
  max-width: 800px;
}

.shadow-md {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.rounded-lg {
  border-radius: 10px;
}

/* Separar los bloques de búsqueda y tabla */
.mt-6 {
  margin-top: 1.5rem;
}
</style>


