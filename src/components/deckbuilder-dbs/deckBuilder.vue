<template>
  <div id="deck-builder">
    <header id="filters-header">
      <input
        type="button"
        value="Leader"
        :class="leader === true ? 'active' : ''"
        @click="leader = true"
      />
      <input
        type="button"
        value="Cards"
        :class="leader !== true ? 'active' : ''"
        @click="leader = false"
      />
    </header>
    <div id="leader-list" v-if="leader == true">
      <div
        id="card"
        v-for="(leader, index) in cardsList.leaders"
        v-bind:key="index"
        @click.prevent="addLeader(leader)"
      >
        <img :src="leader.illustration" :alt="leader.illustration" />
      </div>
    </div>
    <div id="card-list" v-else>
      <div
        id="card"
        v-for="(card, idx) in cardsList.cards"
        v-bind:key="idx"
        @click.prevent="addToDeckList(card)"
      >
        <img :src="card.illustration" :alt="card.illustration" />
      </div>
    </div>
    <div id="deck-list">
      <header id="deck-list-header">
        <div :id="deckList.leader.color"></div>
        <span>{{ deckList.leader.name }}</span>
        <span> {{ deckCounter.totalCards }} </span>
      </header>
      <div id="deck-list-cards">
        <div
          id="deck-list-card"
          v-for="(card, i) in deckList.cards"
          v-bind:key="i"
        >
          <span> {{ card.name }} {{ card.count }}</span>
          <input type="button" value="+" @click="addToDeckList(card)" />
          <input type="button" value="-" @click="removeCard(card)" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";

export default {
  data() {
    return {
      leader: true,
      active: true,
      cardsList: [],
      deckList: {
        leader: {
          name: "",
          color: "",
        },
        cards: [],
      },
    };
  },
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
  },
  methods: {
    getCards() {
      Vue.use(VueAxios, axios);
      this.axios.get("https://localhost:44396/api/DBS").then((response) => {
        this.cardsList = response.data;
      });
    },
    addToDeckList(object) {
      let isSuperCombo = object.description.includes("[Super Combo]");
      if (
        this.deckCounter.totalCards < 60 &&
        (object.rarity != "Secret Rare" || this.deckCounter.secretRare < 1) &&
        (!isSuperCombo || this.deckCounter.superCombo < 4)
      ) {
        for (const card of this.deckList.cards) {
          if (card.name === object.name) {
            if (card.count < 4) {
              console.log(object.count);
              return card.count++;
            } else return null;
          }
        }
        object.count = 1;
        this.deckList.cards.push(object);
        return this.deckList.cards.sort((a, b) => {
          if (a.name < b.name) return -1;
          return a.name > b.name ? 1 : 0;
        });
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
    addLeader(object) {
      this.leader = false;
      this.deckList.leader = object;
    },
  },
  mounted() {
    this.getCards();
  },
};
</script>

<style src="./deckBuilder.css"></style>