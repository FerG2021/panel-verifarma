<template>
  <div class="background-login">
    <Toast />

    <el-card class="card-login">
      <template #header>
        <div style="font-size: 5vh; text-align: center">
          Bienvenido a Verifarma
        </div>
      </template>
      <div class="contenedor-login" v-loading="loadingLogin">
        <div class="formulario">
          <img src="verifarmalogo.png" class="img-login" />
          <el-card class="card-form">
            <form
              @submit.prevent="handleSubmit(!v$.$invalid)"
              class="p-fluid"
              style="margin-top: 30px"
            >
              <div class="field">
                <div class="p-float-label">
                  <InputText
                    id="email"
                    inputId="integeronly"
                    v-model="form.email"
                    style="width: 100%"
                    :class="{ 'p-invalid': v$.email.$invalid && submitted }"
                  />
                  <label
                    for="dni"
                    :class="{ 'p-error': v$.email.$invalid && submitted }"
                    >Usuario</label
                  >
                </div>
                <small
                  v-if="
                    (v$.email.$invalid && submitted) ||
                    v$.email.$pending.$response
                  "
                  class="p-error"
                  >{{
                    v$.email.required.$message.replace("Value", "DNI")
                  }}</small
                >
              </div>

              <div class="field">
                <div class="p-float-label">
                  <Password
                    id="email"
                    v-model="form.password"
                    toggleMask
                    style="width: 100%"
                    :class="{ 'p-invalid': v$.password.$invalid && submitted }"
                    :feedback="false"
                  />
                  <label
                    for="password"
                    :class="{ 'p-error': v$.password.$invalid && submitted }"
                    >Contraseña</label
                  >
                </div>
                <small
                  v-if="
                    (v$.password.$invalid && submitted) ||
                    v$.password.$pending.$response
                  "
                  class="p-error"
                  >{{
                    v$.password.required.$message.replace("Value", "DNI")
                  }}</small
                >
              </div>

              <Button
                type="submit"
                label="Ingresar"
                class="mt-2"
                @click="login()"
              />
            </form>
          </el-card>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import { watch } from "@vue/runtime-core";
import { ElMessage, ElMessageBox } from "element-plus";
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
      form: {
        email: "",
        password: "",
      },
      loadingLogin: false,
      rules: {
        email: [
          {
            required: true,
            message: "Por favor ingrese su mail.",
            trigger: "change",
          },
        ],
        password: [
          {
            required: true,
            message: "Por favor ingrese su contraseña.",
            trigger: "change",
          },
        ],
      },
    };
  },

  validations() {
    return {
      email: {
        required: helpers.withMessage("El email es requerido", required),
      },
      password: {
        required: helpers.withMessage("La contraseña es requerida", required),
      },
    };
  },

  methods: {
    async login() {
      let existeError = 0;
      if (
        this.form.email == null ||
        this.form.email == "" ||
        this.form.password == null ||
        this.form.password == ""
      ) {
        this.$toast.add({
          severity: "error",
          summary: "Campos incompletos",
          detail: "Se deben completar todos los campos",
          life: 3000,
        });
      } else {
        this.loadingLogin = true;
        await this.axios.get("/sanctum/csrf-cookie");
        await this.axios
          .post("/login", this.form)
          .then((res) => {
            this.$store.dispatch("getUser");
          })
          .catch(function (error) {
            existeError = 1;
            if (error.response) {
              // La respuesta fue hecha y el servidor respondió con un código de estado
              // que esta fuera del rango de 2xx
            } else if (error.request) {
              // La petición fue hecha pero no se recibió respuesta
              // `error.request` es una instancia de XMLHttpRequest en el navegador y una instancia de
              // http.ClientRequest en node.js
            } else {
              // Algo paso al preparar la petición que lanzo un Error
              console.log("Error", error.message);
            }
            console.log(error.config);
          });

        this.loadingLogin = false;

        console.log(existeError);

        if (existeError == 1) {
          this.$toast.add({
            severity: "error",
            summary: "Ocurrió un error",
            detail: "Mail o contraseña incorrectos",
            life: 3000,
          });
        }

        return this.$router.replace("/");
      }
    },

    deshabilitarBtnIngresar() {
      if (
        this.form.email == null ||
        this.form.email == "" ||
        this.form.password == null ||
        this.form.password == ""
      ) {
        return true;
      } else {
        return false;
      }
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

      if (!this.showMessage) {
        this.resetForm();
      }
    },

    resetForm() {
      this.form.email = null;
      this.form.password = null;
      this.submitted = false;
    },
  },
};
</script>

<style scoped>
.background-login {
  height: 100vh;
  background-color: var(--dark);
  display: flex;
  align-items: center;
  justify-content: center;
}

.card-login {
  width: 65vh;
}

.contenedor-login {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.material-icons {
  font-size: 18vh;
  color: var(--dark);
}

.card-form {
  height: 100%;
}

.formulario {
  width: 90vh;
  height: 100%;
  text-align: center;
  display: block;
}

.field {
  margin-bottom: 1.5rem;
}

.header {
  margin: 0px !important;
}

.img-login {
  height: 20vh;
}
</style>
