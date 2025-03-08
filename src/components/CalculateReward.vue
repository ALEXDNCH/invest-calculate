<template>
  <div class="bg-gray-100 min-h-screen flex justify-center items-center p-6">
    <div class="w-2xl p-6 bg-white rounded-lg shadow-lg">
      <h1 class="text-2xl font-bold text-center">Калькулятор средней цены и выхода из позиции</h1>

      <div class="mb-4 mt-10">
        <label for="tokenName" class="block font-medium mb-1"
          >Выберите или создайте новый токен:</label
        >
        <select
          v-model="selectedToken"
          @change="loadTokenData"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
        >
          <option value="new">Создать новый токен</option>
          <option
            v-for="(token, index) in savedTokens"
            :key="index"
            :value="index"
          >
            {{ token.name }}
          </option>
        </select>
      </div>

      <div class="mb-4">
        <label for="tokenNameInput" class="block font-medium mb-1"
          >Название токена:</label
        >
        <input
          v-model="currentTokenName"
          type="text"
          id="tokenNameInput"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
          placeholder="Введите название токена"
        />
      </div>

    
      <div class="flex gap-4 mt-6">
        <div class="w-1/2 flex flex-col justify-between">
          <label for="currentInvested" class="block font-medium mb-1"
            >Общие инвестиции ($):</label
          >
          <input
            v-model.number="currentInvested"
            type="number"
            id="currentInvested"
            required
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="Сколько вы уже инвестировали"
          />
        </div>
        <div class="w-1/2 flex flex-col justify-between">
          <label for="currentTokens" class="block font-medium mb-1"
            >Текущее количество монет*:</label
          >
          <input
            v-model.number="currentTokens"
            type="number"
            id="currentTokens"
            required
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="Сколько монет у вас сейчас"
          />
        </div>
      </div>

    
      <div class="flex gap-4 mt-6">
        <div class="w-1/2 flex flex-col justify-between">
          <label for="buyPrice" class="block font-medium mb-1"
            >Новая цена покупки ($):</label
          >
          <input
            v-model.number="buyPrice"
            type="number"
            id="buyPrice"
            required
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="По какой цене вы хотите докупить"
          />
        </div>
        <div class="w-1/2 flex flex-col justify-between">
          <label for="positionDollars" class="block font-medium mb-1"
            >Сумма для входа в позицию ($):</label
          >
          <input
            v-model.number="positionDollars"
            @input="updateTokensFromDollars"
            type="number"
            id="positionDollars"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="Сколько долларов вы планируете инвестировать"
          />
        </div>
      </div>

      <div class="flex gap-4 mt-6">
        <div class="w-1/2 flex flex-col justify-between">
          <label for="tokensToBuy" class="block font-medium mb-1"
            >Количество монет для докупки:</label
          >
          <input
            v-model.number="tokensToBuy"
            @input="updateDollarsFromTokens"
            type="number"
            id="tokensToBuy"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="Сколько монет вы планируете купить"
          />
        </div>
        <div class="w-1/2 flex flex-col justify-between">
          <label for="currentPrice" class="block font-medium mb-1"
            >Текущая цена ($):</label
          >
          <input
            v-model.number="buyPrice"
            type="number"
            id="currentPrice"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="Текущая рыночная цена"
          />
        </div>
      </div>

      <div class="mt-6">
        <h2 class="text-xl font-medium mb-2">Результаты:</h2>
        <p>Всего куплено монет: {{ allTokens }}</p>
        <p>Всего потрачено $: {{ totalInvested }}</p>
        <p class="text-green-500 font-semibold text-lg">
          {{ result }}
        </p>
        <p class="text-blue-500 font-medium mt-2">
          Текущий профит: {{ currentProfitLossPercent }}%
        </p>
        <p class="text-blue-500 font-medium">
          Текущий профит: ${{ currentProfitLossDollars }}
        </p>
        <p class="text-purple-500 font-medium mt-2">
          Профит после усреднения: {{ profitLossAfterAveragingPercent }}%
        </p>
        <p class="text-purple-500 font-medium">
          Профит после усреднения: ${{ profitLossAfterAveragingDollars }}
        </p>
      </div>

      <div class="flex justify-between mt-6">
        <button
          @click="saveTokenData"
          class="bg-green-500 text-white font-medium py-2 px-4 rounded-md hover:bg-green-600 transition duration-300"
        >
          Сохранить токен
        </button>
        <button
          v-if="!isNewToken && savedTokens.length > 0"
          @click="deleteToken"
          class="bg-red-500 text-white font-medium py-2 px-4 rounded-md hover:bg-red-600 transition duration-300"
        >
          Удалить токен
        </button>
      </div>

      <div class="mt-6">
        <h2 class="text-xl font-medium mb-2">Расчет выхода из позиции:</h2>
        <div class="mb-4">
          <label for="targets" class="block font-medium mb-1"
            >Целевые цены за 1 токен (через запятую):</label
          >
          <input
            v-model="targets"
            type="text"
            id="targets"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="Введите целевые цены за 1 токен"
          />
        </div>
        <div class="mb-4">
          <label for="fixedValuePercents" class="block font-medium mb-1"
            >Проценты продажи токенов (через запятую):</label
          >
          <input
            v-model="fixedValuePercents"
            type="text"
            id="fixedValuePercents"
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-200"
            placeholder="Введите проценты продажи"
          />
        </div>
        <button
          @click="calculateCryptoProfit"
          class="bg-blue-500 text-white font-medium py-2 px-4 rounded-md hover:bg-blue-600 transition duration-300"
        >
          Рассчитать прибыль
        </button>
        <div class="mt-4">
          <h3 class="text-lg font-medium">Результаты:</h3>
          <ul>
            <li v-for="(result, index) in exitResults" :key="index">{{ result }}</li>
          </ul>
          <p class="text-green-500 font-semibold mt-2">
            Общая прибыль: ${{ totalProfit.toFixed(2) }}
          </p>
          <p class="text-blue-500 font-medium">
            Общий PnL: {{ totalPnL.toFixed(2) }}%
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from "vue";

