<script setup>
import { ref, computed, onMounted } from 'vue';

const wins = ref(0); // Anzahl Wins
const losses = ref(0); // Anzahl Losses
const draws = ref(0); // Anzahl Unentschieden

const choice = ref(null); //Unsere Auswahl
const computerChoice = ref(null); //Was der Computer wählt
const result = ref(null); // Ob wir gewonnen/verloren/unentschieden haben

const outcomes = {
  // Auswählmöglichkeiten + das Ergebnis gegen den Computer (Stein gewinnt gegen Schere, verliert gegen Papier)
  rock: {
    rock: 'draw',
    paper: 'loss',
    scissors: 'win',
  },
  paper: {
    rock: 'win',
    paper: 'draw',
    scissors: 'loss',
  },
  scissors: {
    rock: 'loss',
    paper: 'win',
    scissors: 'draw',
  },
};

const winPercentage = computed(() => {
  // Berechnet die Chance, gegen den Computer zu gewinnen
  const total = wins.value + draws.value + losses.value;
  return total ? (wins.value / total) * 100 : 0;
});

const play = (c) => {
  choice.value = c;

  const choices = ['rock', 'paper', 'scissors'];
  const random = Math.floor(Math.random() * choices.length);
  computerChoice.value = choices[random];

  const outcome = outcomes[c][computerChoice.value];

  if (outcome === 'win') {
    wins.value++;
    result.value = 'Du hast gewonnen!';
  } else if (outcome === 'draw') {
    draws.value++;
    result.value = 'Es ist unentschieden!';
  } else if (outcome === 'loss') {
    losses.value++;
    result.value = 'Du hast verloren!';
  }

  SaveGame();
};

const SaveGame = () => {
  // Speichert das Spiel sammt Wins, Losses, Draws local ab.
  fetch('https://scherestein-f0c16-default-rtdb.europe-west1.firebasedatabase.app/wins.json', 
  {method: 'POST', headers:{'Content-Type':'application/json'}, body:JSON.stringify({wins:wins.value})})

 //localStorage.setItem('wins', wins.value);
  fetch('https://scherestein-f0c16-default-rtdb.europe-west1.firebasedatabase.app/losses.json',
  {method: 'POST', headers:{'Content-Type':'application/json'}, body:JSON.stringify({losses:losses.value})})

  //localStorage.setItem('losses', losses.value);
  fetch('https://scherestein-f0c16-default-rtdb.europe-west1.firebasedatabase.app/draws.json',
  {method: 'POST', headers:{'Content-Type':'application/json'}, body:JSON.stringify({draws:draws.value})})
  //localStorage.setItem('draws', draws.value);
};

const LoadGame = async () => {
  // Ladet das Spiel vom lokalen Speicher (Alte Wins, Losses, Draws)
  var data = await (await fetch('https://scherestein-f0c16-default-rtdb.europe-west1.firebasedatabase.app/wins.json')).json()
  wins.value = parseInt(data.wins) || 0
  var data = await (await fetch('https://scherestein-f0c16-default-rtdb.europe-west1.firebasedatabase.app/losses.json')).json()
  losses.value = parseInt(data.losses) || 0
  var data = await (await fetch('https://scherestein-f0c16-default-rtdb.europe-west1.firebasedatabase.app/draws.json')).json()
  draws.value = parseInt(data.draws) || 0

  
  //wins.value = parseInt(localStorage.getItem('wins')) || 0;
  //losses.value = parseInt(localStorage.getItem('losses')) || 0;
  //draws.value = parseInt(localStorage.getItem('draws')) || 0;
};

const ResetRound = () => {
  // Resettet die Runde, damit es von Vorne beginnt
  choice.value = null;
  computerChoice.value = null;
  result.value = null;
};

onMounted(() => {
  LoadGame();
  // wenn der buchstabe "r" gedrückt wird, wird die Runde resetted (Das funktioniert durch einen EventListener, der unsere Tastur "belauscht")
  window.addEventListener('keypress', (e) => {
    if (e.key === 'r') {
      ResetRound();
    } else if (e.key === 's') {
      SaveGame();
    }
  });

  // wenn der Buchstabe "s" gedrückt wird, werden die Daten bzw. die Runde gespeichert (Funktioniert mit dem gelichen Event wie bei ResetRound)
});
</script>

