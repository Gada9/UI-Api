<script>
import { useToast } from 'primevue/usetoast';
import { computed, ref } from 'vue';

export default {
    data() {
        const toast = useToast();
        const visibleUpdate = ref(false);
        const visibleDelete = ref(false);

        const nomp = ref('');
        const cantidad = ref('');
        const precioUnitario = ref('');

        const tablaCompras = ref([
            { cns: 1, nomProducto: "Impresora LaserJet Color", cantidad: 2, precioUnitario: 5200.00, precioParcial: 10400.00 },
            { cns: 2, nomProducto: "Monitor LED 31 plg.", cantidad: 3, precioUnitario: 1700.00, precioParcial: 5100.00 }
        ]);

        const productoItem = ref({
            cns: null,
            nomProducto: null,
            cantidad: null,
            precioUnitario: null,
            precioParcial: null
        });

        const productoSeleccionado = ref(null); // Producto para eliminar
        const productoEditando = ref(null);     // Producto para editar

        // Cálculos computados para el subtotal, IVA y total
        const subtotal = computed(() => tablaCompras.value.reduce((sum, p) => sum + p.precioParcial, 0));
        const iva = computed(() => subtotal.value * 0.16);
        const total = computed(() => subtotal.value + iva.value);

        // Funciones para editar y eliminar productos
        function editarProducto(row) {
            // Guardamos los datos del producto para edición.
            productoEditando.value = { ...row };
            nomp.value = row.nomProducto;
            cantidad.value = row.cantidad;
            precioUnitario.value = row.precioUnitario;

            // Abrimos el diálogo.
            visibleUpdate.value = true;
         }

        function updateProducto() {
            if (productoEditando.value) {
                const index = tablaCompras.value.findIndex(item => item.cns === productoEditando.value.cns);
                if (index !== -1) {
                    tablaCompras.value[index].nomProducto = nomp.value;
                    tablaCompras.value[index].cantidad = parseFloat(cantidad.value);
                    tablaCompras.value[index].precioUnitario = parseFloat(precioUnitario.value);
                    tablaCompras.value[index].precioParcial = tablaCompras.value[index].cantidad * tablaCompras.value[index].precioUnitario;

                    // Muestra el mensaje solo después de la actualización
                    toast.add({ severity: 'success', summary: 'Actualización Exitosa', detail: 'Producto actualizado correctamente', life: 3000 });

                    // Cerramos el diálogo y limpiamos el estado.
                    visibleUpdate.value = false;
                    productoEditando.value = null;
                }
            }
        }


        async function eliminarProducto(row) {
            productoSeleccionado.value = row; 
            visibleDelete.value = true;        
        }

        async function deleteProducto() {
            const index = tablaCompras.value.findIndex(item => item.cns === productoSeleccionado.value.cns);
            if (index !== -1) {
                tablaCompras.value.splice(index, 1);
                toast.add({ severity: 'success', summary: 'Eliminación Exitosa', detail: 'Producto eliminado correctamente', life: 3000 });
                visibleDelete.value = false;
                productoSeleccionado.value = null; 
            }
        }

        async function registrarCompra() {
            if (productoItem.value.nomProducto?.trim() !== '' && productoItem.value.cantidad && productoItem.value.precioUnitario) {
                tablaCompras.value.push({
                    cns: tablaCompras.value.length + 1,
                    nomProducto: productoItem.value.nomProducto,
                    cantidad: parseFloat(productoItem.value.cantidad),
                    precioUnitario: parseFloat(productoItem.value.precioUnitario),
                    precioParcial: parseFloat(productoItem.value.cantidad) * parseFloat(productoItem.value.precioUnitario)
                });
                toast.add({ severity: 'success', summary: 'Confirmación', detail: 'Nueva compra registrada', life: 3000 });

                productoItem.value.nomProducto = '';
                productoItem.value.cantidad = '';
                productoItem.value.precioUnitario = '';
            }
        }

        return {
            registrarCompra,
            editarProducto,
            updateProducto,
            deleteProducto,
            eliminarProducto,
            toast,
            visibleDelete,
            visibleUpdate,
            nomp,
            cantidad,
            precioUnitario,
            tablaCompras,
            productoItem,
            productoSeleccionado,
            productoEditando,
            subtotal,
            iva,
            total
        };
    },
    methods: {
        formatoMoneda(value) {
            return value ? value.toLocaleString('en-US', { style: 'currency', currency: 'USD' }) : '';
        }
    }
};
</script>