// Реактивные переменные для калькулятора средней цены
const savedTokens = ref([]);
const selectedToken = ref("new");
const currentTokenName = ref("");
const currentInvested = ref(0);
const currentTokens = ref(0);
const buyPrice = ref(0);
const positionDollars = ref(0);
const tokensToBuy = ref(0);

// Реактивные переменные для расчета выхода из позиции
const targets = ref(0);
const fixedValuePercents = ref("1");
const exitResults = ref([]);
const totalProfit = ref(0);
const totalPnL = ref(0);
const allTokens = computed(() => currentTokens.value + tokensToBuy.value)
const isNewToken = computed(() => selectedToken.value === "new");

// Загрузка данных из localStorage при монтировании компонента
onMounted(() => {
  const storedTokens = JSON.parse(localStorage.getItem("tokens")) || [];
  savedTokens.value = storedTokens;
  selectedToken.value = storedTokens?.[0]?.name;
});

// Сохранение данных токена
const saveTokenData = () => {
  if (!currentTokenName.value.trim()) {
    alert("Введите название токена.");
    return;
  }

  const tokenData = {
    name: currentTokenName.value,
    currentInvested: currentInvested.value,
    currentTokens: currentTokens.value,
    buyPrice: buyPrice.value,
    positionDollars: positionDollars.value,
    tokensToBuy: tokensToBuy.value,
  };

  if (isNewToken.value) {
    savedTokens.value.push(tokenData);
  } else {
    savedTokens.value[selectedToken.value] = tokenData;
  }

  localStorage.setItem("tokens", JSON.stringify(savedTokens.value));
  selectedToken.value = savedTokens.value.length - 1;
};

// Загрузка данных выбранного токена
const loadTokenData = () => {
  if (selectedToken.value === "new") {
    currentTokenName.value = "";
    currentInvested.value = 0;
    currentTokens.value = 0;
    buyPrice.value = 0;
    positionDollars.value = 0;
    tokensToBuy.value = 0;
    return;
  }

  const token = savedTokens.value[selectedToken.value];
  currentTokenName.value = token.name;
  currentInvested.value = token.currentInvested;
  currentTokens.value = token.currentTokens;
  buyPrice.value = token.buyPrice;
  positionDollars.value = token.positionDollars;
  tokensToBuy.value = token.tokensToBuy;
};

// Удаление токена
const deleteToken = () => {
  if (confirm("Вы уверены, что хотите удалить этот токен?")) {
    savedTokens.value.splice(selectedToken.value, 1);
    localStorage.setItem("tokens", JSON.stringify(savedTokens.value));
    selectedToken.value = "new";
  }
};

// Реактивный пересчет монет из долларов
const updateTokensFromDollars = () => {
  if (buyPrice.value > 0 && positionDollars.value >= 0) {
    tokensToBuy.value = positionDollars.value / buyPrice.value;
  }
};

// Реактивный пересчет долларов из монет
const updateDollarsFromTokens = () => {
  if (buyPrice.value > 0 && tokensToBuy.value >= 0) {
    positionDollars.value = tokensToBuy.value * buyPrice.value;
  }
};

