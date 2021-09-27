<template>
  <div>
    <h3>Dodawanie użytkownika</h3>
    <label>Imie: </label>
    <input v-model="inputName" type="text"><br>
    <label>Nazwisko: </label>
    <input v-model="inputSurname" type="text"><br>
    <label>Dzial: </label>
    <input v-model="inputSection" type="text"><br>
    <label>Wynagrodzenie: </label>
    <input v-model="inputSalary" type="number"><br>
    <label>Waluta: </label>
    <input v-model="inputCurrency" type="text"><br>
    <button v-on:click="handleAddPerson">Dodaj</button>

    <h3>Wyszukiwanie i filtry</h3>
    <label>Imie</label><br>
    <input v-model="inputSearchPersonName" v-on:input="searchPerson" type="text"><br>
    <label>Nazwisko</label><br>
    <input v-model="inputSearchPersonSurname" v-on:input="searchPerson" type="text"><br>
    <ul>
      <li v-for="(item, index) in sectionsNames" :key="index">
        <input type="checkbox" :checked="item.active" @change="handleChangeActiveSection(index)">
        <label :for="item.dzial">{{item.dzial}}</label>
      </li>
    </ul>
    <p>Przedział wynagrodzenia</p>
    <input v-model="rangeMin" v-on:input="searchPerson" type="number"> 
    <span> - </span>
    <input v-model="rangeMax" v-on:input="searchPerson" type="number"> 
    <br><br>

    <h3>Wynik wyszukiwania</h3>
    <table>
      <tr>
        <th>Imie:</th>
        <th>Nazwisko:</th>
        <th>Dział:</th>
        <th>Wynagrodzenie:</th>
      </tr>
      <tr v-for="(item, index) in noNullItems" :key="index">
        <td>{{item.imie}}</td>
        <td>{{item.nazwisko}}</td>
        <td>{{item.dzial}}</td>
        <td>{{item.wynagrodzenieKwota}} {{item.wynagrodzenieWaluta}}</td>
      </tr>
      <tr v-for="(value, name, index) in sectionsSalary" :key="index">
        <td colspan="3">{{name}}</td>
        <td>{{value}} PLN</td>
      </tr>
      <tr>
        <td colspan="3">Suma wynagrodzenia</td>
        <td>{{sumSalary}} PLN</td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: 'App',
  data(){
    return{
      PRACOWNICY: [
        {"imie": "Jan", "nazwisko": "Kowalski", "dzial": "IT", "wynagrodzenieKwota": "3000", "wynagrodzenieWaluta": "PLN"},
        {"imie": "Anna", "nazwisko": "Bąk", "dzial": "Administracja", "wynagrodzenieKwota": "2400.50", "wynagrodzenieWaluta": "PLN"},
        {"imie": "Paweł", "nazwisko": "Zabłocki", "dzial": "IT", "wynagrodzenieKwota": "3300", "wynagrodzenieWaluta": "PLN"},
        {"imie": "Tomasz", "nazwisko": "Osiecki", "dzial": "Administracja", "wynagrodzenieKwota": "2100", "wynagrodzenieWaluta": "PLN"},
        {"imie": "Iwona", "nazwisko": "Leihs-Gutowska", "dzial": "Handlowiec", "wynagrodzenieKwota": "3100", "wynagrodzenieWaluta": "PLN"},
      ],
      sectionsNames: [],
      sectionsSalary: {"imie": "Jan"},
      inputName: "",
      inputSurname: "",
      inputSection: "",
      inputSalary: "",
      inputCurrency: "",
      inputSearchPersonName: "",
      inputSearchPersonSurname: "",
      selectedWorkers: [],
      rangeMin: 0,
      rangeMax: 99999,
      activeSections: [],
      sumSalary: 0
    }
  },
  methods: {
    handleAddPerson() {
      let PRACOWNICY = [...this.PRACOWNICY],
          person = {"imie": this.inputName,
                    "nazwisko": this.inputSurname, 
                    "dzial": this.inputSection, 
                    "wynagrodzenieKwota": this.inputSalary, 
                    "wynagrodzenieWaluta": this.inputCurrency};
      PRACOWNICY.push(person);
      this.PRACOWNICY = PRACOWNICY;
      this.getSectionsNames();
      this.searchPerson();
      this.inputName = "";
      this.inputSurname = "";
      this.inputSection = "";
      this.inputSalary = "";
      this.inputCurrency = "";
    },
    getSectionsNames() {
      let names = []
      this.PRACOWNICY.map(item=>{
        if (!names.some(e => e.dzial === item.dzial))return names.push({dzial: item.dzial, active: true})
      })
      this.sectionsNames = names;
    },
    handleChangeActiveSection(index) {
      this.sectionsNames = (
        [...this.sectionsNames.slice(0,index),
        {...this.sectionsNames[index],
        active: !this.sectionsNames[index].active},
        ...this.sectionsNames.slice(index+1)])
        this.searchPerson();
    },
    searchPerson() {
      let searchName=this.inputSearchPersonName.toLowerCase(), 
          searchSurname=this.inputSearchPersonSurname.toLowerCase(),
          rangeMin=this.rangeMin,
          rangeMax=this.rangeMax,
          activeSections=[];

      this.sectionsNames.map(item=>{
        if(item.active===true)activeSections.push(item.dzial)
      })
      this.activeSections = activeSections;

      const select = this.PRACOWNICY.map(item=>{
        if((item.imie.toLowerCase().indexOf(searchName) === 0)&&
          (item.nazwisko.toLowerCase().indexOf(searchSurname) === 0)&&
          (parseFloat(item.wynagrodzenieKwota)>=rangeMin&&parseFloat(item.wynagrodzenieKwota)<=rangeMax)&&
          (activeSections.includes(item.dzial)))return item;
        else return "";
      });
      this.selectedWorkers = select;

      let result = this.selectedWorkers.reduce((sum, item) => {
        if(this.activeSections.includes(item.dzial))sum[item.dzial] = (sum[item.dzial] || 0) + parseFloat(item.wynagrodzenieKwota);
        return sum;
      }, {});
      this.sectionsSalary = result;

      let sumSalary=0;  
      this.selectedWorkers.map(item=>{if(item!=="")return sumSalary+=parseFloat(item.wynagrodzenieKwota)})
      this.sumSalary = sumSalary;
    }
  },
  computed: {
    noNullItems: function() {
      return this.selectedWorkers.filter(function(item) {
        return item !== "";
      });
    }
  },
  mounted: function () {
  this.$nextTick(function () {
    this.getSectionsNames();
    this.searchPerson();
  })
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  margin-top: 60px;
}
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
th, td {
  padding: 10px;
}
html{
  margin: 10px
}
</style>
