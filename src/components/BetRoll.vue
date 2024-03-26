<script setup lang="ts">
import { onMounted, ref } from "vue";
import { Bet } from "../types";

const props = defineProps(["betAmount", "betList", "withAudio"]);
const emit = defineEmits(["done", "winAmount"]);

const transitionColor = ref(false);
const rollingDone = ref(false);

const winningFirst = ref<string[]>([]);
const winningSecond = ref<string[]>([]);
const winningThird = ref<string[]>([]);
const winningStarter = ref<string[]>([]);
const winningConsolation = ref<string[]>([]);
const winnersArray = ref({
  1: false,
  2: false,
  3: false,
  starter: <number[]>[],
  consolation: <number[]>[],
});
const winAmount = ref(0);

onMounted(() => {
  generateWinners();
  setTimeout(() => {
    transitionColor.value = true;
  }, 10);
});

function finish() {
  transitionColor.value = false;
  setTimeout(() => {
    emit('winAmount', winAmount.value)
  }, 500)
}

function convert4DigitsToMap(input: string) {
  const mapOutput = new Map();

  // create hashmap of digits
  for (let i = 0; i < input.length; i++) {
    const digitToCheck = input[i];
    if (![...mapOutput.keys()].includes(digitToCheck)) {
      mapOutput.set(digitToCheck, 1);
    } else {
      mapOutput.set(digitToCheck, mapOutput.get(digitToCheck) + 1);
    }
  }

  return mapOutput;
}

function generateWinners() {
  // Generate winning numbers
  const totalNumbers = 23;
  const winningNumbers = <String[]>[];

  for (let i = 0; i < totalNumbers; i++) {
    const digit1 = String(Math.floor(Math.random() * 10));
    const digit2 = String(Math.floor(Math.random() * 10));
    const digit3 = String(Math.floor(Math.random() * 10));
    const digit4 = String(Math.floor(Math.random() * 10));

    const combined = `${digit1}${digit2}${digit3}${digit4}`;

    winningNumbers.push(combined);
  }

  const arrayFromWinners = Array.from(winningNumbers) as string[];
  winningFirst.value = arrayFromWinners.slice(0, 1);
  winningSecond.value = arrayFromWinners.slice(1, 2);
  winningThird.value = arrayFromWinners.slice(2, 3);
  winningStarter.value = arrayFromWinners.slice(3, 13);
  winningConsolation.value = arrayFromWinners.slice(13, 23);

  for (let bet of props.betList) {
    checkBet(bet);
  }

  setTimeout(() => {
    rollingDone.value = true;
    let winSound = new Audio("/win.mp3");
    let loseSound = new Audio("/lose.mp3");
    winSound.volume = 0.5;
    loseSound.volume = 0.5;

    if (props.withAudio) {
      if (winAmount.value > 0) {
        winSound.play();
      } else {
        loseSound.play();
      }
    }
  }, 2000);
}

