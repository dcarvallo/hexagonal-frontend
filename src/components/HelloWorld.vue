<template>
  <v-container>
    <v-row class="text-center flex justify-center">
     

      <v-col class="mb-4"  md="4">
        <h1 class="display-2 font-weight-bold my-3">
          Consulta datos
        </h1>

        <v-form @submit.prevent="buscar" ref="entryForm">
          <v-text-field
          autocomplete="off"
            v-model="limite"
            :rules="[v => !!v || 'Campo requerido', v => /^\d+$/.test(v) || 'El campo debe ser un numero']"
            label="Ingrese limite superior"
            required
          ></v-text-field>
          
          <v-btn type="submit" class="rounded-xl primary block">Buscar
            <v-progress-circular
                v-if="loading"
                indeterminate
                color="primary"
                style="margin-left: 10px"
            ></v-progress-circular>
          </v-btn>
        </v-form>
      </v-col>

    </v-row>
    <v-dialog
      v-model="dialog"
      width="500"
    >
     
      <v-card >
        <v-card-title class="text-center " :class=" !estado || maxIntentos ? 'red white--text' : 'yellow gray--text' ">
          {{estado && !maxIntentos ? 'Resultado Encontrado': '204'}}
        </v-card-title>

        <v-card-text v-if="estado && !maxIntentos" class="pt-5">
              <div class="text-center">
                 <p> ID - {{resultado.id}} </p>
                  <p> {{resultado.nombre}} </p>
                  <p> Se necesitaron {{resultado.intentos}} de 3 </p>
                  <p> Busqueda realizada en {{resultado.tiempo}} milisegundos </p>
                  
              </div>
          </v-card-text>
          <v-card-text v-if="maxIntentos" class="pt-5">
            <p> Cantidad maxima de intentos superada </p>
          </v-card-text>
           <v-card-text v-if="!estado" class="pt-5">
              <p> El id proporcionado no corresponde a un usuario activo </p>
              <p> Tiempo de respuesta fallida - {{resultado.tiempo}} milisegundos </p>
          </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="primary"
            text
            @click="dialog = false"
          >
            Aceptar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import axios from 'axios';
  export default {
    name: 'HelloWorld',

    data() {
      return {
        dialog: false,
        limite: '',
        loading: false,
        resultado:{
          id:'',
          nombre:'',
          intentos: 0,
          tiempo:''
        },
        maxIntentos: false,
        estado: false,
      }
    },
    methods:{
      buscar(){
        let self = this
        if (self.$refs.entryForm.validate()){
            if(this.resultado.intentos >= 3) {
              this.maxIntentos = true
              this.dialog = true
              return;
            }
            axios.interceptors.request.use( x => {
                // to avoid overwriting if another interceptor
                // already defined the same object (meta)
                x.meta = x.meta || {}
                x.meta.requestStartedAt = new Date().getTime();
                return x;
            })

            axios.interceptors.response.use( x => {
              this.resultado.tiempo = new Date().getTime() - x.config.meta.requestStartedAt
                    console.log(`Execution time for: ${x.config.url} - ${ new Date().getTime() - x.config.meta.requestStartedAt} ms`)
                    return x;
                },
                // Handle 4xx & 5xx responses
                x => {
                  this.resultado.tiempo = new Date().getTime() - x.config.meta.requestStartedAt
                  console.error(`Execution time for: ${x.config.url} - ${new Date().getTime() - x.config.meta.requestStartedAt} ms`)
                  throw x;
                }
            )

            console.log(this.$refs.entryForm.validate())
            axios.get('https://jsonplaceholder.typicode.com/users/'+this.limite)
            .then(res => {
              this.resultado.intentos++
              this.estado = true
              this.dialog = true
              this.resultado.id = res.data.id
              this.resultado.nombre = res.data.name
              console.log(this.resultado)
            })
            .catch(err => {
              this.estado = false
              this.dialog = true
              console.log(err); 
            })

         }
      }
    },
    
  }
</script>
