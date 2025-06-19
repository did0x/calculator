<script setup>
// Reactive state for calculator logic
import { ref, onMounted, onUnmounted } from 'vue'

const display = ref('0');
const previousValue = ref(null);
const operator = ref(null);
const operatorClicked = ref(false);

/**
 * Handles number button clicks.
 * Appends or replaces the number on the display.
 * @param {string} number - The number that was clicked.
 */
const handleNumber = (number) => {
  if (display.value === '0' || operatorClicked.value) {
    display.value = number;
    operatorClicked.value = false;
  } else {
    // Prevent excessively long numbers
    if (display.value.length >= 9) return;
    display.value += number;
  }
};

/**
 * Handles the decimal point button click.
 */
const handleDecimal = () => {
  if (!display.value.includes('.')) {
    display.value += '.';
  }
};

/**
 * Handles operator button clicks (+, -, *, /).
 * @param {string} op - The operator symbol.
 */
const handleOperator = (op) => {
  // If there's already an operator and a previous value, calculate first
  if (operator.value && previousValue.value !== null) {
    handleEquals();
  }
  previousValue.value = display.value;
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
  const current = parseFloat(display.value);
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
      // Handle division by zero
      if (current === 0) {
        result = 'Error';
      } else {
        result = prev / current;
      }
      break;
  }

  // Format the result to fit the display
  if (result !== 'Error') {
    display.value = String(result).slice(0, 10);
  } else {
    display.value = result;
  }

  // Reset state for next calculation
  previousValue.value = null;
  operator.value = null;
  operatorClicked.value = true; // Allow new number entry after equals
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
  } else if (key === '.') {
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
        <button @click="handleDecimal" class="btn btn-dark-gray">.</button>
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
