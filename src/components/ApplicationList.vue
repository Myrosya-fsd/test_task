<template>
  <div>
    <h2>Список заяв</h2>

    <div class="filter">
      <label>
        Дата від:
        <input type="date" v-model="filterFrom" />
      </label>
      <label>
        Дата до:
        <input type="date" v-model="filterTo" />
      </label>
    </div>

    <ul>
      <li v-for="(app, index) in filteredApplications" :key="index">
        <strong>{{ app.number }}</strong>
        — {{ app.date }} — {{ app.customer.lastName }} {{ app.customer.firstName }}
        <button @click="viewDetails(app)">Деталі</button>
      </li>
    </ul>

    <div v-if="selectedApp" class="details">
      <h3>Деталі заяви {{ selectedApp.number }}</h3>
      <p>
        <strong>Причина:</strong>
        {{ selectedApp.reason }}
      </p>
      <p>
        <strong>Коментар:</strong>
        {{ selectedApp.comment }}
      </p>
      <p>
        <strong>Потужність:</strong>
        {{ selectedApp.power }} кВт
      </p>
      <p>
        <strong>Адреса:</strong>
        {{ selectedApp.customer.address }}
      </p>
      <p>
        <strong>Email:</strong>
        {{ selectedApp.customer.email }}
      </p>
      <p>
        <strong>Телефон:</strong>
        {{ selectedApp.customer.phone }}
      </p>

      <h4>Документи</h4>
      <ul>
        <li v-for="(file, idx) in selectedApp.files" :key="idx">
          {{ file.name }}
          <button @click="previewFile(file)">Переглянути</button>
        </li>
      </ul>

      <button @click="editApplication(selectedApp)">Редагувати</button>
      <button @click="printApplication(selectedApp)">Друк</button>
      <button @click="deleteApplication(selectedApp)">Видалити</button>
      <button @click="selectedApp = null">Закрити</button>
    </div>
  </div>
</template>

<script setup>
  import { ref, computed } from 'vue';
  const props = defineProps({ applications: Array });
  const emit = defineEmits(['edit-application']);

  const selectedApp = ref(null);
  const filterFrom = ref('');
  const filterTo = ref('');

  const filteredApplications = computed(() => {
    return props.applications.filter(app => {
      if (filterFrom.value && app.date < filterFrom.value) return false;
      if (filterTo.value && app.date > filterTo.value) return false;
      return true;
    });
  });

  function viewDetails(app) {
    selectedApp.value = app;
  }

  function previewFile(file) {
    const blob = fetch(file.content).then(res => res.blob());
    blob.then(b => {
      const fileURL = URL.createObjectURL(b);
      if (file.type === 'application/pdf') window.open(fileURL, '_blank');
      else alert('Для DOC/DOCX використайте MS Word або Google Docs');
    });
  }

  function editApplication(app) {
    emit('edit-application', app);
    selectedApp.value = null;
  }

  function printApplication(app) {
    const printWindow = window.open('', '', 'width=800,height=600');
    printWindow.document.write('<html><head><title>Заява #' + app.number + '</title></head><body>');
    printWindow.document.write('<h1>Заява #' + app.number + '</h1>');
    printWindow.document.write('<p><strong>Дата:</strong> ' + app.date + '</p>');
    printWindow.document.write('<p><strong>Причина:</strong> ' + app.reason + '</p>');
    printWindow.document.write('<p><strong>Коментар:</strong> ' + app.comment + '</p>');
    printWindow.document.write('<p><strong>Потужність:</strong> ' + app.power + ' кВт</p>');
    printWindow.document.write('<p><strong>Прізвище:</strong> ' + app.customer.lastName + '</p>');
    printWindow.document.write('<p><strong>Ім’я:</strong> ' + app.customer.firstName + '</p>');
    printWindow.document.write('<p><strong>Адреса:</strong> ' + app.customer.address + '</p>');
    printWindow.document.write('<p><strong>Email:</strong> ' + app.customer.email + '</p>');
    printWindow.document.write('<p><strong>Телефон:</strong> ' + app.customer.phone + '</p>');

    if (app.files && app.files.length > 0) {
      printWindow.document.write('<h4>Документи:</h4><ul>');
      app.files.forEach(f => {
        printWindow.document.write('<li>' + f.name + '</li>');
      });
      printWindow.document.write('</ul>');
    }

    printWindow.document.write('</body></html>');
    printWindow.document.close();
    printWindow.focus();
    printWindow.print(); // <-- виклик друку
  }

  function deleteApplication(app) {
    if (confirm(`Видалити заяву №${app.number}?`)) {
      emit('delete-application', app); // повідомляємо батьківський компонент
      selectedApp.value = null;
    }
  }
</script>

<style scoped>
  .filter {
    margin-bottom: 10px;
  }
  .filter label {
    margin-right: 20px;
  }
  .details {
    border: 1px solid #ccc;
    padding: 10px;
    margin-top: 10px;
  }
  ul {
    list-style: none;
    padding-left: 0;
  }
  li {
    margin-bottom: 5px;
  }
  button {
    margin-left: 10px;
  }
</style>
