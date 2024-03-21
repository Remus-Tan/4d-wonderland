<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import { Bet } from "./types";

const cash = ref();
const initialCash = ref();
const gameStarted = ref(false);
const quitGameCheck = ref(false);

const betNumber = ref("");
const bigAmount = ref(0);
const smallAmount = ref(0);
const iBet = ref(false);

const formInvalid = ref(true);
const betNumberError = ref();
const betAmountError = ref();

const betList = ref<Bet[]>([]);

const winningFirst = ref<string[]>([]);
const winningSecond = ref<string[]>([]);
const winningThird = ref<string[]>([]);
const winningStarter = ref<string[]>([]);
const winningConsolation = ref<string[]>([]);
const winAmount = ref(0);
const winnersArray = ref({
  1: false,
  2: false,
  3: false,
  starter: <number[]>[],
  consolation: <number[]>[],
});

const betPlaced = ref();
const rollingDone = ref(false);
const endRoundCheck = ref(false);

const tab = ref("make");

watch([bigAmount, smallAmount], async () => {
  endRoundCheck.value = false;

  try {
    bigAmount.value = Number(bigAmount.value);
  } catch (error) {
    bigAmount.value = 0;
  }

  try {
    smallAmount.value = Number(smallAmount.value);
  } catch (error) {
    console.log(error);
    smallAmount.value = 0;
  }

  if (bigAmount.value < 0) {
    bigAmount.value = 0;
  }

  if (smallAmount.value < 0) {
    smallAmount.value = 0;
  }

  if (bigAmount.value + smallAmount.value < 1) {
    betAmountError.value = "You need to bet at least $1!";
  } else if (bigAmount.value + smallAmount.value > cash.value) {
    betAmountError.value = "You cannot bet more than your balance!";
  } else {
    betAmountError.value = "";
  }
});

watch([betNumber, iBet], async ([newValue], [oldValue]) => {
  const re = /^[0-9]{4}$/;
  betNumber.value = betNumber.value.trim();

  endRoundCheck.value = false;

  if (newValue.length > 4) {
    betNumber.value = oldValue;
  }

  if (!re.test(betNumber.value) || betNumber.value.length != 4) {
    betNumberError.value = "Please enter a valid 4D bet!";
  } else if (
    iBet.value &&
    betNumber.value[0] == betNumber.value[1] &&
    betNumber.value[0] == betNumber.value[2] &&
    betNumber.value[0] == betNumber.value[3]
  ) {
    betNumberError.value = "iBet cannot be placed with 4 of the same digits!";
  } else {
    betNumberError.value = "";
  }
});

watch([betAmountError, betNumberError], async () => {
  if (betAmountError.value == "" && betNumberError.value == "") {
    formInvalid.value = false;
  } else {
    formInvalid.value = true;
  }
});

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

function addBet() {
  cash.value = cash.value - bigAmount.value - smallAmount.value;

  betList.value.push({
    betNumber: betNumber.value,
    bigAmount: bigAmount.value,
    smallAmount: smallAmount.value,
    iBet: iBet.value,
  });
}

