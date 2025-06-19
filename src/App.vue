<script setup>
// Reactive state for calculator logic
import { ref, onMounted, onUnmounted } from 'vue'

const display = ref('0');
const previousValue = ref(null);
const operator = ref(null);
const operatorClicked = ref(false);

// Convert formatted display value to a raw numeric string using '.' as decimal for JS
const toRawNumberString = (val) => {
  // Remove thousands dots and replace decimal comma with dot
  return val.replace(/\./g, '').replace(',', '.');
};

// Format a raw numeric string (with '.' decimal) into display format ('.' thousands, ',' decimal)
// This is lenient and does NOT round—used while user is typing.
const formatNumber = (val) => {
  if (val === '' || isNaN(Number(val))) return val;
  const [intPart, decPart] = val.split('.');
  const formattedInt = intPart.replace(/\B(?=(\d{3})+(?!\d))/g, '.');
  return decPart !== undefined && decPart !== ''
    ? `${formattedInt},${decPart}`
    : formattedInt + (val.endsWith('.') ? ',' : ''); // preserve trailing comma if user typed decimal
};

// Format final calculation result to fit exactly 9 digits (digits before + after) using rounding.
const formatResult = (num) => {
  if (isNaN(num)) return 'Error';

  const isNegative = num < 0;
  const absNum = Math.abs(num);
  const intPartStr = Math.floor(absNum).toString();
  const digitsBefore = intPartStr.length;

  // If integer part already exceeds 9 digits, truncate to first 9 digits (keep sign).
  if (digitsBefore >= 9) {
    const truncated = intPartStr.slice(0, 9);
    const formattedInt = truncated.replace(/\B(?=(\d{3})+(?!\d))/g, '.');
    return (isNegative ? '-' : '') + formattedInt;
  }

  const allowedDecimals = 9 - digitsBefore;

  // Build a string with many decimals, then truncate (not round) to allowed decimals
  const preciseStr = absNum.toFixed(Math.max(allowedDecimals + 5, 6)); // extra precision
  let [iPart, dPart] = preciseStr.split('.');
  dPart = dPart ? dPart.slice(0, allowedDecimals) : '';
  // Remove trailing zeros
  dPart = dPart.replace(/0+$/, '');
  const formattedInt = iPart.replace(/\B(?=(\d{3})+(?!\d))/g, '.');
  const result = dPart && allowedDecimals > 0 ? `${formattedInt},${dPart}` : formattedInt;
  return (isNegative ? '-' : '') + result;
};

/**
 * Handles number button clicks.
 * Appends or replaces the number on the display.
 * @param {string} number - The number that was clicked.
 */
const handleNumber = (number) => {
  let raw = toRawNumberString(display.value);
  if (display.value === '0' || operatorClicked.value) {
    raw = number;
    operatorClicked.value = false;
  } else {
    // Prevent exceeding 9 digits (excluding decimal point)
    const digitsCount = raw.replace('.', '').length;
    if (digitsCount >= 9) return;
    raw += number;
  }
  display.value = formatNumber(raw);
};

/**
 * Handles the decimal point button click.
 */
const handleDecimal = () => {
  const raw = toRawNumberString(display.value);
  if (!raw.includes('.')) {
    display.value = formatNumber(raw + '.');
  }
};

/**
 * Handles operator button clicks (+, -, *, /).
 * @param {string} op - The operator symbol.
 */
const handleOperator = (op) => {
  if (operator.value && previousValue.value !== null) {
    handleEquals();
  }
  previousValue.value = toRawNumberString(display.value);
  operator.value = op;
  operatorClicked.value = true;
};

/**
 * Clears the entire calculator state.
 */
const handleClear = () => {
  display.value = '0';
  previousValue.value = null;
  operator.value = null;
  operatorClicked.value = false;
};

/**
 * Performs the calculation when the equals button is clicked.
 */