<template>
  <div class="allaround">
    <header class="container mx-auto p-6">
      <div class="greeting">
        <h1 class="title">Rock, Paper, Scissors!</h1>
      </div>
      <!--
      <div class="footer">
        <h1>Rock Paper Scissors Game</h1>
      </div>-->
    </header>

    <main class="main">
      <div v-if="choice === null" class="icons">
        <button @click="play('rock')" class="choiceRock">
          <img src="./images/rock.svg" alt="rock" class="bcR" />
        </button>

        <button @click="play('paper')" class="choicePaper">
          <img src="./images/paper.svg" alt="paper" class="bcP" />
        </button>

        <button @click="play('scissors')" class="choiceScissors">
          <img src="./images/scissors.svg" alt="scissors" class="bcS" />
        </button>
      </div>

      <div v-else>
        <div class="result">
          Du hast <span class="choice">{{ choice }}</span> ausgewählt
        </div>

        <div class="compResult">
          Der Computer hat
          <span class="compChoice"> {{ computerChoice }} </span> gewählt
        </div>

        <div class="winner">{{ result }}</div>

        <button @click="ResetRound" class="button">Reset</button>
      </div>

      <div class="stats">{{ wins }} : {{ draws }} : {{ losses }}</div>

      <div class="winrate">Winrate: {{ Math.round(winPercentage) }}%</div>
    </main>
  </div>
</template>

<style>
body,
* {
  background-color: #a0c1a0;
}

h2 {
  color: white;
  font-weight: bold;
  font-size: 30px;
}

.title {
  color: #d3f9d3;
  padding-top: 15px;
  font-weight: bold;
  line-height: 48px;
  font-size: 46px;
}

.allaround {
  text-align: center;
  display: flex;
  flex-direction: column;
}
/*
.main {
  padding: 6px;
  margin: auto;
  flex: 1;
}*/

.greeting {
  text-align: center;
}

.icons {
  padding-top: 100px;
}

.choiceRock {
  background-color: #525251;
  border-radius: 9999px;
  --tw-shadow: 0 10px 15px -3px;
  width: 256px;
  padding: 48px;
  margin-left: 24px;
  margin-right: 24px;
  transition-property: colors;
  transition-duration: 300ms;
}

.choicePaper {
  background-color: #525251;
  border-radius: 9999px;
  --tw-shadow: 0 10px 15px -3px;
  width: 256px;
  padding: 48px;
  margin-left: 24px;
  margin-right: 24px;
  transition-property: colors;
  transition-duration: 300ms;
}

.choiceScissors {
  background-color: #525251;
  border-radius: 9999px;
  --tw-shadow: 0 10px 15px -3px;
  width: 256px;
  padding: 48px;
  margin-left: 24px;
  margin-right: 24px;
  transition-property: colors;
  transition-duration: 300ms;
}

.choiceScissors:hover {
  background-color: #b2f191;
}

.choiceScissors:hover .bcS {
  background-color: #b2f191;
}

.choicePaper:hover {
  background-color: #e6e2d7;
}

.choicePaper:hover .bcP {
  background-color: #e6e2d7;
}

.choiceRock:hover {
  background-color: #878273;
}

.choiceRock:hover .bcR {
  background-color: #878273;
}

.bcR {
  background-color: #525251;
  width: 100%;
}

.bcP {
  background-color: #525251;
  width: 100%;
}

.bcS {
  background-color: #525251;
  width: 100%;
}

.result {
  color: white;
  font-size: 50px;
  padding-top: 50px;
}

.compResult {
  color: white;
  font-size: 50px;
}

.winner {
}

.stats {
}

.winrate {
  font-size: 18px;
}

.button {
  background-color: #64c3b8;
  padding: 10px 30px;
  border-radius: 20%;
}

.footer {
  text-align: center;
  margin-top: 500px;
}
</style>
