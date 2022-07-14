<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>
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
      cities: []
    }
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = 'Melhor Frete'
    let target = 'http://localhost:3000/transport'

    const options = {
      method: 'GET',
      header: {
        Accept: "application/json",
        "Content-Type": "application/json; charset=utf-8",
      }
    }
    
    this.cities = fetch(target, options)
      .then((response) => response.json())
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
      console.log(this.appName)
    },
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
