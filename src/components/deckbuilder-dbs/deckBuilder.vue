<template>
  <div id="deck-builder">
    <!-- #region Filters -->
    <header id="filters-header">
      <!-- #region Leaders Filters -->
      <input
        type="button"
        value="Leader"
        :class="filters.card_type === 'Leader' ? 'active' : ''"
        @click="resetFilters(), (filters.card_type = 'Leader')"
      />
      <!-- #endregion -->

      <!-- #region Cards Filters -->
      <input
        type="button"
        value="Cards"
        :class="filters.card_type === 'Cards' ? 'active' : ''"
        @click="resetFilters(), (filters.card_type = 'Cards')"
      />
      <!-- #endregion -->

      <input type="text" v-model="filters.name" placeholder="Goku" />
<br>

      <!-- #region Sets Filters -->
      <select v-model="filters.sets">
        <option>Any Sets</option>
        <option v-for="set of cardsList.sets" :key="set">
          {{ set }}
        </option>
      </select>
      <!-- #endregion -->

      <!-- #region Color Filters -->
      <select v-model="filters.color">
        <option>Any Color</option>
        <option v-for="color of cardsList.colors" :key="color">
          {{ color }}
        </option>
      </select>
      <!-- #endregion -->
<br>

      <!-- #region Trait Filters -->
      <select v-model="filters.trait">
        <option>Any Special Trait</option>
        <option v-for="trait of cardsList.special_trait" :key="trait">
          {{ trait }}
        </option>
      </select>
      <!-- #endregion -->

      <!-- #region Era Filters -->
      <select v-model="filters.era">
        <option>Any Era</option>
        <option v-for="era of cardsList.era" :key="era">
          {{ era }}
        </option>
      </select>
      <!-- #endregion -->
<br>
      <!-- #region Power Filters -->
      <select v-model="filters.power" v-if="filters.card_type == 'Cards'">
        <option>Any Power</option>
        <option v-for="power of cardsList.power" :key="power">
          {{ power }}
        </option>
      </select>
      <!-- #endregion -->

      <!-- #region Power Filters -->
      <select v-model="filters.rarity" v-if="filters.card_type == 'Cards'">
        <option>Any Rarity</option>
        <option v-for="rarity of cardsList.rarities" :key="rarity">
          {{ rarity }}
        </option>
      </select>
      <!-- #endregion -->

      <!-- #region Energy Cost Filters -->
      <select v-model="filters.energy" v-if="filters.card_type == 'Cards'">
        <option>Any Energy</option>
        <option v-for="energy of cardsList.energy" :key="energy">
          {{ energy }}
        </option>
      </select>
      <!-- #endregion -->

      <!-- #region Combo Power Filters -->
      <select v-model="filters.combo_power" v-if="filters.card_type == 'Cards'">
        <option>Any Combo Power</option>
        <option
          v-for="combo_power of cardsList.combo_power"
          :key="combo_power"
        >
          {{ combo_power }}
        </option>
      </select>
      <!-- #endregion -->
      <input type="button" value="Clear All" @click="resetFilters()" />
    </header>

    <!-- #region Cards List -->
    <div id="card-list">
      <div>
        <select v-model="filters.sortBy">
          <option value="Name">Card Name: A to Z</option>
          <option value="NameDesc">Card Name: Z to A</option>
          <option v-if="filters.card_type == 'Cards'" value="Energy">
            Energy Cost: low to high
          </option>
          <option v-if="filters.card_type == 'Cards'" value="EnergyDesc">
            Energy Cost: high to low
          </option>
          <option v-if="filters.card_type == 'Cards'" value="Power">
            Power: low to high
          </option>
          <option v-if="filters.card_type == 'Cards'" value="PowerDesc">
            Power: high to low
          </option>
        </select>
      </div>

      <div
        id="card"
        v-for="(card, idx) in filterBy"
        v-bind:key="idx"
        @click.prevent="addToDeckList(card)"
      >
        <img :src="card.illustration" :alt="card.illustration" />
      </div>
    </div>
    <!-- #endregion -->

    <!-- #region Deck List -->
    <div id="deck-list">
      <header id="deck-list-header">
        <div :id="deckList.leader.color"></div>
        <span>{{ deckList.leader.name }}</span>
        <span> {{ deckCounter.totalCards }} </span>
      </header>
      <div id="deck-list-cards">
        <div id="deck-list-card" v-for="(card, i) in deckList.cards" :key="i">
          <span> {{ card.name }} {{ card.count }} </span>
          <input type="button" value="+" @click="addToDeckList(card)" />
          <input type="button" value="-" @click="removeCard(card)" />
        </div>
      </div>
    </div>
    <!-- #endregion -->
  </div>
</template>

<script>
//#region Import
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
//#endregion

