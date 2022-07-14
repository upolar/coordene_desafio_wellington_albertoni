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
            <option v-for="city in getCities" :key="city" required> {{city}}</option>
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
  BNavbarBrand
} from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand
  },
  data() {
      const appName = ''


    return {
      appName,
      selected: null,
      transport: [],
      peso: 0
    }
  },
  computed:{
    getCities() {
      /* Obtem as cidades que a API forneceu e retorna um Set com as cidades únicas */
      let cities = new Set()

      try {
        let data = this.transport;
        data.forEach(function(valor) {
            cities.add(valor['city'])
          }
        );
      } catch (err){
        console.error(err);
      }
      return cities;
    },
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = 'Melhor Frete';
    let target = 'http://localhost:3000/transport';

    const options = {
      method: 'GET',
    };
    
    fetch(target, options)
      .then((response) => response.json())
      .then((data) => {
        this.transport = data
      });
  },
  methods: {
   getGroupCitySelected() {
      /* Obtem um array de transportadores com base na cidade selecionada */
      return this.transport.filter(function(value) {
          return value['city'] == this.selected
      }.bind(this))
    },

   getAlternatives() {
    /* Obtem a transportadora mais rápida e a mais barata e adiciona no HTML a informação. */
    var result = document.getElementById("result");
    var group = this.getGroupCitySelected();
    let cheaperHTML = '';
    let fasterHTML = '';
    let cheaper = null;
    let faster = null;
    
    
    if (group && this.peso > 0) {
      let is_peso_heavy = this.peso > 100;
      let price_cheaper; 
      let price_faster;
      
      let localeBrlToFixed = (val, fixed=2) => { // Arredonda para 2 casas decimais e retorna formatado em BRL Currency
        return Number(val.toFixed(fixed)).toLocaleString("pt-BR", {style:"currency", currency: "BRL"})
      };

      cheaper = this.getCheaper(group, is_peso_heavy);
      faster = this.getFaster(group);

      price_cheaper = localeBrlToFixed(this.getPrice(cheaper, is_peso_heavy) * this.peso);
      if (faster['lead_time'] == cheaper['lead_time']) {
        fasterHTML = `<p> Nós não temos uma opção de frete mais rápido disponível :( </p>`
      } else {
        price_faster = localeBrlToFixed(this.getPrice(faster, is_peso_heavy) * this.peso);
        fasterHTML = `<p> Frete mais rápido: <strong>Transportadora ${faster['name']} - ${price_faster} - ${faster['lead_time']}</strong></p>`;
      }
      
      cheaperHTML = `<p>Frete mais barato: <strong>Transportadora ${cheaper['name']} - ${price_cheaper} - ${cheaper['lead_time']}</strong></p>`;
    }

    result.innerHTML = cheaperHTML + fasterHTML
    
   },
   
  getPrice(item, heavy) {
    /* Retorna o preço com base no peso 
      Peso > 100 - retorna o preço const_transport_heavy 
      Peso < 100 - retorna o preço const_transport_light */
    let type = heavy ? 'cost_transport_heavy' : 'cost_transport_light'

    return Number(item[type].replace(/[^0-9.]/g, ""))
  },

  getCheaper(group, heavy) {
    /* Retorna o frete mais barato dentre as transportadoras do grupo */
    let cheaper = group[0];
    let price_chp = this.getPrice(cheaper, heavy);3

    for (var item in group) {  
      let tmp_price = this.getPrice(group[item], heavy)
      if (tmp_price < price_chp) {
        price_chp = tmp_price;
        cheaper = group[item];
      }
    }
    return cheaper;
  },
  getFaster(group) {
    let faster = group[0];
    var lead_time = (val) => {return val['lead_time'].replace(/[^0-9]/g, "")}

    for (var item in group) {  
      let tmp_time = lead_time(group[item]);

      if (Number(lead_time(faster)) > Number(tmp_time)) {
        faster = group[item];
      }
    }
    return faster
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
