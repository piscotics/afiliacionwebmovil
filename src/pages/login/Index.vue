<template>
  <div class="full-width row justify-center items-end content-center">
    <div class=" q-pa-md" style="width: 500px;">
      <q-card>
        <q-card-section class="text-center">
          <q-img src="@/assets/pisco_logo.png" style="width: 250px" />
        </q-card-section>

        <q-card-section>
          <q-form ref="form_login">
            <q-input
              filled
              v-model="username"
              label="Usuario"
              lazy-rules
              :rules="loginRules"
            />

            <q-input
              type="password"
              filled
              v-model="password"
              label="Contraseña"
              lazy-rules
              :rules="loginRules"
            />

            <div class="flex flex-center col-12 btn-estandar">
              <q-btn
                label="Login"
                type="button"
                @click.prevent="login"
                class="col-6 btn-primary"
                rounded
                :disable="!btnLogin"
              />
              <q-btn
                v-if="isMobile"
                label="Configuración"
                type="button"
                flat
                class="btn-optional col-6"
                to="/config"
              />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </div>
  </div>
</template>

<script>
import api from "src/services/api";
import { Security } from "src/services/http/security";
//import endpoint from "src/services/endpoints";
import NotifyDialog from "src/services/notify";
import axios from "axios";
export default {
  name: "login",

  data() {
    return {
      URLactual :"",
      url: "@/assets/img/pisco_logo.png",
      btnLogin: true,
      username: "devfko",
      password: "12345",
      loginRules: [val => (val && val.length > 0) || "Campo Obligatorio (*)"]
    };
  },


  beforeMount() {
      

    this.loadDataConfig();
    var URLactual = window.location;
     //alert(URLactual);
    let value;
      if(this.isMobile){
      // Se establece la URL de la configuración, para evitar
      // que tenga que cerrar la App y volver a ingresar
        value = this.$q.localStorage.getItem("pisco-afilweb");
        if (value != null) {
            api.baseURL = value.server;
             axios.defaults.baseUrl = value.server;
        }
      }else{
        //Guardar la configuración de conexión
        let dataConfig = {
           server: "http://" + URLactual.hostname + ":9051",
           subdomain: URLactual.hostname
        };

        //probar eliminando local store de pisco-afilweb
        
        if(this.$q.localStorage.isEmpty = false){
          this.$q.localStorage.remove("pisco-afilweb");
          this.$q.localStorage.set("pisco-afilweb", dataConfig);
         
        }else{
          this.$q.localStorage.set("pisco-afilweb", dataConfig);
         
        }

          axios.defaults.baseUrl = this.server;
        value = this.$q.localStorage.getItem("pisco-afilweb");
      }

      console.log("axios",  axios.defaults.baseUrl)
      console.log("api",   api.baseURL)
      console.log("value",   value)
    
  },

  methods: {
    loadDataConfig() {

       
     localStorage.setItem("identificacionTitular", "") 
     localStorage.setItem("contratoTitular","")  

      var URLactual = window.location;
     // alert(URLactual);
      let value;
      if(this.isMobile){
      // Se establece la URL de la configuración, para evitar
      // que tenga que cerrar la App y volver a ingresar
        value = this.$q.localStorage.getItem("pisco-afilweb");
         if (value != null) {
            api.baseURL = value.server;
             axios.defaults.baseUrl = value.server;
        }
      }else{
        //Guardar la configuración de conexión
        let dataConfig = {
           server: "http://" + URLactual.hostname + ":9051",
           subdomain: URLactual.hostname
        };

        //probar eliminando local store de pisco-afilweb
        
        if(this.$q.localStorage.isEmpty = false){
          this.$q.localStorage.remove("pisco-afilweb");
          this.$q.localStorage.set("pisco-afilweb", dataConfig);
         
        }else{
          this.$q.localStorage.set("pisco-afilweb", dataConfig);
       
        }

          axios.defaults.baseUrl = this.server;
        value = this.$q.localStorage.getItem("pisco-afilweb");
      }
      if (value == null) {
        // this.triggerNegative("No existen parámeros de conexión configurados");
        this.btnLogin = false;
      }

         console.log("axios",  axios.defaults.baseUrl)
      console.log("api",   api.baseURL)
      console.log("value",   value)
    
    },
    async login() {
      let state = true;
      await this.$refs.form_login.validate().then(success => {
        if (!success) {
          state = false;
        }
      });

      if (state) {

        this.$q.loading.show({
          message:
            "<span class='text-orange text-weight-bold'>Consultando, por favor espere...</span>"
        });

        let credentials = {
          username: this.username,
          password: this.password
        };
        await Security.login(credentials).then((response) => {
          if (response?.status === 200) {
            this.isAuthenticated = true;
            this.user_logged = response.data;
            this.$router.push({ path: "/home" });
            localStorage.setItem("UsuarioLogueado",this.username)
            NotifyDialog.triggerPositive(
             
              `Bienvenido, ${response.data.nombre} ${response.data.apellido}`
            );
          } else {
            NotifyDialog.triggerNegative(
              "Usuario incorrecto, no existe o no está configurado correctamente"
            );
          }
        })

        this.$q.loading.hide();
      }
    }
  },

  computed: {
    isMobile: {
      //Se captura el valor de la variable isMobile del framework
      get() {
        return this.$store.state.platform_mobile.isMobile;
      },
      set(val) {
        this.$store.commit("platform_mobile/setMobile", val);
      }
    },
    isAuthenticated: {
      get() {
        return this.$store.state.user_authenticated.isAuthenticated;
      },
      set(val) {
        this.$store.commit("user_authenticated/setAuthenticated", val);
      }
    },
    user_logged: {
      get() {
        return this.$store.state.user_authenticated.user_logged;
      },
      set(val) {
        this.$store.commit("user_authenticated/setUserLogged", val);
      }
    }
  }
};
</script>

<style></style>
