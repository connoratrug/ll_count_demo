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

    <h2>subsections</h2>
    <select v-model="selectedSubSection">
      <option v-for="subSection in subSections" :key="subSection.subsection_id" :value="subSection.subsection_id" >{{subSection.name}}</option>
    </select>
    <div>
      <h2>variable counts</h2>
      <table>
        <tr><th></th><th v-for="round in rounds" :key="round">{{round}}</th></tr>
        <tr v-for="variable in countsPerVariable" :key="variable.name">
          <th>{{variable.name}}</th>
          <td v-for="round in rounds" :key="round">{{variable.countsPerRound[round]}}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
const participantGroupsEntityTypeId = 'lifelines_aaaac2sufodaknv2bzvip3qaai'
const variableEntityTypeId = 'lifelines_aaaac2suggtn2nv2bzvip3qaaq'
const subSectionEntityTypeId = 'lifelines_aaaac2suge5kmnv2bzvip3qaai'
const variableTreeEntityTypeId = 'lifelines_aaaac2sugltc4nv2bzvip3qaay'

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
      subSections: [],
      selectedSubSection: null,
      groupIds: [],
      countsPerVariable: [],
      rounds: []
    }
  },
  computed: {
    qParam () {
      return this.query ? `;${this.query}` : ''
    },
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
    async fetchSubSections () {
      const response = await fetch(`http://localhost:8080/api/v2/${subSectionEntityTypeId}?molgenis-token=test`)
      const subSectionsJson = await response.json()
      this.subSections = subSectionsJson.items
      this.selectedSubSection = this.subSections[0].subsection_id
    },
    async fetchVariableGroups () {
      const response = await fetch(`http://localhost:8080/api/v2/${variableTreeEntityTypeId}?q=subsection_id==${this.selectedSubSection}&attrs=variable_id(groups,name)&molgenis-token=test`)
      const variableJson = await response.json()
      const variables = variableJson.items
        .map(item => item.variable_id)
        .map(variable => ({
          name: variable.name, 
          groupIds: variable.groups.map(group => group.variable_group_id)
        }))
      const allGroupIds = variables.flatMap(variable => variable.groupIds)

      this.groupIds = allGroupIds;
      
      const participantsResponse = await fetch(`http://localhost:8080/api/v2/${participantGroupsEntityTypeId}?molgenis-token=test&q=variable_group_id=in=(${allGroupIds.join(',')})${this.qParam}&num=10000&attrs=~id,count,variable_group_id(~id,assessment_id(abbreviation))`)
      const assessments = await participantsResponse.json()
      const counts = assessments.items.map(it => ({
          count: it.count, 
          group: it.variable_group_id.variable_group_id, 
          collectionRound: it.variable_group_id.assessment_id.abbreviation
        }))

      this.countsPerVariable = variables.map(variable => {
        const countsForThisVariable = counts.filter(count => variable.groupIds.includes(count.group))

        const countsPerRoundForThisVariable = countsForThisVariable.reduce((accum, item) => {
          if(!accum[item.collectionRound]) {
            accum[item.collectionRound] = 0
          }
          accum[item.collectionRound] += item.count
          return accum
        }, {})

        return {
          name: variable.name,
          countsPerRound: countsPerRoundForThisVariable
        }
      })
      this.rounds = counts.reduce((accum, item) => {
        if(!accum.includes(item.collectionRound)) {
          accum.push(item.collectionRound)
        }
        return accum
      }, []).sort()
    }
  },
  watch: {
    selectedSubSection: function (val) {
      if (val !== null) {
        this.fetchVariableGroups()
      }
    },
    query: function () {
      this.fetchVariableGroups()
    }
  },
  created () {
     this.fetchSubSections()
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

td, th{
  border: 1px solid black;
  padding: 1rem;
}
</style>
