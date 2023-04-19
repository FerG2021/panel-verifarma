<template>
  <div>
    <Toast />

    <Dialog
      v-model:visible="display"
      icon="pi pi-refresh"
      :style="{ width: '35%' }"
      class="flex justify-content-center"
      :draggable="false"
    >
      <template #header>
        <h3 style="margin: 0px">
          <i class="pi pi-stop" style="font-size: 20px" /> Modificar farmacia
        </h3>
      </template>

      <div style="margin-top: 5px; width: 100%">
        <!-- <h5 style="margin: 0px">DNI</h5> -->
        <div style="display: flex" v-if="loading == true">
          <ProgressSpinner
            aria-label="Basic ProgressSpinner"
            style="margin: auto"
          />
        </div>
        <form
          @submit.prevent="handleSubmit(!v$.$invalid)"
          class="p-fluid"
          style="margin-top: 30px"
          :loading="loading"
          v-else
        >
          <!-- Nombre -->
          <div class="field">
            <div class="p-float-label">
              <InputText
                id="nombre"
                inputId="integeronly"
                v-model="v$.nombre.$model"
                style="width: 100%"
                :class="{ 'p-invalid': v$.nombre.$invalid && submitted }"
              />
              <label
                for="nombre"
                :class="{ 'p-error': v$.nombre.$invalid && submitted }"
                >Nombre <span style="color: red">*</span></label
              >
            </div>
            <small
              v-if="
                (v$.nombre.$invalid && submitted) ||
                v$.nombre.$pending.$response
              "
              class="p-error"
              >{{
                v$.nombre.required.$message.replace("Value", "Nombre")
              }}</small
            >
          </div>

          <!-- Direccion -->
          <div class="field">
            <div class="p-float-label">
              <InputText
                id="direccion"
                v-model="direccion"
                toggleMask
                style="width: 100%"
                :class="{ 'p-invalid': v$.direccion.$invalid && submitted }"
              />
              <label
                for="direccion"
                :class="{ 'p-error': v$.direccion.$invalid && submitted }"
                >Dirección <span style="color: red">*</span></label
              >
            </div>
            <small
              v-if="
                (v$.direccion.$invalid && submitted) ||
                v$.direccion.$pending.$response
              "
              class="p-error"
            >
              {{ v$.direccion.required.$message.replace("Value", "Dirección") }}
            </small>
          </div>

          <div class="contenedor-map">
            <GMapMap
              :center="center"
              :zoom="15"
              map-type-id="terrain"
              style="width: 32vw; height: 40vh; margin: auto"
              @click="addMarker"
            >
              <GMapMarker
                :key="index"
                v-for="(m, index) in markers"
                :position="m.position"
                :clickable="true"
                :draggable="true"
              />
            </GMapMap>
          </div>

          <Button
            label="Guardar"
            type="submit"
            class="mt-2"
            :loading="loadingBtnGuardar"
          />
        </form>
      </div>
    </Dialog>
  </div>
</template>

<script>
import { email, required } from "@vuelidate/validators";
import { useVuelidate } from "@vuelidate/core";
import { helpers } from "@vuelidate/validators";

export default {
  setup: () => ({ v$: useVuelidate() }),

  data() {
    return {
      display: false,
      submitted: false,
      isFormValid: false,
      loadingBtnGuardar: false,
      loading: true,

      // form
      id: null,
      nombre: "",
      direccion: "",
      lat: null,
      lng: null,

      center: { lat: null, lng: null },
      markers: [
        {
          position: {
            lat: null,
            lng: null,
          },
        },
      ],
    };
  },

  validations() {
    return {
      nombre: {
        required: helpers.withMessage("El nombre es requerido", required),
      },

      direccion: {
        required: helpers.withMessage("La dirección es requerida", required),
      },
    };
  },

  methods: {
    abrir(id) {
      this.id = id;
      this.submitted = false;
      this.display = true;
      // this.isFormValid = false;
      this.resetForm();
      this.getDatos();
    },

    cerrar() {
      this.display = false;
    },

    async getDatos() {
      this.loading = true;
      await this.axios.get("/api/farmacia/" + this.id).then((response) => {
        if (response.data.code == 200) {
          this.nombre = response.data.data.name;
          this.direccion = response.data.data.address;
          this.lat = parseFloat(response.data.data.lat);
          this.lng = parseFloat(response.data.data.lng);

          this.center.lat = parseFloat(response.data.data.lat);
          this.center.lng = parseFloat(response.data.data.lng);

          this.markers[0].position.lat = parseFloat(response.data.data.lat);
          this.markers[0].position.lng = parseFloat(response.data.data.lng);

          this.loading = false;
        }
      });
    },

    addMarker(value) {
      this.markers[0].position.lat = value.latLng.lat();
      this.markers[0].position.lng = value.latLng.lng();

      this.lat = value.latLng.lat();
      this.lng = value.latLng.lng();
    },

    handleSubmit(isFormValid) {
      this.isFormValid = isFormValid;
      this.submitted = true;

      if (!isFormValid) {
        return;
      }

      this.toggleDialog();
    },

    toggleDialog() {
      this.showMessage = !this.showMessage;
      this.guardar();
    },

    resetForm() {
      this.loadingBtnGuardar = false;
      this.submitted = false;

      this.nombre = "";
      this.direccion = "";
      this.lat = null;
      this.lng = null;
    },

    async guardar() {
      this.loadingBtnGuardar = true;

      if (
        this.lat == null ||
        this.lat == "" ||
        this.lng == null ||
        this.lng == ""
      ) {
        this.$toast.add({
          severity: "error",
          summary: "Se ha producido un error",
          detail: "Se deben completar la latitud y longitud",
          life: 5000,
        });

        return;
      }

      let params = {
        nombre: this.nombre,
        direccion: this.direccion,
        lat: this.lat,
        lng: this.lng,
      };

      await this.axios
        .put("/api/farmacia/" + this.id, params)
        .then((response) => {
          if (response.data.code == 200) {
            this.$toast.add({
              severity: "success",
              summary: "Mensaje de confirmación",
              detail: "Farmacia modificada con éxito",
              life: 3000,
            });

            this.display = false;
            this.$emit("actualizarTabla");
          } else {
            this.$toast.add({
              severity: "error",
              summary: "Se ha producido un error",
              detail: response.data.data,
              life: 5000,
            });

            // for (const property in response.data.data) {
            //   // console.log(`${property}: ${response.data.data[property]}`);
            //   this.$toast.add({
            //     severity: "error",
            //     summary: "Se ha producido un error",
            //     detail: `${response.data.data[property]}`,
            //     life: 5000,
            //   });
            // }
          }
        });

      this.loadingBtnGuardar = false;
    },
  },
};
</script>

<style scoped>
.field {
  margin-bottom: 1.5rem;
}

.header {
  margin: 0px !important;
}

.p-dialog {
  border-radius: 30% !important;
}

.contenedor-map {
  margin-bottom: 20px;
  display: flex;
}
</style>