const handleEquals = () => {
  if (operator.value === null || previousValue.value === null) {
    return;
  }

  const prev = parseFloat(previousValue.value);
  const current = parseFloat(toRawNumberString(display.value));
  let result = 0;

  switch (operator.value) {
    case '+':
      result = prev + current;
      break;
    case '-':
      result = prev - current;
      break;
    case '*':
      result = prev * current;
      break;
    case '/':
      result = current === 0 ? 'Error' : prev / current;
      break;
  }

  if (result !== 'Error') {
    display.value = formatResult(result);
  } else {
    display.value = result;
  }

  previousValue.value = null;
  operator.value = null;
  operatorClicked.value = true;
};

/**
 * Handles keyboard input for the calculator.
 * Maps key presses to their corresponding button handlers.
 * @param {KeyboardEvent} e - The keyboard event.
 */
const handleKeyDown = (e) => {
  const key = e.key;

  if (key >= '0' && key <= '9') {
    handleNumber(key);
    e.preventDefault();
  } else if (key === '.' || key === ',') {
    handleDecimal();
    e.preventDefault();
  } else if (["+", "-", "*", "/"].includes(key)) {
    handleOperator(key);
    e.preventDefault();
  } else if (key === 'x' || key === 'X') {
    handleOperator('*');
    e.preventDefault();
  } else if (key === 'Enter' || key === '=') {
    handleEquals();
    e.preventDefault();
  } else if (key === 'Escape' || key === 'Backspace' || key === 'c' || key === 'C') {
    handleClear();
    e.preventDefault();
  }
};

// Register and clean up keyboard event listeners
onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
});
</script>

<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-100">
    <div class="bg-white max-w-sm w-full mx-35 rounded-3xl shadow-lg p-6 space-y-2">
      <!-- Calculator Display -->
      <div class="bg-gray-800 text-white text-right rounded-lg p-4 h-36 flex flex-col justify-end items-end space-y-1">
        <!-- Previous value and operator -->
        <span v-if="previousValue && operator" class="text-xl font-light opacity-70 break-all">{{ previousValue }} {{ operator }}</span>
        <!-- Current display value -->
        <h1 class="text-5xl font-light break-all" style="line-height: 1.2;">{{ display }}</h1>
      </div>

      <!-- Calculator Buttons -->
      <div class="calculator-grid">
        <!-- Row 1 -->
        <button @click="handleOperator('+')" class="btn btn-dark-gray">+</button>
        <button @click="handleOperator('-')" class="btn btn-dark-gray">-</button>
        <button @click="handleOperator('*')" class="btn btn-dark-gray">×</button>
        <button @click="handleOperator('/')" class="btn btn-dark-gray">÷</button>
      
        <!-- Row 2 -->   
        <button @click="handleNumber('7')" class="btn btn-dark-gray">7</button>
        <button @click="handleNumber('8')" class="btn btn-dark-gray">8</button>
        <button @click="handleNumber('9')" class="btn btn-dark-gray">9</button>
        <!-- Equals button spanning two rows -->
        <button @click="handleEquals" class="btn btn-orange btn-equals">=</button>

        <!-- Row 3 -->
        <button @click="handleNumber('4')" class="btn btn-dark-gray">4</button>
        <button @click="handleNumber('5')" class="btn btn-dark-gray">5</button>
        <button @click="handleNumber('6')" class="btn btn-dark-gray">6</button>

        <!-- Row 4 -->
        <button @click="handleNumber('1')" class="btn btn-dark-gray">1</button>
        <button @click="handleNumber('2')" class="btn btn-dark-gray">2</button>
        <button @click="handleNumber('3')" class="btn btn-dark-gray">3</button>

        <!-- Row 5 -->
        <button @click="handleNumber('0')" class="btn btn-dark-gray">0</button>
        <button @click="handleDecimal" class="btn btn-dark-gray">,</button>
        <button @click="handleClear" class="btn btn-gray">AC</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