function checkBet(bet: Bet) {
  // normal bet checker
  if (!bet.iBet) {
    for (let [index, prizeCategory] of [
      winningFirst.value,
      winningSecond.value,
      winningThird.value,
      winningStarter.value,
      winningConsolation.value,
    ].entries()) {
      let position = 0;
      for (let winner of prizeCategory) {
        if (winner == bet.betNumber) {
          if (bet.bigAmount > 0) {
            switch (Number(index)) {
              // 1st
              case 0:
                winnersArray.value[1] = true;
                winAmount.value += 2000 * bet.bigAmount;
                break;

              // 2nd
              case 1:
                winnersArray.value[2] = true;
                winAmount.value += 1000 * bet.bigAmount;
                break;

              // 3rd
              case 2:
                winnersArray.value[3] = true;
                winAmount.value += 490 * bet.bigAmount;
                break;

              // Starter
              case 3:
                winnersArray.value["starter"].push(position);
                winAmount.value += 250 * bet.bigAmount;
                break;

              // Consolation
              case 4:
                winnersArray.value["consolation"].push(position);
                winAmount.value += 60 * bet.bigAmount;
                break;
            }
          }

          if (bet.smallAmount > 0) {
            switch (Number(index)) {
              // 1st
              case 0:
                winAmount.value += 3000 * bet.smallAmount;
                break;

              // 2nd
              case 1:
                winAmount.value += 2000 * bet.smallAmount;
                break;

              // 3rd
              case 2:
                winAmount.value += 800 * bet.smallAmount;
                break;
            }
          }
        }
        position++;
      }
    }
  }

  // iBet checker
  if (bet.iBet) {
    const ibetMap = convert4DigitsToMap(bet.betNumber);

    // check same digit for payout
    let repeatDigits = 0;

    const ibetmapIterator = ibetMap.values();
    const firstVal = ibetmapIterator.next().value;
    const secondVal = ibetmapIterator.next().value;
    const thirdVal = ibetmapIterator.next().value;

    if (firstVal == 1 && secondVal == 1 && thirdVal == 1) {
      repeatDigits = 0;
    } else if (
      (firstVal == 1 && secondVal == 2) ||
      (firstVal == 2 && secondVal == 1)
    ) {
      repeatDigits = 1;
    } else if (
      (firstVal == 2 && secondVal == 2) ||
      (firstVal == 1 && secondVal == 1 && thirdVal == 2)
    ) {
      repeatDigits = 2;
    } else {
      repeatDigits = 3;
    }

    // check winnings
    for (let [index, prizeCategory] of [
      winningFirst.value,
      winningSecond.value,
      winningThird.value,
      winningStarter.value,
      winningConsolation.value,
    ].entries()) {
      let position = 0;

      for (let winner of prizeCategory) {
        const winnerMap = convert4DigitsToMap(winner);
        let possibleWinner = true;

        for (let [digit, occurrence] of winnerMap) {
          //
          // Check if digit in winner number exists in bet number
          // Then, check if qty of digit matches
          //
          // If possible winner, check
          // Otherwise, complete for loop without further checking
          //
          if (possibleWinner) {
            if (![...ibetMap.keys()].includes(digit)) {
              possibleWinner = false;
            } else if (!(occurrence == ibetMap.get(digit))) {
              possibleWinner = false;
            }
          }
        }

        if (possibleWinner) {
          if (bet.bigAmount > 0) {
            switch (Number(index)) {
              // 1st
              case 0:
                winnersArray.value[1] = true;
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 83 * bet.bigAmount;
                    break;
                  case 1:
                    winAmount.value += 166 * bet.bigAmount;
                    break;
                  case 2:
                    winAmount.value += 335 * bet.bigAmount;
                    break;
                  case 3:
                    winAmount.value += 500 * bet.bigAmount;
                    break;
                }
                break;

              // 2nd
              case 1:
                winnersArray.value[2] = true;
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 41 * bet.bigAmount;
                    break;
                  case 1:
                    winAmount.value += 83 * bet.bigAmount;
                    break;
                  case 2:
                    winAmount.value += 168 * bet.bigAmount;
                    break;
                  case 3:
                    winAmount.value += 250 * bet.bigAmount;
                    break;
                }
                break;

              // 3rd
              case 2:
                winnersArray.value[3] = true;
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 20 * bet.bigAmount;
                    break;
                  case 1:
                    winAmount.value += 40 * bet.bigAmount;
                    break;
                  case 2:
                    winAmount.value += 85 * bet.bigAmount;
                    break;
                  case 3:
                    winAmount.value += 127 * bet.bigAmount;
                    break;
                }
                break;

              // Starter
              case 3:
                winnersArray.value["starter"].push(Number(position));
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 10 * bet.bigAmount;
                    break;
                  case 1:
                    winAmount.value += 20 * bet.bigAmount;
                    break;
                  case 2:
                    winAmount.value += 41 * bet.bigAmount;
                    break;
                  case 3:
                    winAmount.value += 62 * bet.bigAmount;
                    break;
                }
                break;

              // Consolation
              case 4:
                winnersArray.value["consolation"].push(Number(position));
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 3 * bet.bigAmount;
                    break;
                  case 1:
                    winAmount.value += 6 * bet.bigAmount;
                    break;
                  case 2:
                    winAmount.value += 10 * bet.bigAmount;
                    break;
                  case 3:
                    winAmount.value += 15 * bet.bigAmount;
                    break;
                }
                break;
            }
          }

          if (bet.smallAmount > 0) {
            switch (Number(index)) {
              // 1st
              case 0:
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 125 * bet.smallAmount;
                    break;
                  case 1:
                    winAmount.value += 250 * bet.smallAmount;
                    break;
                  case 2:
                    winAmount.value += 500 * bet.smallAmount;
                    break;
                  case 3:
                    winAmount.value += 750 * bet.smallAmount;
                    break;
                }
                break;

              // 2nd
              case 1:
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 83 * bet.smallAmount;
                    break;
                  case 1:
                    winAmount.value += 167 * bet.smallAmount;
                    break;
                  case 2:
                    winAmount.value += 333 * bet.smallAmount;
                    break;
                  case 3:
                    winAmount.value += 500 * bet.smallAmount;
                    break;
                }
                break;

              // 3rd
              case 2:
                switch (repeatDigits) {
                  case 0:
                    winAmount.value += 33 * bet.smallAmount;
                    break;
                  case 1:
                    winAmount.value += 66 * bet.smallAmount;
                    break;
                  case 2:
                    winAmount.value += 133 * bet.smallAmount;
                    break;
                  case 3:
                    winAmount.value += 200 * bet.smallAmount;
                    break;
                }
                break;
            }
          }
        }

        position++;
      }
    }
  }
}
</script>

