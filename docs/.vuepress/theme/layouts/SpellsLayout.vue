<template>
  <div class="spells">

    <div class="d-flex flex-wrap align-center">
      <Breadcrumb class="mr-auto mb-4" />
      <div class="d-flex flex-wrap align-center">
        <v-btn color="primary" class="mr-4 mb-4" depressed link to="/creation-de-sort/"><v-icon left>mdi-plus</v-icon> Créer un sort</v-btn>
        <MySpellsButton />
      </div>
    </div>

    <h1>Grimoire</h1>

    <div class="columns-toggles d-md-flex d-none align-center mb-2">
      <div><strong>Colonnes affichées :</strong></div>
      <v-checkbox
        class="mt-0 mr-4"
        v-model="showColumn.school"
        label="École"
        hide-details
        @change="setShowColumn"
      ></v-checkbox>
      <v-checkbox
        class="mt-0 mr-4"
        v-model="showColumn.castingTime"
        label="Temps d'incantation"
        hide-details
        @change="setShowColumn"
      ></v-checkbox>
      <v-checkbox
        class="mt-0 mr-4"
        v-model="showColumn.duration"
        label="Durée"
        hide-details
        @change="setShowColumn"
      ></v-checkbox>
      <v-checkbox
        class="mt-0 mr-4"
        v-model="showColumn.concentration"
        label="Concentration"
        hide-details
        @change="setShowColumn"
      ></v-checkbox>
      <v-checkbox
        class="mt-0 mr-4"
        v-model="showColumn.ritual"
        label="Rituel"
        hide-details
        @change="setShowColumn"
      ></v-checkbox>
      <v-checkbox
        class="mt-0 mr-4"
        v-model="showColumn.components"
        label="Composantes"
        hide-details
        @change="setShowColumn"
      ></v-checkbox>
      <v-checkbox
        class="mt-0 mr-4"
        v-model="showColumn.description"
        label="Description"
        hide-details
        @change="setShowColumn"
      ></v-checkbox>
    </div>

    <div class="active-filters mb-2">
      <div class="classes-filter mb-1" v-if="selectedClasses.length > 0">
        <strong>Classes</strong> : <v-chip class="mr-1" v-for="(c, idx) in selectedClasses">{{ c }}</v-chip>
      </div>
      <div class="levels-filter mb-1" v-if="selectedLevels.length > 0">
        <strong>Niveaux de sorts</strong> : <v-chip class="mr-1" v-for="(level, idx) in selectedLevels" v-html="level"></v-chip>
      </div>
      <div class="schools-filter mb-1" v-if="selectedSchools.length > 0">
        <strong>Écoles de magie</strong> : <v-chip class="mr-1" v-for="(school, idx) in selectedSchools">{{ school }}</v-chip>
      </div>
      <div class="components-filter mb-1" v-if="componentVerbal !== undefined || componentSomatic !== undefined || componentMaterial !== undefined">
        <strong>Composantes d'incantation</strong> :
        <v-chip class="mr-1" v-if="componentVerbal === true" dark color="green">verbales</v-chip>
        <v-chip class="mr-1" v-if="componentVerbal === false" dark color="red">verbales</v-chip>

        <v-chip class="mr-1" v-if="componentSomatic === true" dark color="green">somatiques</v-chip>
        <v-chip class="mr-1" v-if="componentSomatic === false" dark color="red">somatiques</v-chip>

        <v-chip class="mr-1" v-if="componentMaterial === true" dark color="green">matérielles</v-chip>
        <v-chip class="mr-1" v-if="componentMaterial === false" dark color="red">matérielles</v-chip>
      </div>
      <div class="concentration-filter mb-1" v-if="mustBeConcentration !== undefined">
        <strong>Concentration</strong> :
        <v-chip class="mr-1" v-if="mustBeConcentration === true" dark color="green">oui</v-chip>
        <v-chip class="mr-1" v-if="mustBeConcentration === false" dark color="red">non</v-chip>
      </div>
      <div class="concentration-filter mb-1" v-if="mustBeRitual !== undefined">
        <strong>Rituel</strong> :
        <v-chip class="mr-1" v-if="mustBeRitual === true" dark color="green">oui</v-chip>
        <v-chip class="mr-1" v-if="mustBeRitual === false" dark color="red">non</v-chip>
      </div>
    </div>

    <v-data-table
      class="data-table"
      :headers="headers"
      :items="spells"
      item-key="key"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc"
      must-sort
      :search="search"
      :items-per-page.sync="itemsPerPage"
      :page.sync="page"
      @page-count="pageCount = $event"
      hide-default-footer
      show-expand
      @click:row="onClickRow"
      mobile-breakpoint="0"
    >

      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length" class="pa-4">
          <Spell :spell="item" />
        </td>
      </template>

      <template v-slot:item.isInSpellBook="{ item }">
        <v-simple-checkbox off-icon="mdi-bookmark-outline" on-icon="mdi-bookmark" @input="toggleSpellInSpellBook(item)" :value="isSpellInSpellBook(item)"></v-simple-checkbox>
      </template>

      <template v-slot:item.title="{ item }">
        <router-link :to="{ path: item.path }" class="subtitle-2">{{ item.title }}</router-link>
      </template>

      <template v-slot:item.frontmatter.level="{ item }">
        <span v-if="item.frontmatter.level == 0">Tour de magie</span>
        <span v-else>{{ item.frontmatter.level }}</span>
      </template>

      <template v-slot:item.frontmatter.concentration="{ item }">
        <span v-if="item.frontmatter.concentration">concentration</span>
      </template>

      <template v-slot:item.frontmatter.ritual="{ item }">
        <span v-if="item.frontmatter.ritual">rituel</span>
      </template>

      <template v-slot:item.frontmatter.components="{ item }">
        <template v-if="item.frontmatter.components">
          <template v-if="item.frontmatter.components.verbal">V</template><template v-if="item.frontmatter.components.verbal && (item.frontmatter.components.somatic || item.frontmatter.components.material)">,</template>
          <template v-if="item.frontmatter.components.somatic">S</template><template v-if="item.frontmatter.components.somatic && item.frontmatter.components.material">,</template>
          <template v-if="item.frontmatter.components.material">M</template>
        </template>
      </template>

      <template v-slot:item.frontmatter.description="{ item }">
        <div v-html="item.frontmatter.description"></div>
      </template>

    </v-data-table>

    <v-row class="align-center  mb-12 pb-6">
      <v-col :cols="12" md="3">
        <v-select v-model="itemsPerPage" :items="itemsPerPageChoices" label="Lignes par page" @change="selectItemPerPage"></v-select>
      </v-col>
      <v-col :cols="12" md="9">
        <v-pagination v-model="page" :length="pageCount" :total-visible="7" @input="changePage"></v-pagination>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import Breadcrumb from '@theme/components/Breadcrumb'