function makeBet() {
  betPlaced.value = true;

  // Generate winning numbers
  const totalNumbers = 23;
  const winningNumbers = [];

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

  winningStarter.value[0] = "1234";

  for (let bet of betList.value) {
    checkBet(bet);
  }

  setTimeout(() => {
    rollingDone.value = true;
    let winSound = new Audio("/win.mp3");
    let loseSound = new Audio("/lose.mp3");
    winSound.volume = 0.5;
    loseSound.volume = 0.5;

    if (winAmount.value > 0) {
      winSound.play();
    } else {
      loseSound.play();
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

onMounted(() => {
  cash.value = 50;
});
</script>

<template>
  <main
    class="text-2xl h-screen w-screen max-h-screen max-w-screen overflow-hidden flex justify-center items-center"
  >
    <!-- Header -->
    <div class="hidden xl:block">
      <p v-if="gameStarted" class="absolute left-4 top-4">
        Balance:
        <span
          class="font-xl font-medium text-red-400"
          :class="{ '!text-green-400': cash >= initialCash }"
          >${{ cash }}</span
        >
      </p>
      <button
        @click="quitGameCheck = true"
        class="red-button mb-auto absolute right-4 top-4"
        v-if="gameStarted && !quitGameCheck"
      >
        Quit Game
      </button>
      <p
        class="text-red-500 mb-auto absolute right-4 top-4"
        v-if="quitGameCheck && gameStarted"
      >
        Are you sure?
      </p>
      <button
        v-if="quitGameCheck && gameStarted"
        @click="
          () => {
            // Reset game state
            gameStarted = !gameStarted;
            betNumber = '';
            betNumberError = '';
            bigAmount = 0;
            smallAmount = 0;
            betAmountError = '';
            cash = 50;
            quitGameCheck = false;
          }
        "
        class="red-button mb-auto absolute right-4 top-16"
      >
        Yes
      </button>
      <button
        v-if="quitGameCheck && gameStarted"
        @click="quitGameCheck = false"
        class="secondary-button mb-auto absolute right-24 top-16"
      >
        No
      </button>
    </div>

    <!-- Main -->

    <div class="flex flex-col items-center mx-12">
      <div
        v-if="!gameStarted"
        class="flex flex-col justify-center items-center gap-4"
      >
        <h1 class="font-bold text-center">
          Welcome to
          <span
            class="text-transparent bg-clip-text bg-gradient-to-r from-amber-500 to-pink-500"
            >4D Wonderland</span
          >
        </h1>
        <label class="text-center">Choose your starting cash amount</label>
        <input
          v-model="cash"
          class="w-32 flex-grow-1 text-center"
          type="range"
          min="50"
          max="1000"
          step="50"
        />
        <p class="font-xl font-medium text-green-400">${{ cash }}</p>
        <button
          @click="
            gameStarted = !gameStarted;
            initialCash = cash;
          "
          class="orange-button"
        >
          Start
        </button>
      </div>

      <p v-if="gameStarted" class="xl:hidden">
        Balance:
        <span
          class="font-xl font-medium text-red-400"
          :class="{ '!text-green-400': cash >= initialCash }"
          >${{ cash }}</span
        >
      </p>

      <div
        v-if="gameStarted"
        class="flex justify-evenly w-fit p-1 mb-4 gap-2 bg-stone-200 border rounded-md"
      >
        <label
          for="make-bet"
          class="tab flex p-1 w-36 text-base justify-center rounded-md hover:cursor-pointer has-[:checked]:bg-white has-[:checked]:font-semibold"
          >Make Bet
          <input
            type="radio"
            v-model="tab"
            id="make-bet"
            value="make"
            name="make-tab"
            class="w-20"
          />
        </label>
        <div>
          <label
            for="view-bets"
            class="tab flex p-1 w-36 text-base justify-center rounded-md hover:cursor-pointer has-[:checked]:bg-white has-[:checked]:font-semibold"
            :class="{
              'animate__animated animate__flash animate__infinite animate__slow':
                betList.length > 0,
            }"
            >View Bets ({{ betList.length }})
            <input
              type="radio"
              v-model="tab"
              id="view-bets"
              value="view"
              name="view-tab"
              class="w-20 hidden"
          /></label>
        </div>
      </div>

      <div
        v-if="gameStarted && tab == 'view'"
        class="flex flex-col justify-center items-center gap-4"
      >
        <table class="table-fixed">
          <thead class="bg-stone-200 text-base">
            <tr
              class="*:font-medium *:p-2 *:text-left *:w-24 *:text-stone-700 *:border border-stone-300"
            >
              <th>Number</th>
              <th>Big</th>
              <th>Small</th>
              <th>iBet?</th>
              <th></th>
            </tr>
          </thead>
          <tbody class="bg-stone-100 text-base">
            <tr
              v-for="(bet, index) in betList"
              class="*:p-2 *:text-left *:capitalize border border-stone-300"
            >
              <td>{{ bet.betNumber }}</td>
              <td>${{ bet.bigAmount }}</td>
              <td>${{ bet.smallAmount }}</td>
              <td>{{ bet.iBet ? "✅" : "❌" }}</td>
              <td
                class="hover:bg-slate-400/50 hover:invert hover:cursor-pointer justify-center flex"
                @click="
                  () => {
                    betList.splice(index, 1);
                    cash += bet.smallAmount + bet.bigAmount;
                  }
                "
              >
                <img
                  src="https://cdn-icons-png.flaticon.com/512/3096/3096673.png"
                  alt="Delete"
                  class="w-6"
                />
              </td>
            </tr>
          </tbody>
        </table>

        <button
          class="orange-button orange-button-disabled"
          v-if="betList.length == 0"
          disabled
        >
          You don't have any bets yet!
        </button>
        <button
          class="orange-button"
          v-if="betList.length != 0"
          @click="makeBet"
        >
          Send Bets
        </button>
      </div>

      <div
        v-if="gameStarted && tab == 'make'"
        class="flex flex-col justify-center items-center gap-4"
      >
        <form @submit.prevent="addBet" class="flex flex-col items-center gap-4">
          <input
            v-model="betNumber"
            type="text"
            placeholder="4D"
            class="border-2 rounded-sm p-1 flex w-24 text-center"
          />
          <p v-if="betNumberError" class="form-error text-center">
            {{ betNumberError }}
          </p>
          <div class="border-2 rounded-sm p-1 flex w-72 justify-end">
            <p class="pl-4 text-gray-400 text-right">Big $</p>
            <input
              v-model.number="bigAmount"
              type="number"
              placeholder="Amount"
              class="text-center w-36"
            />
          </div>
          <div class="border-2 rounded-sm p-1 flex w-72 justify-end">
            <p class="pl-4 text-gray-400 text-right">Small $</p>
            <input
              v-model.number="smallAmount"
              type="number"
              placeholder="Amount"
              class="text-center w-36"
            />
          </div>
          <p v-if="betAmountError" class="form-error text-center">
            {{ betAmountError }}
          </p>
          <div class="space-y-4">
            <div class="flex items-center justify-center gap-2">
              <label for="iBet" class="hover:cursor-pointer">iBet</label>
              <input
                type="checkbox"
                name="iBet"
                id="iBet"
                v-model="iBet"
                class="w-6 h-6"
              />
            </div>
          </div>
          <button
            type="submit"
            class="orange-button"
            :class="{
              'orange-button-disabled':
                formInvalid || cash - bigAmount - smallAmount < 0,
            }"
            :disabled="formInvalid || cash - bigAmount - smallAmount < 0"
          >
            Place Bet
          </button>
        </form>
        <a
          class="mt-12 text-lg bg-orange-500 text-white p-1 px-2 rounded-md hover:bg-orange-300 transition-all"
          target="_blank"
          href="https://www.singaporepools.com.sg/en/rules/Pages/4d-game-rules-general.html"
          >4D Game Rules / Prize Table</a
        >
      </div>

      <button
        @click="
          () => {
            // Reset game state
            gameStarted = !gameStarted;
            betNumber = '';
            betNumberError = '';
            bigAmount = 0;
            smallAmount = 0;
            betAmountError = '';
            cash = 50;
          }
        "
        class="red-button xl:hidden mt-20"
        v-if="gameStarted"
      >
        Quit Game
      </button>
    </div>
  </main>

  <div
    class="top-0 left-0 absolute w-screen h-screen transition-colors duration-500 flex flex-col justify-center items-center"
    :class="{
      '!bg-black/80': cash == 0 && endRoundCheck,
      '-translate-x-full': cash != 0 || !endRoundCheck,
    }"
  >
    <button class="red-button !bg-transparent">You ran out of cash!</button>
    <button
      class="red-button mt-4"
      @click="
        () => {
          // Reset game state
          gameStarted = !gameStarted;
          betNumber = '';
          betNumberError = '';
          bigAmount = 0;
          smallAmount = 0;
          betAmountError = '';
          cash = 50;
        }
      "
    >
      Restart 🥲
    </button>
  </div>

  <div
    class="top-0 left-0 absolute w-screen h-screen transition-colors duration-500 flex flex-col justify-center items-center gap-2"
    :class="{ '!bg-black/80': betPlaced, '-translate-x-full': !betPlaced }"
  >
    <img
      src="https://media.tenor.com/H2kPWWXr2uAAAAAi/cute-rolling.gif"
      alt="Rolling Thing"
      class="transition-all duration-500 opacity-0"
      :class="{
        '!opacity-100 w-max h-auto': !rollingDone && betPlaced,
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
        class="transition-colors duration-500 text-black/0"
        :class="{ 'orange-button': betPlaced }"
        @click="
          betPlaced = false;
          rollingDone = false;
          cash += winAmount;
          winAmount = 0;
          winnersArray = {
            1: false,
            2: false,
            3: false,
            starter: <number[]>[],
            consolation: <number[]>[],
          };
          betList = [];
          tab = 'make';
          endRoundCheck = true;
        "
      >
        Done
      </button>
    </div>
  </div>
</template>

<style scoped>
h1 {
  font-size: 1.875rem /* 30px */;
  line-height: 2.25rem /* 36px */;
}

.orange-button {
  background-color: orange;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: large;
  font-weight: 500;
  color: white;
}

.red-button {
  background-color: red;
  padding: 8px 12px;
  border-radius: 6px;
  font-weight: 500;
  color: white;
}

.secondary-button {
  background-color: rgb(218, 218, 218);
  padding: 8px 12px;
  border-radius: 6px;
  font-weight: 500;
  border: 1px solid rgb(195, 195, 195)
}

.orange-button-disabled {
  background-color: rgb(253 186 116);
}

.form-error {
  color: red;
}

.tab input {
  display: none;
}
</style>