// Расчет текущего профита/убытка
const currentProfitLossPercent = computed(() => {
  if (currentTokens.value > 0 && currentInvested.value > 0 && buyPrice.value > 0) {
    const avgPrice = currentInvested.value / currentTokens.value;
    return (((buyPrice.value - avgPrice) / avgPrice) * 100).toFixed(2);
  }
  return "N/A";
});

const currentProfitLossDollars = computed(() => {
  if (currentTokens.value > 0 && currentInvested.value > 0 && buyPrice.value > 0) {
    const currentValue = currentTokens.value * buyPrice.value;
    return (currentValue - currentInvested.value).toFixed(2);
  }
  return "N/A";
});

// Расчет профита/убытка после усреднения
const profitLossAfterAveragingPercent = computed(() => {
  if (
    currentTokens.value > 0 &&
    currentInvested.value > 0 &&
    buyPrice.value > 0 &&
    tokensToBuy.value > 0
  ) {
    const totalInvested = currentInvested.value + positionDollars.value;
    const totalTokens = currentTokens.value + tokensToBuy.value;
    const newAvgPrice = totalInvested / totalTokens;
    return (((buyPrice.value - newAvgPrice) / newAvgPrice) * 100).toFixed(2);
  }
  return "N/A";
});

const profitLossAfterAveragingDollars = computed(() => {
  if (
    currentTokens.value > 0 &&
    currentInvested.value > 0 &&
    buyPrice.value > 0 &&
    tokensToBuy.value > 0
  ) {
    const totalInvested = currentInvested.value + positionDollars.value;
    const totalTokens = currentTokens.value + tokensToBuy.value;
    const newAvgPrice = totalInvested / totalTokens;
    const currentValue = totalTokens * buyPrice.value;
    return (currentValue - totalInvested).toFixed(2);
  }
  return "N/A";
});

// Отслеживание изменений в buyPrice для обновления зависимых значений
watch(buyPrice, () => {
  updateTokensFromDollars();
  updateDollarsFromTokens();
});

const additionalInvestment = computed(() => buyPrice.value * tokensToBuy.value);
const totalInvested = computed(() => currentInvested.value + additionalInvestment.value);
const totalTokens = computed(() => currentTokens.value + tokensToBuy.value);
const avgPrice = computed(() => totalInvested.value / totalTokens.value);

const result = computed(() => `Новая средняя цена: $${avgPrice.value.toFixed(8)}`);

// Функция для расчета прибыли
function calculateCryptoProfit() {
  let remainingTokens = allTokens.value; // Оставшиеся токены
  console.log('remainingTokens', remainingTokens);
  let previousPrice = avgPrice.value; // Предыдущая цена
  console.log('avgPrice', previousPrice);
  let profitSum = 0; // Общая прибыль

  exitResults.value = []; // Очищаем результаты

  parsedTargets.value.forEach((targetPrice, index) => {
    const sellPercent = parsedFixedValuePercents.value[index] || 0; // Процент продажи
    const tokensToSell = remainingTokens * sellPercent; // Количество токенов для продажи
    console.log('tokensToSell', tokensToSell);
    const profit = tokensToSell * targetPrice; // Прибыль от продажи

    // Добавляем результаты в массив
    exitResults.value.push(
      `Цена токена: ${targetPrice.toFixed(4)} USD`
    );
    exitResults.value.push(
      `Прибыль от продажи ${sellPercent * 100}% токенов: $${profit.toFixed(2)}`
    );

    // Обновляем состояние
    profitSum += profit;
    remainingTokens -= tokensToSell;
    previousPrice = targetPrice; // Обновляем предыдущую цену на текущую целевую
  });



  // Общий PnL
  const totalPnLValue = ((profitSum + remainingTokens * previousPrice) / totalInvested.value) * 100;

  console.log('profitSum', profitSum);
  console.log('remainingTokens', remainingTokens);
  console.log('previousPrice', previousPrice);
  console.log('totalInvested', totalInvested.value);

  console.log('totalPnLValue', totalPnLValue);

  // Сохраняем результаты
  totalProfit.value = profitSum;
  totalPnL.value = totalPnLValue;
}

const parsedTargets = computed(() =>
  targets.value
    .split(",")
    .map((item) => parseFloat(item.trim()))
    .filter((item) => !isNaN(item))
);

const parsedFixedValuePercents = computed(() =>
  fixedValuePercents.value
    .split(",")
    .map((item) => parseFloat(item.trim()))
    .filter((item) => !isNaN(item))
);
</script>

<style>
label::after {
  content: "*";
  color: red;
  margin-left: 0.25rem;
}
</style>