import { setUrlParams, getUrlParameter } from '@theme/util/filterHelpers'
import { isResourceInLibrary, handleTooltips } from '@theme/util'
import Spell from '@theme/components/Spell'
import MySpellsButton from '@theme/global-components/MySpellsButton'
import Cookies from 'js-cookie'

export default {
  components: { Breadcrumb, Spell, MySpellsButton },

  data () {
    return {
      sortBy: 'title',
      sortDesc: false,
      itemsPerPage: 10,
      itemsPerPageChoices: [
        {text: '5', value: 5},
        {text: '10', value: 10},
        {text: '15', value: 15},
        {text: 'Tous', value: -1},
      ],
      page: 1,
      pageCount: 0,
      showColumn: {
        school: true,
        castingTime: true,
        duration: true,
        concentration: true,
        ritual: true,
        components: true,
        description: false,
      }
    }
  },

  computed: {
    ...mapState({
      search: state => state.spellFilters.search,
      classes: state => state.spellFilters.classes,
      levels: state => state.spellFilters.levels,
      schools: state => state.spellFilters.schools,
      mustBeConcentration: state => state.spellFilters.mustBeConcentration,
      mustBeRitual: state => state.spellFilters.mustBeRitual,
      componentVerbal: state => state.spellFilters.componentVerbal,
      componentSomatic: state => state.spellFilters.componentSomatic,
      componentMaterial: state => state.spellFilters.componentMaterial,
    }),

    headers() {
      let headers = [
        { text: "", align: 'center', sortable: false, value: 'isInSpellBook' },
        { text: "Nom", align: 'start', sortable: true, value: 'title' },
        { text: "Niveau", align: 'center', sortable: true, value: 'frontmatter.level' }
      ]
      if (this.showColumn.school && this.$vuetify.breakpoint.mdAndUp) {
        headers.push({ text: "École", align: 'start', sortable: false, value: 'frontmatter.school' })
      }
      if (this.showColumn.castingTime && this.$vuetify.breakpoint.mdAndUp) {
        headers.push({ text: "Temps d'incantation", align: 'start', sortable: false, value: 'frontmatter.casting_time' })
      }
      if (this.showColumn.duration && this.$vuetify.breakpoint.mdAndUp) {
        headers.push({ text: "Durée", align: 'start', sortable: false, value: 'frontmatter.duration' })
      }
      if (this.showColumn.concentration && this.$vuetify.breakpoint.mdAndUp) {
        headers.push({ text: "Concentration", align: 'center', sortable: false, value: 'frontmatter.concentration' })
      }
      if (this.showColumn.ritual && this.$vuetify.breakpoint.mdAndUp) {
        headers.push({ text: "Rituel", align: 'center', sortable: false, value: 'frontmatter.ritual' })
      }
      if (this.showColumn.components && this.$vuetify.breakpoint.mdAndUp) {
        headers.push({ text: "Composantes", align: 'center', sortable: false, value: 'frontmatter.components' })
      }
      if (this.showColumn.description && this.$vuetify.breakpoint.mdAndUp) {
        headers.push({ text: "Description", align: 'start', sortable: false, value: 'frontmatter.description' })
      }
      return headers
    },

    selectedClasses() {
      let result = []
      for (let c of this.classes) {
        if (c.value) {
          result.push(c.label)
        }
      }
      return result
    },

    selectedLevels() {
      let result = []
      for (let level of this.levels) {
        if (level.value) {
          result.push(this.levelDisplay(level))
        }
      }
      return result
    },

    selectedSchools() {
      let result = []
      for (let school of this.schools) {
        if (school.value) {
          result.push(school.label)
        }
      }
      return result
    },

    spells() {
      let results = this.$pagination.pages

      // Filter concentration
      if (this.mustBeConcentration !== undefined) {
        results = results.filter(item => {
          return item.frontmatter.concentration === this.mustBeConcentration
        })
      }

      // Filter ritual
      if (this.mustBeRitual !== undefined) {
        results = results.filter(item => {
          return item.frontmatter.ritual === this.mustBeRitual
        })
      }

      // Filter components
      if (this.componentVerbal !== undefined) {
        results = results.filter(item => {
          return item.frontmatter.components.verbal === this.componentVerbal
        })
      }
      if (this.componentSomatic !== undefined) {
        results = results.filter(item => {
          return item.frontmatter.components.somatic === this.componentSomatic
        })
      }
      if (this.componentMaterial !== undefined) {
        results = results.filter(item => {
          return item.frontmatter.components.material === this.componentMaterial
        })
      }

      // Filter classes
      let selectedClasses = []
      for (var i = 0; i < this.classes.length; i++) {
        if (this.classes[i].value) {
          selectedClasses.push(this.classes[i].label)
        }
      }
      if (selectedClasses.length) {
        let classFiltered = []
        for (var i = 0; i < selectedClasses.length; i++) {
          for (var j = 0; j < results.length; j++) {
            if (results[j].frontmatter.classes.indexOf(selectedClasses[i]) > -1) {
              if (classFiltered.indexOf(results[j]) < 0) {
                classFiltered.push(results[j])
              }
            }
          }
        }
        results = classFiltered
      }

      // Filter levels
      let selectedLevels = []
      for (var i = 0; i < this.levels.length; i++) {
        if (this.levels[i].value) {
          selectedLevels.push(this.levels[i].level)
        }
      }
      if (selectedLevels.length) {
        results = results.filter(item => {
          return selectedLevels.indexOf(item.frontmatter.level) > -1
        })
      }

      // Filter schools
      let selectedSchools = []
      for (var i = 0; i < this.schools.length; i++) {
        if (this.schools[i].value) {
          selectedSchools.push(this.schools[i].label)
        }
      }
      if (selectedSchools.length) {
        results = results.filter(item => {
          return selectedSchools.indexOf(item.frontmatter.school) > -1
        })
      }

      // let csv = ''
      // csv += "niveau;nom;école;temps d'incantation;rituel;durée;concentration;portée;composantes;classes;description\n"
      // for (var spell of results) {
      //   let components = ''
      //   if (spell.frontmatter.components.verbal) {
      //     components += 'V'
      //     if (spell.frontmatter.components.somatic || spell.frontmatter.components.material) {
      //       components += ', '
      //     }
      //   }
      //   if (spell.frontmatter.components.somatic) {
      //     components += 'S'
      //     if (spell.frontmatter.components.material) {
      //       components += ', '
      //     }
      //   }
      //   if (spell.frontmatter.components.material) {
      //     components += 'M'
      //     if (spell.frontmatter.components.materials) {
      //       components += ' (' + spell.frontmatter.components.materials + ')'
      //     }
      //   }
      //   let concentration = ''
      //   if (spell.frontmatter.concentration) {
      //     concentration += 'concentration'
      //   }
      //   let ritual = ''
      //   if (spell.frontmatter.ritual) {
      //     ritual += 'rituel'
      //   }
      //   let classes = spell.frontmatter.classes.join(', ')
      //
      //   csv += spell.frontmatter.level + ';' + spell.title + ';' + spell.frontmatter.school + ';' + spell.frontmatter.casting_time + ';' + ritual + ';' + spell.frontmatter.duration + ';' + concentration + ';' + spell.frontmatter.range + ';' + components + ';' + classes + ';' + '"' + spell.rawContent + '"' + '\n'
      // }
      // console.log(csv)

      // let json = []
      // for (var spell of results) {
      //   let s = {}
      //   s.name = spell.frontmatter.title
      //   s.casting_time = spell.frontmatter.casting_time
      //   s.classes = spell.frontmatter.classes
      //   s.components = spell.frontmatter.components
      //   s.concentration = spell.frontmatter.concentration
      //   s.description = spell.frontmatter.description
      //   s.duration = spell.frontmatter.duration
      //   s.level = spell.frontmatter.level
      //   s.range = spell.frontmatter.range
      //   s.ritual = spell.frontmatter.ritual
      //   s.school = spell.frontmatter.school
      //   s.source = spell.frontmatter.source
      //   s.content = spell.rawContent
      //   json.push(s)
      // }
      //
      // console.log(json)

      // let json = []
      // for (var spell of results) {
      //   let s = {}
      //   s.title = spell.frontmatter.title
      //   let classes = []
      //   const EQUIVALENTS = {
      //     'Barde': 'bard',
      //     'Barbare': 'barbarian',
      //     'Clerc': 'cleric',
      //     'Druide': 'druid',
      //     'Guerrier': 'figther',
      //     'Moine': 'monk',
      //     'Paladin': 'paladin',
      //     'Rôdeur': 'ranger',
      //     'Roublard': 'rogue',
      //     'Ensorceleur/Sorcelame': 'sorcerer',
      //     'Magicien': 'wizard',
      //     'Sorcier': 'warlock',
      //     'Ombrelame': 'shadowblade',
      //   }
      //   for (var classe of spell.frontmatter.classes) {
      //     if (EQUIVALENTS[classe]) {
      //       classes.push(EQUIVALENTS[classe])
      //     }
      //   }
      //   s.classes = classes
      //   s.slug = spell.path.split('/')[2]
      //   json.push(s)
      // }
      // console.log(json)

      return results
    }
  },

  watch: {
    spells: function (newSpells, oldSpells) {
      let self = this
      setTimeout(function () {
        handleTooltips({pages:self.$site.pages})
      }, 100);
    }
  },

  methods: {
    isSpellInSpellBook (spell) {
      return isResourceInLibrary(spell, this.$store.state.mySpells.spells)
    },
    toggleSpellInSpellBook (spell) {
      if (this.isSpellInSpellBook(spell)) {
        this.$store.commit('mySpells/removeSpell', spell)
        this.$store.commit('setSnackbarText', "Le sort " + spell.title + " a été supprimé de votre grimoire")
        this.$store.commit('setIsOpenSnackbar', true)
      } else {
        this.$store.commit('mySpells/addSpell', spell)
        this.$store.commit('setSnackbarText', "Le sort " + spell.title + " a été ajouté à votre grimoire")
        this.$store.commit('setIsOpenSnackbar', true)
      }
    },

    selectItemPerPage (value) {
      setUrlParams("lignes", [value])
      let self = this
      setTimeout(function () {
        handleTooltips({pages:self.$site.pages})
      }, 100);
    },

    changePage (page) {
      // console.log(page)
      setUrlParams("page", [page])
      let self = this
      setTimeout(function () {
        handleTooltips({pages:self.$site.pages})
      }, 100);
    },

    onClickRow (row, item) {
      item.expand(!item.isExpanded)
    },

    levelDisplay (level) {
      if (level.level == 0) {
        return 'Tour de magie'
      } else if (level.level == 1) {
        return level.level.toString() + '<sup>er</sup>'
      }
      return level.level.toString() + '<sup>ème</sup>'
    },

    setShowColumn () {
      Cookies.set('5e-drs-grimoire-colonnes', this.showColumn, { expires: 365 })
    },
  },

  mounted () {
    this.$store.commit('setDrawer', this.$vuetify.breakpoint.lgAndUp)
    this.$store.commit('setHasRightDrawer', true)
    this.$store.commit('setRightDrawer', this.$vuetify.breakpoint.lgAndUp)
    this.$store.commit('setInRightDrawer', 'spellFilters')

    let itemsPerPage = parseInt(getUrlParameter(window.location.href, "lignes"))
    if (itemsPerPage) {
      this.itemsPerPage = itemsPerPage
    }
    let page = parseInt(getUrlParameter(window.location.href, "page"))
    if (page) {
      this.page = page
    }

    // Choix des colonnes à afficher
    const showColumn = Cookies.get('5e-drs-grimoire-colonnes')
    if (showColumn) {
      this.showColumn = JSON.parse(showColumn)
    }

    let self = this
    setTimeout(function () {
      handleTooltips({pages:self.$site.pages})
    }, 100);
    this.$router.afterEach(() => {
      setTimeout(function () {
        handleTooltips({pages:self.$site.pages})
      }, 100)
    })
  }
}
</script>

<style lang="scss">

</style>
