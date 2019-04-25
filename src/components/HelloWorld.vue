<template>
  <div>
    <h1>Hello Count</h1>
    <div>
      <form>
        <h2>age</h2>
        <label for="kids">0-18</label><input type="checkbox" name="age" v-model="kids" id="kids"/>
        <label for="adults">18-65</label><input type="checkbox" name="age" v-model="adults" id="adults"/>
        <label for="oap">65+</label><input type="checkbox" name="age" v-model="oap" id="oap"/>
        <h2>gender</h2>
          <label for="male">male</label><input type="checkbox" name="age" v-model="male" id="male"/>
          <label for="female">female</label><input type="checkbox" name="age" v-model="female" id="female"/>
        <h2>subcohorts</h2>
          <label for="next">NEXT</label><input type="checkbox" name="next" v-model="next" id="next"/>
          <label for="hair">HAIR</label><input type="checkbox" name="hair" v-model="hair" id="hair"/>
          <label for="gwas">GWAS</label><input type="checkbox" name="gwas" v-model="gwas" id="gwas"/>
          <label for="imal">IMAL</label><input type="checkbox" name="imal" v-model="imal" id="imal"/>
          <label for="dagx">DAGX</label><input type="checkbox" name="dagx" v-model="dagx" id="dagx"/>
          <label for="myst">MYST</label><input type="checkbox" name="myst" v-model="myst" id="myst"/>
          <label for="ugli">UGLI</label><input type="checkbox" name="ugli" v-model="ugli" id="dagx"/>
      </form>
    </div>
    <div>
      <h2>Number of participants: {{numberOfParticipants}}</h2>
      {{query}}
    </div>
  </div>
</template>

<script>
const participant_groups = 'lifelines_aaaac2sufodaknv2bzvip3qaai'

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data () {
    return {
      kids: false,
      adults: true,
      oap: false,
      male: false,
      female: false,
      imal: false,
      next: false,
      hair: false,
      gwas: false,
      imal: false,
      dagx: false,
      myst: false,
      ugli: false,
      numberOfParticipants: 0
    }
  },
  computed: {
    query () {
      const queryParts = []
      const ageGroups = []
      if (this.kids) {
        ageGroups.push(0)
      }
      if (this.adults) {
        ageGroups.push(1)
      }
      if (this.oap) {
        ageGroups.push(2)
      }
      if (ageGroups.length) {
        queryParts.push('age_group=in=(' + ageGroups.join(',') + ')')
      }
      const genders = []
      if (this.male) {
        genders.push(1)
      }
      if (this.female) {
        genders.push(2)
      }
      if (genders.length) {
        queryParts.push('gender=in=(' + genders.join(',') + ')')
      }
      const subcohorts = []
      if (this.next) {
        subcohorts.push('next==true')
      }
      if (this.hair) {
        subcohorts.push('hair==true')
      }
      if (this.gwas) {
        subcohorts.push('gwas==true')
      }
      if (this.imal) {
        subcohorts.push('imal==true')
      }
      if (this.dagx) {
        subcohorts.push('dagx==true')
      }
      if (this.myst) {
        subcohorts.push('myst==true')
      }
      if (this.ugli) {
        subcohorts.push('ugli==true')
      }
      if (subcohorts.length) {
        queryParts.push('(' + subcohorts.join(',') + ')')
      }
      return queryParts.join(';')
    }
  },
  methods: {
    fetchNumberOfPartipants () {
      const qParam = this.query ? `&q=${this.query}` : ''
      fetch(`http://localhost:8080/api/v2/${participant_groups}?molgenis-token=test${qParam}&num=10000`)
        .then(it => it.json())
        .then(it => this.numberOfParticipants = it.items.map(it => it.count).reduce((sum, item) => item + sum, 0))
    }
  },
  watch: {
    query: function () {
      this.fetchNumberOfPartipants()
    }
  },
  created () {
    
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
