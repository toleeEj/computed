<template>
  <div class="container">
    <h1>Expense Tracker</h1>

    <!-- Expense Form -->
    <form @submit.prevent="addExpense" class="expense-form">
      <input v-model="newExpense.name" placeholder="Expense Name" required />
      <input v-model.number="newExpense.amount" type="number" placeholder="Amount in $" required min="0" />
      <select v-model="newExpense.category" required>
        <option disabled value="">Select Category</option>
        <option>Food</option>
        <option>Transport</option>
        <option>Entertainment</option>
        <option>Other</option>
      </select>
      <button type="submit">Add Expense</button>
    </form>

    <!-- Expense List -->
    <ul class="expense-list">
      <li v-for="(expense, index) in expenses" :key="index">
        {{ expense.name }} - {{ expense.category }} - ${{ expense.amount }}
        <button @click="removeExpense(index)">Remove</button>
      </li>
    </ul>

    <!-- Computed Total Expense -->
    <h2>Total Expense: ${{ totalExpense }}</h2>

    <!-- Watched Food Expense -->
    <p v-if="foodExpenseExceeded" class="warning">Warning: You've spent too much on Food!</p>

    <!-- Category Breakdown -->
    <h2>Expenses by Category:</h2>
    <ul class="category-breakdown">
      <li v-for="(amount, category) in categoryBreakdown" :key="category">
        {{ category }}: ${{ amount }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newExpense: {
        name: '',
        amount: null,
        category: ''
      },
      expenses: JSON.parse(localStorage.getItem('expenses')) || [],
      foodThreshold: 100,
      foodExpenseExceeded: false
    };
  },
  computed: {
    totalExpense() {
      return this.expenses.reduce((sum, expense) => sum + expense.amount, 0);
    },
    totalFoodExpense() {
      return this.expenses
        .filter(expense => expense.category === 'Food')
        .reduce((sum, expense) => sum + expense.amount, 0);
    },
    categoryBreakdown() {
      return this.expenses.reduce((categories, expense) => {
        if (!categories[expense.category]) {
          categories[expense.category] = 0;
        }
        categories[expense.category] += expense.amount;
        return categories;
      }, {});
    }
  },
  watch: {
    totalFoodExpense(newValue) {
      this.foodExpenseExceeded = newValue > this.foodThreshold;
    },
    expenses: {
      handler(newExpenses) {
        localStorage.setItem('expenses', JSON.stringify(newExpenses));
      },
      deep: true
    }
  },
  methods: {
    addExpense() {
      if (this.newExpense.name && this.newExpense.amount && this.newExpense.category) {
        this.expenses.push({ ...this.newExpense });
        this.newExpense.name = '';
        this.newExpense.amount = null;
        this.newExpense.category = '';
      }
    },
    removeExpense(index) {
      this.expenses.splice(index, 1);
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #f4f4f4;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

h1, h2 {
  color: #2c3e50;
  text-align: center;
}

.expense-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

input, select {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 100%;
}

button {
  background-color: #2980b9;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #3498db;
}

.expense-list {
  margin-top: 20px;
  padding: 0;
  list-style: none;
}

.expense-list li {
  background-color: #ecf0f1;
  margin: 10px 0;
  padding: 10px;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
}

.warning {
  color: red;
  text-align: center;
  font-weight: bold;
}

.category-breakdown {
  margin-top: 20px;
  padding: 0;
  list-style: none;
}
</style>