export default {
  data() {
    return {
      active: true,
      cardsList: [],
      deckList: {
        leader: {
          name: "",
          color: "",
        },
        cards: [],
      },
      filters: {
        name: "",
        trait: "Any Special Trait",
        card_type: "Leader",
        color: "Any Color",
        sets: "Any Sets",
        energy: "Any Energy",
        power: "Any Power",
        combo_power: "Any Combo Power",
        rarity: "Any Rarity",
        era: "Any Era",
        sortBy: "Name",
      },
    };
  },
  //#region Computed
  computed: {
    deckCounter() {
      let cardsCounter = 0;
      let superComboCounter = 0;
      let secretRareCounter = 0;

      for (const card of this.deckList.cards) {
        cardsCounter += card.count;
        if (card.rarity === "Secret Rare") secretRareCounter++;
        if (card.description.includes("[Super Combo]"))
          superComboCounter += card.count;
      }

      let obj = {
        totalCards: cardsCounter,
        superCombo: superComboCounter,
        secretRare: secretRareCounter,
      };
      return obj;
    },
    filterBy() {
      let filtered = [];
      if (this.cardsList.cards) {
        let filters = this.filters;
        console.log(parseInt(filters.energy));
        //#region Cards Filters
        for (const card of this.cardsList.cards) {
          if (filters.sets == "Any Sets" || filters.sets == card.sets) {
            if (filters.era == "Any Era" || filters.era == card.era) {
              if (
                card.name.toLowerCase().includes(filters.name.toLowerCase())
              ) {
                if (
                  filters.power == "Any Power" ||
                  filters.power == card.power
                ) {
                  if (
                    filters.color == "Any Color" ||
                    filters.color == card.color
                  ) {
                    if (
                      filters.energy == "Any Energy" ||
                      parseInt(filters.energy) == parseInt(card.energy)
                    ) {
                      if (
                        filters.rarity == "Any Rarity" ||
                        filters.rarity == card.rarity
                      ) {
                        if (
                          filters.combo_power == "Any Combo Power" ||
                          filters.combo_power == card.combo_power
                        ) {
                          if (
                            card.card_type == "Leader" &&
                            filters.card_type == "Leader"
                          ) {
                            filtered.push(card);
                          } else if (
                            filters.card_type != "Leader" &&
                            card.card_type != "Leader"
                          ) {
                            filtered.push(card);
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
        //#endregion

        //#region SortBy
        if (filters.sortBy == "Name") {
          filtered.sort((a, b) => {
            if (a.name < b.name) return -1;
            return a.name > b.name ? 1 : 0;
          });
        } else if (filters.sortBy == "NameDesc") {
          filtered.sort((a, b) => {
            if (a.name > b.name) return -1;
            return a.name < b.name ? 1 : 0;
          });
        } else if (filters.sortBy == "Energy") {
          filtered.sort((a, b) => {
            if (a.energy < b.energy) return -1;
            return a.energy > b.energy ? 1 : 0;
          });
        } else if (filters.sortBy == "EnergyDesc") {
          filtered.sort((a, b) => {
            if (a.energy > b.energy) return -1;
            return a.energy < b.energy ? 1 : 0;
          });
        } else if (filters.sortBy == "Power") {
          filtered.sort((a, b) => {
            if (parseInt(a.power) < parseInt(b.power)) return -1;
            return a.power > b.power ? 1 : 0;
          });
        } else if (filters.sortBy == "PowerDesc") {
          filtered.sort((a, b) => {
            if (parseInt(a.power) > parseInt(b.power)) return -1;
            return a.power < b.power ? 1 : 0;
          });
        }
        //#endregion
      }
      return filtered;
    },
  },
  //#endregion

  //#region Methods
  methods: {
    getCards() {
      Vue.use(VueAxios, axios);
      this.axios.get("https://localhost:44396/api/DBS").then((response) => {
        this.cardsList = response.data;
      });
    },
    addToDeckList(object) {
      if (this.filters.card_type == "Leader") {
        this.resetFilters();
        this.filters.card_type = "Cards";
        this.deckList.leader = object;
      } else {
        let isSuperCombo = object.description.includes("[Super Combo]");

        if (
          this.deckCounter.totalCards < 60 &&
          (object.rarity != "Secret Rare" || this.deckCounter.secretRare < 1) &&
          (!isSuperCombo || this.deckCounter.superCombo < 4)
        ) {
          for (const card of this.deckList.cards) {
            if (card.name === object.name) {
              if (card.count < 4) {
                return card.count++;
              } else return null;
            }
          }

          let newCards = {
            id: object.id,
            name: object.name,
            card_type: object.card_type,
            color: object.color,
            description: object.description,
            power: object.power,
            energy: object.energy,
            comboCost: object.comboCost,
            comboPower: object.comboPower,
            character: object.character,
            era: object.era,
            sets: object.sets,
            specialTrait: object.specialTrait,
            rarity: object.rarity,
            illustration: object.illustration,
            count: 1,
          };
          this.deckList.cards.push(newCards);

          return this.deckList.cards.sort((a, b) => {
            if (a.name < b.name) return -1;
            return a.name > b.name ? 1 : 0;
          });
        }
      }
    },
    removeCard(object) {
      let i = 0;
      for (const card of this.deckList.cards) {
        i++;
        if (card.name === object.name) {
          if (card.count > 1) {
            return card.count--;
          } else return this.deckList.cards.splice(i - 1, 1);
        }
      }
    },
    resetFilters() {
      let filters = {
        name: "",
        trait: "Any Special Trait",
        card_type: this.filters.card_type,
        color: "Any Color",
        sets: "Any Sets",
        energy: "Any Energy",
        power: "Any Power",
        combo_power: "Any Combo Power",
        rarity: "Any Rarity",
        era: "Any Era",
        sortBy: "Name",
      };

      return (this.filters = filters);
    },
  },
  //#endregion
  mounted() {
    this.getCards();
  },
};
</script>

<style src="./deckBuilder.css"></style>