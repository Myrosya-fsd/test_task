<template>
  <div class="customer-directory">
    <h2>Довідник замовників</h2>

    <div class="search">
      <label>
        Пошук:
        <input type="text" v-model="searchQuery" placeholder="Прізвище, Ім’я або Адреса" />
      </label>
    </div>

    <ul>
      <li v-for="(customer, idx) in filteredCustomers" :key="idx">
        <strong>{{ customer.lastName }} {{ customer.firstName }}</strong>
        — {{ customer.address }}
        <span v-if="customer.email">| Email: {{ customer.email }}</span>
        <span v-if="customer.phone">| Телефон: {{ customer.phone }}</span>
      </li>
    </ul>

    <p v-if="filteredCustomers.length === 0">Нічого не знайдено.</p>
  </div>
</template>

<script setup>
  import { computed, ref, watch } from 'vue';

  const props = defineProps({
    applications: {
      type: Array,
      required: true,
    },
  });

  const searchQuery = ref('');

  // Створюємо унікальний список замовників
  const customers = computed(() => {
    const map = new Map();
    props.applications.forEach(app => {
      const key = `${app.customer.lastName}-${app.customer.firstName}-${app.customer.address}`;
      if (!map.has(key)) {
        map.set(key, { ...app.customer });
      }
    });
    return Array.from(map.values());
  });

  // Фільтрація по пошуку
  const filteredCustomers = computed(() => {
    if (!searchQuery.value) return customers.value;

    const query = searchQuery.value.toLowerCase();
    return customers.value.filter(
      c =>
        c.lastName.toLowerCase().includes(query) ||
        c.firstName.toLowerCase().includes(query) ||
        c.address.toLowerCase().includes(query)
    );
  });
</script>

<style scoped>
  .customer-directory {
    margin-top: 30px;
    border-top: 1px solid #ccc;
    padding-top: 20px;
  }
  .search {
    margin-bottom: 10px;
  }
  input[type='text'] {
    padding: 5px;
    width: 300px;
  }
  ul {
    list-style: none;
    padding-left: 0;
  }
  li {
    margin-bottom: 5px;
  }
</style>
