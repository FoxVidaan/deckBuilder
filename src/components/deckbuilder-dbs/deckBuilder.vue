<template>
  <div id="deck-builder">
    <!-- #region Filters -->
    <header id="filters-header">
      <input
        type="button"
        value="Leader"
        :class="filters.cardType === 'Leader' ? 'active' : ''"
        @click="resetFilters(), (filters.cardType = 'Leader')"
      />
      <input
        type="button"
        value="Cards"
        :class="filters.cardType === 'Cards' ? 'active' : ''"
        @click="resetFilters(), (filters.cardType = 'Cards')"
      />
      <input type="text" v-model="filters.name" placeholder="Goku" />
      <select v-model="filters.sets">
        <option>Any Sets</option>
        <option v-for="(set, index) in cardsList.sets" :key="index">
          {{ set }}
        </option>
      </select>
      <select v-model="filters.power" v-if="filters.cardType == 'Cards'">
        <option>Any Power</option>
        <option v-for="(set, index) in cardsList.power" :key="index">
          {{ set }}
        </option>
      </select>
      <input type="button" value="Clear All" @click="resetFilters()">
    </header>
    <!-- #endregion -->

    <!-- #region Cards List -->
    <div id="card-list">
      <div>
        <select v-model="filters.sortBy">
          <option value="Name">Card Name: A to Z</option>
          <option value="NameDesc">Card Name: Z to A</option>
          <option v-if="filters.cardType == 'Cards'" value="Energy">
            Energy Cost: low to high
          </option>
          <option v-if="filters.cardType == 'Cards'" value="EnergyDesc">
            Energy Cost: high to low
          </option>
          <option v-if="filters.cardType == 'Cards'" value="Power">
            Power: low to high
          </option>
          <option v-if="filters.cardType == 'Cards'" value="PowerDesc">
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
        cardType: "Leader",
        sets: "Any Sets",
        power: "Any Power",
        rarity: "Any Rarity",
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

      if (this.cardsList.leaders && this.cardsList.cards) {
        let filters = this.filters;

        if (filters.cardType == "Leader") {
          //#region Leaders Filters
          for (const card of this.cardsList.leaders) {
            if (
              (filters.sets == "Any Sets" || filters.sets == card.sets) &&
              card.name.toLowerCase().includes(filters.name.toLowerCase())
            ) {
              filtered.push(card);
            }
          }
          //#endregion
        } else {
          //#region Cards Filters
          for (const card of this.cardsList.cards) {
            if (
              (filters.sets == "Any Sets" || filters.sets == card.sets) &&
              (filters.power == "Any Power" || filters.power == card.power) &&
              card.name.toLowerCase().includes(filters.name.toLowerCase())
            ) {
              filtered.push(card);
            }
          }
          //#endregion
        }

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
      console.log(object.power);
      if (this.filters.cardType == "Leader") {
        this.resetFilters();
        this.filters.cardType = "Cards";
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
            cardType: object.cardType,
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
        cardType: this.filters.cardType,
        sets: "Any Sets",
        power: "Any Power",
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