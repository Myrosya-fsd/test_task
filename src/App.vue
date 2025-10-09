<template>
  <div class="container">
    <h1>Програма «Заява»</h1>

    <div class="actions">
      <button @click="showForm = true">Нова заява</button>
      <button @click="showCustomers = true">Довідник замовників</button>
    </div>

    <!-- Список заяв -->
    <application-list
      :applications="applications"
      @edit-application="startEditing"
      @delete-application="deleteApplication"
    />

    <!-- Модальне вікно для форми -->
    <div v-if="showForm" class="modal-overlay">
      <div class="modal">
        <application-form
          :next-number="nextNumber"
          :edit-app="editingApp"
          @save-application="saveApplicationModal"
        />
        <button class="close-btn" @click="closeForm">Закрити</button>
      </div>
    </div>

    <!-- Модальне вікно для довідника -->
    <div v-if="showCustomers" class="modal-overlay">
      <div class="modal">
        <customer-directory :applications="applications" />
        <button class="close-btn" @click="showCustomers = false">Закрити</button>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted } from 'vue';
  import ApplicationForm from './components/ApplicationForm.vue';
  import ApplicationList from './components/ApplicationList.vue';
  import CustomerDirectory from './components/CustomerDirectory.vue';

  const applications = ref([]);
  const nextNumber = ref(1);
  const editingApp = ref(null);
  const showForm = ref(false);
  const showCustomers = ref(false);

  onMounted(() => {
    const savedApps = JSON.parse(localStorage.getItem('applications')) || [];
    applications.value = savedApps;

    if (savedApps.length > 0) {
      nextNumber.value = Math.max(...savedApps.map(a => a.number)) + 1;
    }
  });

  function saveApplicationModal(app) {
    if (editingApp.value) {
      const index = applications.value.findIndex(a => a.number === editingApp.value.number);
      if (index !== -1) applications.value[index] = app;
      editingApp.value = null;
    } else {
      applications.value.push(app);
      nextNumber.value = app.number + 1;
    }
    localStorage.setItem('applications', JSON.stringify(applications.value));
    showForm.value = false;
  }

  function startEditing(app) {
    editingApp.value = app;
    showForm.value = true;
  }

  function deleteApplication(app) {
    applications.value = applications.value.filter(a => a.number !== app.number);
    localStorage.setItem('applications', JSON.stringify(applications.value));
    if (editingApp.value && editingApp.value.number === app.number) editingApp.value = null;
  }

  function closeForm() {
    showForm.value = false;
    editingApp.value = null;
  }
</script>

<style scoped>
  .container {
    max-width: 900px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
  }

  h1 {
    text-align: center;
    margin-bottom: 20px;
  }

  .actions {
    display: flex;
    justify-content: center;
    gap: 20px;
    margin-bottom: 20px;
  }

  button {
    padding: 8px 15px;
    cursor: pointer;
  }

  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal {
    background: white;
    padding: 20px;
    border-radius: 10px;
    max-width: 800px;
    width: 90%;
    max-height: 90vh;
    overflow-y: auto;
    position: relative;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  }

  .close-btn {
    display: block;
    margin: 15px auto 0;
    padding: 8px 15px;
  }
</style>
