<template>
  <main class="about-page">
    <Card>
      <template #header>
        <h1 style="margin-top: 15px; margin-left: 15px">Farmacias</h1>
      </template>

      <template #content>
        <div style="margin-top: 10px">
          <DataTable
            :value="categorias"
            responsiveLayout="scroll"
            :loading="loading"
            :globalFilterFields="['name', 'address']"
            v-model:filters="filters"
            filterDisplay="menu"
            style="text-align: center"
            headerStyle="text-align: center"
          >
            <template #header>
              <div style="display: flex">
                <div>
                  <span class="p-input-icon-left">
                    <i class="pi pi-search" />
                    <InputText
                      v-model="filters['global'].value"
                      placeholder="Escriba para buscar"
                    />
                  </span>
                </div>

                <div style="margin-left: auto">
                  <Button
                    label="Nueva farmacia"
                    @click="$refs.modalNuevo.abrir()"
                  />
                </div>
              </div>
            </template>

            <Column field="name" header="Nombre">
              <template #body="slotProps">
                <span>
                  {{ slotProps.data.name }}
                </span>
              </template>
            </Column>

            <Column field="address" header="Dirección">
              <template #body="slotProps">
                <span>
                  {{ slotProps.data.address }}
                </span>
              </template>
            </Column>

            <!-- Modificar -->
            <Column field="modificar" header="Modificar" style="width: 20px">
              <template #body="slotProps">
                <div style="display: flex">
                  <div style="margin: auto">
                    <Button
                      icon="pi pi-pencil"
                      class="p-button-rounded p-button-warning mr-2"
                      @click="$refs.modalModificar.abrir(slotProps.data.id)"
                      style="margin-right: 5px"
                    />
                  </div>
                </div>
              </template>
            </Column>

            <!-- Eliminar -->
            <Column field="eliminar" header="Eliminar" style="width: 20px">
              <template #body="slotProps">
                <div style="display: flex">
                  <div style="margin: auto">
                    <Button
                      icon="pi pi-trash"
                      class="p-button-rounded p-button-danger"
                      @click="eliminar(slotProps)"
                    />
                  </div>
                </div>
              </template>
            </Column>
          </DataTable>
        </div>
      </template>
    </Card>
  </main>

  <modal-nuevo ref="modalNuevo" @actualizar-tabla="obtenerTodos"></modal-nuevo>

  <modal-modificar
    ref="modalModificar"
    @actualizar-tabla="obtenerTodos"
  ></modal-modificar>

  <ConfirmDialog></ConfirmDialog>
</template>

<script>
import { ElMessage, ElMessageBox } from "element-plus";
import { FilterMatchMode, FilterOperator } from "primevue/api";

import ModalNuevo from "./modales/nuevo.vue";
import ModalModificar from "./modales/modificar.vue";

export default {
  components: {
    ModalNuevo,
    ModalModificar,
  },

  data() {
    return {
      categorias: [],
      loading: false,
      filters: {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
      },
    };
  },

  mounted() {
    this.obtenerTodos();
  },

  methods: {
    async obtenerTodos() {
      this.categorias = [];
      this.loading = true;
      await this.axios.get("/api/farmacias").then((response) => {
        if (response.data.code == 200) {
          this.categorias = response.data.data;
        }
      });

      this.loading = false;
    },

    async eliminar(row) {
      this.$confirm.require({
        header: "Confirmación",
        message: "¿Está seguro que desea eliminar la farmacia?",
        icon: "pi pi-info-circle",
        acceptClass: "p-button-danger",
        acceptIcon: "pi pi-check",
        rejectIcon: "pi pi-times",
        accept: () => {
          this.eliminarFarmacia(row);
        },
        reject: () => {
          // this.$toast.add({severity:'error', summary:'Rejected', detail:'You have rejected', life: 3000});
        },
        onHide: () => {
          // this.$toast.add({severity:'error', summary:'Hide', detail:'You have hidden', life: 3000});
        },
      });
    },

    async eliminarFarmacia(row) {
      await this.axios
        .delete("/api/farmacia/" + row.data.id)
        .then((response) => {
          if (response.data.code == 200) {
            this.$toast.add({
              severity: "success",
              summary: "Mensaje de confirmación",
              detail: "Farmacia eliminada con éxito",
              life: 3000,
            });
            this.obtenerTodos();
          }
        });
    },

    moneda(x) {
      return x.toLocaleString("es-AR");
    },
  },
};
</script>

<style>
.product-image {
  width: 70px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
  border-radius: 8px;
  padding: 3px;
}

.headerClass {
  text-align: center !important;
}

.p-column-header-content {
  text-align: center !important;
  align-content: center !important;
  /* border: 1px solid red !important; */
}

.p-column-title {
  /* border: 1px solid green !important; */
  text-align: center !important;
  align-content: center !important;
}
</style>
