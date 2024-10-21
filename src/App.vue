<template>
  <header>
    <h1>Monster Slayer</h1>
  </header>
  <section id="monster" class="container">
    <h2>Monster Health</h2>
    <div class="healthbar">
      <div class="healthbar__value" :style="monsterBarStyles"></div>
    </div>
  </section>
  <section id="player" class="container">
    <h2>Your Health</h2>
    <div class="healthbar">
      <div class="healthbar__value" :style="playerBarStyles"></div>
    </div>
  </section>
  <section class="container" v-if="winner">
    <h2>Game Over!</h2>
    <h3 v-if="winner === 'monster'">You Lost!</h3>
    <h3 v-else-if="winner === 'player'">You Win!</h3>
    <h3 v-else>Its a draw!</h3>
    <button @click="startGame">Start new game!</button>
  </section>
  <section id="controls" v-else>
    <button @click="attackMonster">ATTACK</button>
    <button :disabled="mayUseSpecialAttack" @click="specialAttackMonster">
      SPECIAL ATTACK
    </button>
    <button @click="healPlayer">HEAL</button>
    <button @click="surrender">SURRENDER</button>
  </section>
  <section id="log" class="container">
    <h2>Battle Log</h2>
    <ul>
      <li v-for="logMessage in logMessages">
        <!-- {{ logMessage.actionBy }} - {{ logMessage.actionType }} - {{ logMessage.actionValue }} -->
        <span
          :class="{
            'log--player': logMessage.actionBy === 'Player',
            'log--monster': logMessage.actionBy === 'Monster',
          }"
        >
          {{ logMessage.actionBy === "Player" ? "Player" : "Monster" }}</span
        >
        <span v-if="logMessage.actionType === 'heal'"
          >heals himself for
          <span class="log--heal">{{ logMessage.actionValue }}</span>
        </span>
        <span v-else
          >attacks and heals
          <span class="log--damage">{{ logMessage.actionValue }}</span>
        </span>
      </li>
    </ul>
  </section>
</template>

<script>
function getRandomValue(min, max) {
  return Math.floor(Math.random() * (max - min)) + min;
}
export default {
  name: "app",
  data() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      currentRound: 0,
      winner: null,
      logMessages: [],
    };
  },
  computed: {
    monsterBarStyles() {
      if (this.monsterHealth < 0) {
        return { width: "0%" };
      }
      return { width: this.monsterHealth + "%" };
    },
    playerBarStyles() {
      if (this.playerHealth < 0) {
        return { width: "0%" };
      }
      return { width: this.playerHealth + "%" };
    },
    mayUseSpecialAttack() {
      return this.currentRound % 3 !== 0;
    },
  },
  watch: {
    playerHealth(value) {
      if (value <= 0 && this.monsterHealth <= 0) {
        this.winner = "draw";
      } else if (value <= 0) {
        this.winner = "monster";
      }
    },
    monsterHealth(value) {
      if (value <= 0 && this.playerHealth <= 0) {
        this.winner = "draw";
      } else if (value <= 0) {
        this.winner = "player";
      }
    },
  },
  methods: {
    startGame() {
      this.playerHealth = 100;
      this.monsterHealth = 100;
      this.currentRound = 0;
      this.winner = null;
      this.logMessages = [];
    },
    attackMonster() {
      this.currentRound++;
      const attackValue = getRandomValue(5, 12);
      this.monsterHealth -= attackValue;
      this.addLogMessage("Player", "attack", attackValue);
      this.attackPlayer();
    },
    attackPlayer() {
      const attackValue = getRandomValue(8, 15);
      this.playerHealth -= attackValue;
      this.addLogMessage("Monster", "attack", attackValue);
    },
    specialAttackMonster() {
      this.currentRound++;
      const attackValue = getRandomValue(10, 25);
      this.monsterHealth -= attackValue;
      this.addLogMessage("Player", "special-attack", attackValue);
      this.attackPlayer();
    },
    healPlayer() {
      const healValue = getRandomValue(8, 20);
      if (this.playerHealth + healValue > 100) {
        this.playerHealth = 100;
      } else {
        this.playerHealth += healValue;
      }
      this.addLogMessage("Player", "Heals", healValue);
      this.attackPlayer();
    },
    surrender() {
      this.winner = "monster";
    },
    addLogMessage(who, what, value) {
      this.logMessages.unshift({
        actionBy: who,
        actionType: what,
        actionValue: value,
      });
    },
  },
};
</script>
