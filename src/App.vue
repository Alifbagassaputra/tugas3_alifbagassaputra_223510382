<template>
  <div id="app">
    <h1>List Harga Barang</h1>
    
    <!-- Form untuk memasukkan pengeluaran baru -->
    <form @submit.prevent="addOrUpdateExpense" class="form">
      <label for="description">Nama barang:</label>
      <input type="text" id="description" v-model="newExpense.description" required>

      <label for="amount">Harga:</label>
      <input type="number" id="amount" v-model.number="newExpense.amount" required>

      <button type="submit">{{ editingIndex !== null ? 'Update' : 'Tambahkan' }}</button>
      <button type="button" @click="cancelEdit" v-show="editingIndex !== null">Batal</button>
    </form>

    <!-- Form untuk filter -->
    <div class="filter">
      <label for="search">Cari Barang:</label>
      <input type="text" id="search" v-model="searchKeyword">
    </div>

    <!-- Filter buttons -->
    <div class="filter-buttons">
      <button @click="showAll" :class="{ 'active': currentFilter === 'All' }">Semua</button>
      <button @click="showActive" :class="{ 'active': currentFilter === 'Active' }">Aktif</button>
      <button @click="showCompleted" :class="{ 'active': currentFilter === 'Completed' }">Completed</button>
    </div>

    <!-- Daftar pengeluaran -->
    <div class="expenses">
      <h2>List</h2>
      <ul>
        <li v-for="(expense, index) in filteredExpenses" :key="index" :class="{ 'highlight': expense.amount > limit, 'checked': expense.checked }">
          <input type="checkbox" v-model="expense.checked"> <!-- Tambahkan checkbox -->
          <span class="description" :class="{ 'checked': expense.checked }">{{ expense.description }}</span> 
          <span class="amount" :class="{ 'checked': expense.checked }">{{ formatRupiah(expense.amount) }}</span>
          <div class="action-buttons">
            <button @click="editExpense(index)" class="edit-button">Edit</button> <!-- Tambahkan tombol Edit -->
            <button @click="removeExpense(index)" class="delete-button">Hapus</button> <!-- Tambahkan tombol Hapus -->
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      newExpense: {
        description: '',
        amount: ''
      },
      expenses: [],
      limit: 50000, // Batas pengeluaran untuk penyorotan
      searchKeyword: '', // Kata kunci pencarian
      editingIndex: null, // Indeks item yang sedang diedit
      currentFilter: 'All' // Filter saat ini ('All', 'Active', 'Completed')
    };
  },
  created() {
    // Memeriksa apakah ada data yang tersimpan di local storage saat aplikasi dimuat
    if (localStorage.getItem('expenses')) {
      this.expenses = JSON.parse(localStorage.getItem('expenses'));
    }
  },
  computed: {
    filteredExpenses() {
      if (this.currentFilter === 'Completed') {
        return this.expenses.filter(expense => expense.checked);
      } else if (this.currentFilter === 'Active') {
        return this.expenses.filter(expense => !expense.checked);
      } else {
        if (!this.searchKeyword) {
          return this.expenses;
        }
        const keyword = this.searchKeyword.toLowerCase();
        return this.expenses.filter(expense =>
          expense.description.toLowerCase().includes(keyword)
        );
      }
    }
  },
  methods: {
    addOrUpdateExpense() {
      if (this.editingIndex !== null) {
        // Jika sedang dalam mode edit, update item yang sedang diedit
        this.expenses[this.editingIndex] = {...this.newExpense, checked: false};
        this.editingIndex = null; // Keluar dari mode edit setelah proses update
      } else {
        // Jika tidak dalam mode edit, tambahkan item baru
        this.expenses.push({...this.newExpense, checked: false});
      }
      this.saveToLocalStorage(); // Menyimpan data ke local storage setelah menambah atau mengedit pengeluaran
      this.resetForm();
    },
    editExpense(index) {
      // Memasukkan nilai item yang sedang diedit ke dalam form
      this.newExpense = {...this.expenses[index]};
      this.editingIndex = index; // Menandai item yang sedang diedit
    },
    cancelEdit() {
      // Membatalkan mode edit dan mengosongkan form
      this.editingIndex = null;
      this.resetForm();
    },
    removeExpense(index) {
      this.expenses.splice(index, 1); // Menghapus item dari array
      this.saveToLocalStorage(); // Menyimpan data ke local storage setelah menghapus pengeluaran
    },
    saveToLocalStorage() {
      localStorage.setItem('expenses', JSON.stringify(this.expenses)); // Menyimpan data ke local storage
    },
    resetForm() {
      // Mengosongkan form setelah submit atau pembatalan edit
      this.newExpense = { description: '', amount: '' };
    },
    formatRupiah(amount) {
      return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR' }).format(amount);
    },
    showAll() {
      this.currentFilter = 'All';
    },
    showActive() {
      this.currentFilter = 'Active';
    },
    showCompleted() {
      this.currentFilter = 'Completed';
    }
  }
};
</script>

<style scoped>
/* General styles */
body {
  font-family: 'Montserrat', sans-serif;
  margin: 0;
  padding: 0;
  background-color: #c1e90e;
}

#app {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f4c5c6;
}

h1 {
  text-align: center;
  color: #ffffff;
  margin-bottom: 25px;
}

.form {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-bottom: 20px;
}

.form label {
  width: 100px;
}

.form input {
  flex: 1;
  padding: 8px;
  margin-bottom: 10px;
}

.form button {
  background-color: #ffebd8;
  color: #ffffff;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
}

.form button:hover {
  background-color: #e51310;
}

.filter {
  margin-bottom: 20px;
}

.filter-buttons {
  margin-bottom: 20px;
}

.filter-buttons button {
  margin-right: 10px;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
}

.filter-buttons button.active {
  background-color: #ffebd8; /* Warna latar belakang tombol aktif */
}

.expenses {
  background-color: #fbe7ab;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  padding: 20px;
}

.expenses h2 {
  font-size: 24px;
  color: #f8a57f;
  margin-bottom: 20px;
}

.expenses ul {
  list-style: none;
  padding: 0;
}

.expenses li {
  border-bottom: 1px solid #59565600;
  padding: 10px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.expenses li:last-child {
  border-bottom: none;
}

.expenses .highlight {
  color: #ee7036;
}

.expenses .description {
  flex: 1;
}

.expenses .amount {
  font-weight: bold;
}

.checked {
  text-decoration: line-through; /* Coret teks jika item telah dicentang */
}

.action-buttons button {
  margin-left: 5px;
}

.edit-button {
  background-color: #4CAF50; /* Hijau */
}

.delete-button {
  background-color: #f44336; /* Merah */
}

.edit-button:hover, .delete-button:hover {
  background-color: #45a049; /* Hijau Gelap / Merah Gelap saat di hover */
}
</style>
