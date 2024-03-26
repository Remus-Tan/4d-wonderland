<script setup lang="ts">
import { ref, watch } from "vue";
import { Bet } from "../types.ts";
const props = defineProps(["cash", "initialCash", "gameLoopReset"]);
const emit = defineEmits([
  "updateCash",
  "sendBet",
  "gameLoopReset",
  "gameOver",
]);

const tab = ref("make");
const betNumber = ref("");
const bigAmount = ref("");
const smallAmount = ref("");
const iBet = ref(false);

const formInvalid = ref(true);
const betNumberError = ref();
const betAmountError = ref();

const betList = ref<Bet[]>([]);

// Watch props
// When game loop resets, remove existing bets
// If bankrupt, send player back to start
watch(props, async () => {
  if (props.gameLoopReset) {
    if (props.cash == 0) {
      emit("gameOver");
    }
    
    betList.value = [];
    tab.value = "make";
    
    emit("gameLoopReset");
  }

});

watch(bigAmount, async (_, oldBig) => {
  const bigNum = Number(bigAmount.value);
  const smallNum = Number(smallAmount.value);

  bigAmount.value = bigAmount.value.trim();

  if (bigAmount.value[0] == "0" || bigNum < 0 || isNaN(bigNum)) {
    bigAmount.value = oldBig;
  }

  // If both are blank
  // Show error
  if (!bigAmount.value && !smallAmount.value) {
    betAmountError.value = "You must bet at least $1!";
  } else {
    betAmountError.value = "";
  }

  // If both are not blank
  // Check combined value
  if (bigAmount.value && smallAmount.value) {
    if (Number(bigAmount.value) + Number(smallAmount.value) > props.cash) {
      betAmountError.value = "You cannot bet more than your balance!";
    } else {
      betAmountError.value = "";
    }
  } else {
    // If only big is not blank
    // Then check value of big
    if (bigAmount.value) {
      if (bigNum > props.cash) {
        betAmountError.value = "You cannot bet more than your balance!";
      } else {
        betAmountError.value = "";
      }
    }

    // If only small is not blank
    // Then check value of small
    if (smallAmount.value) {
      if (smallNum > props.cash) {
        betAmountError.value = "You cannot bet more than your balance!";
      } else {
        betAmountError.value = "";
      }
    }
  }
});

watch(smallAmount, async (_, oldSmall) => {
  const bigNum = Number(bigAmount.value);
  const smallNum = Number(smallAmount.value);

  smallAmount.value = smallAmount.value.trim();

  if (smallAmount.value[0] == "0" || bigNum < 0 || isNaN(smallNum)) {
    smallAmount.value = oldSmall;
  }

  // If both are blank
  // Show error
  if (!bigAmount.value && !smallAmount.value) {
    betAmountError.value = "You must bet at least $1!";
  } else {
    betAmountError.value = "";
  }

  // If both are not blank
  // Check combined value
  if (bigAmount.value && smallAmount.value) {
    if (Number(bigAmount.value) + Number(smallAmount.value) > props.cash) {
      betAmountError.value = "You cannot bet more than your balance!";
    } else {
      betAmountError.value = "";
    }
  } else {
    // If only big is not blank
    // Then check value of big
    if (bigAmount.value) {
      if (bigNum > props.cash) {
        betAmountError.value = "You cannot bet more than your balance!";
      } else {
        betAmountError.value = "";
      }
    }

    // If only small is not blank
    // Then check value of small
    if (smallAmount.value) {
      if (smallNum > props.cash) {
        betAmountError.value = "You cannot bet more than your balance!";
      } else {
        betAmountError.value = "";
      }
    }
  }
});

watch([betNumber, iBet], async ([newValue], [oldValue]) => {
  const re = /^[0-9]{4}$/;
  betNumber.value = betNumber.value.trim();

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

function addBet() {
  const changeAmount =
    props.cash - Number(bigAmount.value) - Number(smallAmount.value);
  emit("updateCash", changeAmount);

  betList.value.push({
    betNumber: betNumber.value,
    bigAmount: Number(bigAmount.value),
    smallAmount: Number(smallAmount.value),
    iBet: iBet.value,
  });
}
</script>

<template>
  <div class="flex flex-col items-center mx-12">
    <div
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
            'animate__animated animate__flash animate__infinite':
              betList.length > 0 && tab != 'view',
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
      v-if="tab == 'view'"
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
            <th>iBet</th>
            <th></th>
          </tr>
        </thead>
        <tbody class="bg-stone-100 text-base">
          <tr
            v-for="(bet, index) in betList"
            class="*:p-2 *:text-left *:capitalize border border-stone-300"
          >
            <td>{{ bet.betNumber }}</td>
            <td v-if="bet.bigAmount > 0">${{ bet.bigAmount }}</td>
            <td v-else></td>
            <td v-if="bet.smallAmount > 0">${{ bet.smallAmount }}</td>
            <td v-else></td>
            <td>{{ bet.iBet ? "✅" : "❌" }}</td>
            <td
              class="hover:bg-slate-400/50 hover:invert hover:cursor-pointer justify-center flex"
              @click="
                () => {
                  betList.splice(index, 1);
                  emit(
                    'updateCash',
                    props.cash + bet.smallAmount + bet.bigAmount
                  );
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
        @click="emit('sendBet', betList)"
      >
        Send Bets
      </button>
    </div>

    <div
      v-if="tab == 'make'"
      class="flex flex-col justify-center items-center gap-4"
    >
      <form @submit.prevent="addBet" class="flex flex-col items-center gap-4">
        <div class="border-2 rounded-sm p-1 flex w-72 justify-end">
          <p class="px-4 text-gray-400 text-right">4D Entry</p>
          <input
            v-model="betNumber"
            type="text"
            placeholder="0000"
            class="text-center w-36 border bg-gray-100"
            :class="{
              '!text-green-600 bg-green-300':
                !betNumberError && betNumber != '',
              '!text-red-600 bg-red-300 placeholder-red-600': betNumberError,
            }"
          />
        </div>
        <p v-if="betNumberError" class="form-error text-center">
          {{ betNumberError }}
        </p>
        <div class="border-2 rounded-sm p-1 flex w-72 justify-end">
          <p class="px-4 text-gray-400 text-right">Big $</p>
          <input
            v-model="bigAmount"
            class="text-center w-36 border bg-gray-100"
            :class="{
              '!text-red-600 bg-red-300': betAmountError,
            }"
          />
        </div>
        <div class="border-2 rounded-sm p-1 flex w-72 justify-end">
          <p class="px-4 text-gray-400 text-right">Small $</p>
          <input
            v-model="smallAmount"
            class="text-center w-36 border bg-gray-100"
            :class="{
              '!text-red-600 bg-red-300': betAmountError,
            }"
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
          :class="{
            'orange-button': !formInvalid && props.cash > 0,
            'orange-button-disabled':
              formInvalid ||
              props.cash - Number(bigAmount) - Number(smallAmount) < 0,
          }"
          :disabled="
            formInvalid ||
            props.cash - Number(bigAmount) - Number(smallAmount) < 0
          "
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
  </div>
</template>

<style>
.orange-button {
  background-color: orange;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: large;
  font-weight: 500;
  color: white;
}

.orange-button-disabled {
  background-color: rgb(253 186 116);
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

.form-error {
  color: red;
}

.tab input {
  display: none;
}
</style>
