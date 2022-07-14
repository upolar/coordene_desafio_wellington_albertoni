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
        <h1>Como é o frete que você precisa?</h1>
        <hr>
        <div class="destino mt-3">
          <h3>Destino</h3>
          <select v-model="selected">
            <option disabled value="" selected>Escolha uma cidade</option>
            <option v-for="city in getCities" :key="city" required> {{city}}</option>
          </select> 
        </div> 

        <div class="peso mt-3">
          <h3>Peso</h3>
          <input v-model.number="peso"/>
        </div>

        <b-button v-on:click="getAlternatives" class="mt-3">Analisar</b-button>

        <div id="result" class="mt-3" style="display: none">
          <h4>Estas são as melhores alternativas de frete que encontramos para você.</h4>
          <hr>
          <div id="cheaper" class="alternative mt-4">
            <b-icon icon="coin" ></b-icon>
            <div id="csec">
            </div>
          </div>

          <div id="faster" class="alternative mt-4">
            <b-iconstack>
              <b-icon stacked icon="list-nested" shift-h="-8" rotate="180"></b-icon>
              <b-icon stacked icon="stopwatch"></b-icon>
            </b-iconstack>
            <div id="fsec" >
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  

</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
  BIcon,
  BIconstack
} from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
    BIcon,
    BIconstack
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
    var result = document.getElementById("result")
    var cheaper = document.getElementById("csec");
    var faster = document.getElementById("fsec")
    
    var group = this.getGroupCitySelected();

    let cheaperHTML = '';
    let fasterHTML = '';
    let cheaperItem = null;
    let fasterItem = null;
    
    
    if (group && this.peso > 0) {
      let isPesoHeavy = this.peso > 100;
      let priceCheaper;
      let priceFaster;
      
      let localeBrlToFixed = (val, fixed=2) => { // Arredonda para 2 casas decimais e retorna formatado em BRL Currency
        return Number(val.toFixed(fixed)).toLocaleString("pt-BR", {style:"currency", currency: "BRL"})
      };

      cheaperItem = this.getCheaper(group, isPesoHeavy);
      fasterItem = this.getFaster(group);

      priceCheaper = localeBrlToFixed(this.getPrice(cheaperItem, isPesoHeavy) * this.peso);
      cheaperHTML = `Frete mais barato: <strong>Transportadora ${cheaperItem['name']} - ${priceCheaper} - ${cheaperItem['lead_time']}</strong>`;

      // Se o tempo do mais rápido e do mais barato forem iguais não existe tempo mais rápido.
      if (fasterItem['lead_time'] == cheaperItem['lead_time']) {   
        fasterHTML = `Nós não temos uma opção de frete mais rápido disponível :(`;
      } else {
        priceFaster = localeBrlToFixed(this.getPrice(fasterItem, isPesoHeavy) * this.peso);
        fasterHTML = `Frete mais rápido: <strong>Transportadora ${fasterItem['name']} - ${priceFaster} - ${fasterItem['lead_time']}</strong>`;
      }
      faster.innerHTML = fasterHTML;
      cheaper.innerHTML = cheaperHTML;
      result.style.display = "block";
    }
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

.frete {
  width: 80vh;
  margin: 8vh 0vh 0vh 10vh;
}
.frete h1{
  font-size: 1.5em;
}

.frete h1, .frete h2{
  font-weight: bold;
}
.frete > input select {
  margin: 1vh 1vh 1vh 1vh;
}

.alternative {
  border-style: dotted;
  border-radius: 2px;
}

#fsec, #csec{
  margin-left: 10px;
  display: inline;
}
#faster.alternative{
  padding: 15px;
  background-color: #c9daf8;
  border-color: #4883dd;
}

#cheaper.alternative {
  padding: 15px;
  background-color: #d9ead3;
  border-color: #508a2e;
}
</style>