<template>
	<div class="p-grid">
		<Toast/>
		<div class="p-col-12">
			<div class="card">
			<Panel header="PUNTO DE VENTA (POS)" style="height: 100%">
				<Toolbar class="p-mb-4">
				<template v-slot:start>
					<InputText type="text" size="40" placeholder="Nombre del producto..." v-model="productoItem.nomProducto"/>
					<InputText class="ml-8" type="text" placeholder="Cant" v-model="productoItem.cantidad"/>
					<InputText class="ml-8" type="text" placeholder="$ Precio U." v-model="productoItem.precioUnitario"/>										
				</template>
				<template v-slot:end>
          <!--  le quite el class="p-button-success p-mr-2" al boton a regristrar y le puse completa la estructura de Button para que jale 
            el color que quiera el usuario -->
					<Button label="Registrar" icon="pi pi-plus"  @click="registrarCompra()"></Button>
				</template>
				</Toolbar>
				<br>

			<DataTable :value="tablaCompras" v-model:selection="productoItem" class="p-datatable-gridlines" dataKey="cns" :rows="5" 
				paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
				:rowsPerPageOptions="[5,10,25]"
				currentPageReportTemplate="Visualizando {last} de {totalRecords} productos"
				style="margin-bottom: 20px" :paginator="true" responsiveLayout="scroll">				
			
				<Column field="cns" header="Cns" :sortable="true" style="width:50px"></Column>
				<Column field="nomProducto" header="Nombre del Producto" style="width:370px"></Column>				
				<Column field="precioUnitario" header="Precio U." dataType="numeric" style="width:80px">
					<template #body="slotProps">
                            {{ formatoMoneda(slotProps.data.precioUnitario) }}
                    </template>
				</Column>
				<Column field="cantidad" header="Cant." style="width:60px;text-align:center"></Column>
				<Column field="precioParcial" header="Precio P." style="width:80px">
					<template #body="slotProps">
                            			{{ formatoMoneda(slotProps.data.precioParcial) }}
                    			</template></Column>
				<Column style="width:140px">
					<template #header>
						Acciones
					</template>
					<template #body="slotProps">
            <!--  le quite el class="p-button-success p-mr-2" al boton a regristrar y le puse completa la estructura de Button para que jale 
            el color que quiera el usuario  (<Button icon="pi pi-pencil" type="button" class="p-button-success p-mr-2 p-mb-1" @click="editarProducto(slotProps.data)"></Button>)-->
                        <Button icon="pi pi-pencil" type="button" @click="editarProducto(slotProps.data)"></Button>
                        <Dialog v-model:visible="visibleUpdate" modal header="Actualizar datos de un producto" :style="{ width: '45vw' }" draggable="false" style="position: fixed;">
                            <p>
                                <div class="flex gap-2">
                                    <div class="flex-auto">
                                        <label for="nomp"><strong>Nombre del producto: </strong></label>
                                        <InputText class="ml-2" id="nomp" v-model="nomp"/>  
                                    </div>                         
                                    <div class="flex-auto">
                                        <label for="precioUnitario"><strong>Precio Unitario: </strong></label>
                                        <InputText class="ml-2" id="precioUnitario" v-model="precioUnitario"/>
                                    </div>
                                    <div class="flex-auto">
                                        <label for="cantidad"><strong>Cantidad: </strong></label>
                                        <InputText class="ml-2" id="cantidad" v-model="cantidad"/>
                                    </div>
                                </div>
                                <br>
                            </p>
                            <template #footer>
                                <Button label="Actualizar" icon="pi pi-replay" @click="updateProducto()" />
                            </template>
                        </Dialog>
                        <Button icon="pi pi-trash" type="button" class="p-button-danger p-mb-1" @click="eliminarProducto(slotProps.data)"></Button>
                        <Dialog v-model:visible="visibleDelete" modal header="Estas seguro que quieres borrar este producto?" :style="{ width: '30vw' }">
                            <p>
                                <br>
                            </p>
                            <template #footer>
                                <Button label="Eliminar" severity="warning" icon="pi pi-check" @click="deleteProducto(slotProps.data)" autofocus />
                            </template>
                        </Dialog>
					</template>
				</Column>
			</DataTable>

			<br>
				<Toolbar class="p-mb-4">
				<template v-slot:start>
								
				</template>
        <template v-slot:end>
          <div style="display: inline-block; margin-right: 22rem;">
              <strong>Subtotal:</strong> {{ formatoMoneda(subtotal) }}
          </div>
          <div style="display: inline-block; margin-right: 22rem;">
              <strong>IVA (16%):</strong> {{ formatoMoneda(iva) }}
          </div>
          <div style="display: inline-block;">
              <strong>Total:</strong> {{ formatoMoneda(total) }}
          </div>
        </template>
				</Toolbar>
			</Panel>
			</div>	
		</div>
	</div>
</template>