<template>
  <div
    class="top-0 left-0 absolute w-screen h-screen transition-all duration-500 flex flex-col justify-center items-center gap-2 bg-black/80"
    :class="{ 'opacity-0': !transitionColor, 'opacity-100': transitionColor }"
  >
    <img
      src="https://media.tenor.com/H2kPWWXr2uAAAAAi/cute-rolling.gif"
      alt="Rolling Thing"
      class="transition-all duration-500 opacity-0"
      :class="{
        '!opacity-100 w-max h-auto': !rollingDone,
        'w-0 h-0': rollingDone,
      }"
    />
    <div
      v-if="rollingDone"
      class="bg-white p-8 rounded-lg text-2xl flex flex-col gap-4 animate__animated animate__flipInY"
    >
      <table class="w-full">
        <div class="bg-gradient-to-r from-rose-400 to-red-500 pb-1">
          <p class="text-3xl font-medium bg-white text-center">First Prize</p>
        </div>
        <tr class="flex justify-around">
          <td
            class="text-center w-full"
            :class="{
              '!text-red-500 !font-semibold !bg-red-100':
                winnersArray[1] == true,
            }"
          >
            {{ winningFirst[0] }}
          </td>
        </tr>
      </table>
      <table class="w-full">
        <div class="bg-gradient-to-r from-fuchsia-500 to-cyan-500 pb-1">
          <p class="text-3xl font-medium bg-white text-center">Second Prize</p>
        </div>
        <tr class="flex justify-around">
          <td
            class="text-center w-full"
            :class="{
              '!text-red-500 !font-semibold !bg-red-100':
                winnersArray[2] == true,
            }"
          >
            {{ winningSecond[0] }}
          </td>
        </tr>
      </table>
      <table class="w-full">
        <div class="bg-gradient-to-r from-emerald-400 to-cyan-400 pb-1">
          <p class="text-3xl font-medium bg-white text-center">Third Prize</p>
        </div>
        <tr class="flex justify-around">
          <td
            class="text-center w-full"
            :class="{
              '!text-red-500 !font-semibold !bg-red-100':
                winnersArray[3] == true,
            }"
          >
            {{ winningThird[0] }}
          </td>
        </tr>
      </table>

      <div class="grid grid-cols-2 gap-2">
        <table class="w-full">
          <p class="text-3xl font-medium border-b-4 border-sky-600 text-center">
            Starter
          </p>
          <tr class="flex justify-around" v-for="n in 5">
            <td
              class="text-center w-full"
              :class="{
                '!text-red-500 !font-semibold !bg-red-100': winnersArray[
                  'starter'
                ].includes(n * 2 - 2),
              }"
            >
              {{ winningStarter[n * 2 - 2] }}
            </td>
            <td
              class="text-center w-full"
              :class="{
                '!text-red-500 !font-semibold !bg-red-100': winnersArray[
                  'starter'
                ].includes(n * 2 - 1),
              }"
            >
              {{ winningStarter[n * 2 - 1] }}
            </td>
          </tr>
        </table>

        <table class="w-full">
          <p
            class="text-3xl font-medium border-b-4 border-emerald-600 text-center"
          >
            Consolation
          </p>
          <tr class="flex justify-around" v-for="n in 5">
            <td
              class="text-center w-full"
              :class="{
                '!text-red-500 !font-semibold !bg-red-100': winnersArray[
                  'consolation'
                ].includes(n * 2 - 2),
              }"
            >
              {{ winningConsolation[n * 2 - 2] }}
            </td>
            <td
              class="text-center w-full"
              :class="{
                '!text-red-500 !font-semibold !bg-red-100': winnersArray[
                  'consolation'
                ].includes(n * 2 - 1),
              }"
            >
              {{ winningConsolation[n * 2 - 1] }}
            </td>
          </tr>
        </table>
      </div>
    </div>

    <div
      v-if="rollingDone"
      class="bg-white p-4 rounded-lg text-2xl flex flex-col gap-4 animate__animated animate__flipInY"
    >
      <p v-if="winAmount > 0">
        You won <span class="text-green-500">${{ winAmount }}</span
        >!
      </p>
      <p v-if="winAmount == 0">You didn't win...</p>
      <button
        type="button"
        class="orange-button"
        @click="finish"
      >
        Done
      </button>
    </div>
  </div>
</template>
