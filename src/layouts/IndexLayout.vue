<template>
  <q-layout class="flex column justify-between items-center text-center">
    <header class=" q-my-lg">
      <div class="limiter flex  items-center"
           :class="{
              'justify-between': role !== 2,
              'justify-center': role === 2,
           }">
        <Logo :style="{'font-size': role === 2 ? '2.5vw': ''}">
          Выборы презедента ЮФМЛ
        </Logo>
        <HeaderNavigation v-if="role !== 2"/>
      </div>
    </header>
    <section class="candidates-wrapper limiter flex column justify-center">
      <div class="candidates flex justify-center">
        <Candidate
          v-for="(candidate, id) in candidates"
          :key="candidate"
          :candidate="candidate"
          :color="candidatesColors[id]"
        />
      </div>
    </section>
    <h2 class="congratulations" v-if="winnerName">Поздравляем
      {{ morphName(name = winnerName[0], surname = winnerName[1], gender = winnerName[2]) }} с победой!</h2>
    <div class="limiter q-my-md">
      <section class="votes flex column justify-center q-px-md" v-if="isVoted || isVoteDisplayShown">
        <VotesDisplay :colors="candidatesColors" :candidates="candidates"/>
      </section>
    </div>

  </q-layout>
</template>

<script>
import Logo from "components/Logo";
import HeaderNavigation from "components/HeaderNavigation";
import Candidate from "components/Candidate";
import VotesDisplay from "components/VotesDisplay";
import axios from "axios";
import {mapGetters,} from 'vuex'
import constants from "src/js/constants";
var petrovich = require('petrovich');

function hsbToHex(h, s, b) {
  b /= 100;
  const a = s * Math.min(b, 1 - b) / 100;
  const f = n => {
    const k = (n + h / 30) % 12;
    const color = b - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
    return Math.round(255 * color).toString(16).padStart(2, '0');   // convert to Hex and prefix "0" if needed
  };
  return `#${f(0)}${f(8)}${f(4)}`;
}

export default {
  username: "IndexLayout",
  components: {
    Logo,
    HeaderNavigation,
    Candidate,
    VotesDisplay,
  },
  mounted() {
    setInterval(() => {
      document.location.reload()
    }, 2000 * 60)
    axios.get(constants.serverIp + 'candidates/').then((req) => {
      this.candidates = req.data
    })
    axios.get(constants.serverIp + 'winner-name/').then(req => {
      this.winnerName = req.data
    })
  },
  data() {
    return {
      candidates: null,
      // candidatesColors: null,
      width: null,
      winnerName: '',
      // role: null,
    }
  },
  methods: {
    // ...mapGetters('mainStore', ['sessionId']),
    generateCandidatesColors(count) {
      let colors = []
      let candidatesCount = count
      for (let hue = 0; hue < candidatesCount; hue++) {
        colors.push(hsbToHex(hue * 360 / candidatesCount, 72, 51))
      }
      return colors
    },
    morphName(name, surname, gender) {
      gender = gender ? 'male' : 'female'
      let person = {
        gender,
        first: name,
        last: surname,
      }
      let morphedData = petrovich(person, 'accusative')
      return morphedData.first + " " + morphedData.last
    }
  },
  computed: {
    ...mapGetters('mainStore', ['sessionId', 'isVoted', 'isVoteDisplayShown', 'isNameShown', 'role']),
    candidatesColors() {
      if (this.candidates) {
        return this.generateCandidatesColors(Object.keys(this.candidates).length)
      }
      return this.generateCandidatesColors(10)
    }
  }

}
</script>

<style lang="scss" scoped>
h2{
  font-size:calc( 20px + 2vw )!important;
}
.layout-wrapper {
  height: 100%;
}

.q-layout {
  min-height: 100vh !important;
}

.limiter {
  flex-grow: 1;

}

header {
  color: black;
  width: 100%;


  .limiter {
    height: 100%;
  }
}

.votes {
  flex-grow: 1;
  //min-height: 100px;
}

.candidate-wrapper {
  width: 100%;

  .candidate-wrapper {
    max-width: 22%;
    margin-left: 1.5%;
    margin-right: 1.5%;
  }
}

@media (max-width: 768px) {
  .candidate-wrapper {
    max-width: 45% !important;
    margin-top: 5%;
    margin-bottom: 5%;
  }
}

@media (max-width: $breakpoint-xs-max) {
  .candidate-wrapper {
    max-width: 100% !important;
    margin-top: 5%;
    margin-bottom: 5%;
  }
}
</style>
