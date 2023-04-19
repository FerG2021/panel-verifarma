<template>
  <main>
    <el-card>
      <template #header>
        <h1>Mi cuenta</h1>
      </template>

      <div>
        <div style="margin-bottom: 20px">
          <div style="margin-right: 10px">Habilitar edición de datos</div>
          <InputSwitch v-model="habilitarEdicion" />
          <div v-if="habilitarEdicion == true">
            <InlineMessage severity="warn"
              >Al modificar algunos de los datos de su cuenta la sesión se
              cerrará y deberá voler a ingresar</InlineMessage
            >
          </div>
        </div>

        <form
          @submit.prevent="handleSubmit(!v$.$invalid)"
          class="p-fluid"
          style="margin-top: 30px"
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
                :disabled="habilitarEdicionInputs()"
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

          <!-- Email -->
          <div class="field">
            <div class="p-float-label">
              <InputText
                id="email"
                inputId="integeronly"
                v-model="v$.email.$model"
                style="width: 100%"
                :class="{ 'p-invalid': v$.email.$invalid && submitted }"
                :disabled="habilitarEdicionInputs()"
              />
              <label
                for="email"
                :class="{ 'p-error': v$.email.$invalid && submitted }"
                >Email <span style="color: red">*</span></label
              >
            </div>
            <small
              v-if="
                (v$.email.$invalid && submitted) || v$.email.$pending.$response
              "
              class="p-error"
              >{{ v$.email.required.$message.replace("Value", "email") }}</small
            >
          </div>

          <!-- Contraseña -->
          <div class="field">
            <div class="p-float-label">
              <InputText
                id="contrasena"
                inputId="integeronly"
                v-model="contrasena"
                style="width: 100%"
                :disabled="habilitarEdicionInputs()"
              />
              <label for="contrasena">Contraseña </label>
            </div>
          </div>

          <!-- Repetir contraseña -->
          <div class="field">
            <div class="p-float-label">
              <InputText
                id="repetirContrasena"
                inputId="integeronly"
                v-model="repetirContrasena"
                style="width: 100%"
                :disabled="habilitarEdicionInputs()"
              />
              <label for="repetirContrasena">Repetir contraseña </label>
            </div>
          </div>

          <Button
            label="Guardar"
            type="submit"
            class="mt-2"
            :loading="loadingBtnGuardar"
          />
        </form>
      </div>
    </el-card>
  </main>
</template>

<script>
import { email, required } from "@vuelidate/validators";
import { useVuelidate } from "@vuelidate/core";
import { helpers } from "@vuelidate/validators";

export default {
  setup: () => ({ v$: useVuelidate() }),

  data() {
    return {
      // form
      id: null,
      nombre: null,
      email: null,
      contrasena: null,
      repetirContrasena: null,

      habilitarEdicion: false,

      loadingBtnGuardar: false,
    };
  },

  validations() {
    return {
      nombre: {
        required: helpers.withMessage("El nombre es requerido", required),
      },
      email: {
        required: helpers.withMessage("El email es requerido", required),
      },
    };
  },

  created() {
    this.getDatosMiCuenta();
  },

  methods: {
    getDatosMiCuenta() {
      this.id = this.$store.state.user.id;
      this.nombre = this.$store.state.user.name;
      this.email = this.$store.state.user.email;
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
      this.onSubmit();
    },

    resetForm() {
      this.nombre = null;
      this.email = null;
      this.contrasena = null;
      this.repetirContrasena = null;
      this.submitted = false;
    },

    habilitarEdicionInputs() {
      if (this.habilitarEdicion == false) {
        return true;
      } else {
        return false;
      }
    },

    async onSubmit() {
      this.loadingBtnGuardar = true;
      let params = {
        id: this.id,
        nombre: this.nombre,
        email: this.email,
      };

      if (this.contrasena != null) {
        params.contrasena = this.contrasena;
      } else {
        params.contrasena = null;
      }

      if (this.repetirContrasena != null) {
        params.repetirContrasena = this.repetirContrasena;
      } else {
        params.repetirContrasena = null;
      }

      const respuestaApi = await this.axios.put(
        "/api/usuario/actualizar/" + this.id,
        params
      );

      if (respuestaApi.data.code == 200) {
        this.$toast.add({
          severity: "success",
          summary: "Mensaje de confirmación",
          detail: "¡Datos de la cuenta modificados con éxito!",
          life: 3000,
        });
        this.$emit("actualizarTabla");
        this.logout();
      } else {
        if (respuestaApi.data.code == 400) {
          let erroresMostrar = "// ";
          let erorres = Object.values(respuestaApi.data.data);

          erorres.forEach((elemento) => {
            erroresMostrar = erroresMostrar + " " + elemento + " //";
          });

          this.$toast.add({
            severity: "error",
            summary: "Se ha producido un error",
            detail: erroresMostrar,
            life: 5000,
          });
        }
      }

      this.loadingBtnGuardar = false;
    },

    deshabilitarBtnGuardarDatos() {
      if (
        this.form.nombre == null ||
        this.form.nombre == "" ||
        this.form.email == null ||
        this.form.email == "" ||
        this.habilitarEdicion == false
      ) {
        return true;
      } else {
        return false;
      }
    },

    async logout() {
      await this.$store.dispatch("logout");
      // redirect
      return this.$router.replace("/login");
    },
  },
};
</script>

<style scoped>
.field {
  margin-top: 30px;
}
</style>
