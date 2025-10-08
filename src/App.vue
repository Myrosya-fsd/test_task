<template>
  <div class="container">
    <h1>Програма «Заява»</h1>

    <application-form
      @save-application="saveApplication"
      :next-number="nextNumber"
      :edit-app="editingApp"
    />

    <application-list
      :applications="applications"
      @edit-application="startEditing"
      @delete-application="deleteApplication"
    />

    <!-- Довідник замовників -->
    <customer-directory :applications="applications" />
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

  onMounted(() => {
    const savedApps = JSON.parse(localStorage.getItem('applications')) || [];
    applications.value = savedApps;

    if (savedApps.length > 0) {
      const maxNumber = Math.max(...savedApps.map(a => a.number));
      nextNumber.value = maxNumber + 1;
    } else {
      nextNumber.value = 1;
    }
  });

  function saveApplication(app) {
    if (editingApp.value) {
      const index = applications.value.findIndex(a => a.number === editingApp.value.number);
      if (index !== -1) applications.value[index] = app;
      editingApp.value = null;
    } else {
      applications.value.push(app);
      nextNumber.value = app.number + 1;
    }
    localStorage.setItem('applications', JSON.stringify(applications.value));
  }

  function startEditing(app) {
    editingApp.value = app;
  }

  function deleteApplication(app) {
    applications.value = applications.value.filter(a => a.number !== app.number);
    localStorage.setItem('applications', JSON.stringify(applications.value));
    if (editingApp.value && editingApp.value.number === app.number) editingApp.value = null;
  }
</script>

<style>
  .container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
  }
  h1 {
    text-align: center;
    margin-bottom: 20px;
  }
</style>
