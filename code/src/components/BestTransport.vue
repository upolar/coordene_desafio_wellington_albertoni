<template>
  <div class="bestTransport">
    <div class="title">
        <b-navbar toggleable="lg" type="dark" variant="info">
          <b-navbar-brand class="ml-2">
            <b>{{ appName }}</b>
          </b-navbar-brand>
        </b-navbar>
      </div>

    <div id="frete" class="frete">
      <div class="forms">

        <div class="destino">
          <h4>Destino</h4>
          <select v-model="selected">
            <option disabled value="" selected>Escolha uma cidade</option>
            <option v-for="city in cities" :key="city" required> {{city}}</option>
          </select> 
        </div> 

        <div class="peso">
          <h4>Peso</h4>
          <input v-model.number="peso"/>
        </div>

        <button v-on:click="getAlternatives">Analisar</button>

        <div id="result"></div>
      </div>
      
    </div>

  </div>
  

</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
      const appName = ''

    return {
      appName,
      selected: null,
      transport: [],
      peso: 0,
      cities: []
    }
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = 'Melhor Frete';
    let target = 'http://localhost:3000/transport';

    const options = {
      method: 'GET',
      header: {
        Accept: "application/json",
        "Content-Type": "application/json; charset=utf-8",
      }
    };
    
    fetch(target, options)
      .then((response) =>
          response.json())
      .then((data) => {
          this.transport = data
          this.loadCitiesList()
        });
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    async loadCitiesList() {
      /* Carrega as cidades (sem repetir elementos) em um objeto Set e
       armazena em this.cities */
      let cities = new Set()

      try {
        let data = await this.transport;
        data.forEach(function(valor) {
            cities.add(valor['city'])
          }
        )
      } catch (err){
        console.error(err)
      }

      this.cities = cities
   },
   getAlternatives() {
   }
  },
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
</style>
