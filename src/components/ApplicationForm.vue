<template>
  <form @submit.prevent="submitForm">
    <h2>{{ formTitle }}</h2>

    <div>
      <label>Номер заяви:</label>
      <input v-model="form.number" type="text" readonly />
    </div>

    <div>
      <label>Дата створення:</label>
      <input v-model="form.date" type="date" readonly />
    </div>

    <div>
      <label>Причина звернення:</label>
      <select v-model="form.reason" required>
        <option disabled value="">Виберіть причину</option>
        <option value="нове підключення">Нове підключення</option>
        <option value="збільшення потужності">Збільшення існуючої потужності</option>
      </select>
    </div>

    <div>
      <label>Коментар:</label>
      <input v-model="form.comment" type="text" />
    </div>

    <div>
      <label>Потужність приєднання (кВт):</label>
      <input v-model.number="form.power" type="number" step="0.01" min="0" required />
    </div>

    <h3>Дані замовника</h3>
    <div>
      <label>Прізвище:</label>
      <input v-model="form.customer.lastName" type="text" required />
    </div>
    <div>
      <label>Ім’я:</label>
      <input v-model="form.customer.firstName" type="text" required />
    </div>
    <div>
      <label>Адреса:</label>
      <input v-model="form.customer.address" type="text" required />
    </div>
    <div>
      <label>Email:</label>
      <input v-model="form.customer.email" type="email" />
    </div>
    <div>
      <label>Телефон:</label>
      <input v-model="form.customer.phone" type="tel" />
    </div>

    <h3>Документи</h3>
    <input type="file" @change="handleFiles" multiple accept=".pdf,.doc,.docx" />
    <ul>
      <li v-for="(file, index) in form.files" :key="index">
        {{ file.name }}
        <button type="button" @click="previewFile(file)">Переглянути</button>
      </li>
    </ul>

    <button type="submit">{{ editingApp ? 'Оновити заяву' : 'Зберегти заяву' }}</button>
    <button type="button" @click="resetForm">Очистити форму</button>
  </form>
</template>

<script setup>
  import { reactive, watch, computed, onMounted } from 'vue';

  const emit = defineEmits(['save-application']);
  const props = defineProps({ nextNumber: Number, editApp: Object });

  const form = reactive({
    number: '',
    date: '',
    reason: '',
    comment: '',
    power: null,
    customer: { lastName: '', firstName: '', address: '', email: '', phone: '' },
    files: [],
  });

  const editingApp = computed(() => !!props.editApp);
  const formTitle = computed(() => (editingApp.value ? 'Редагування заяви' : 'Нова заява'));

  // Конвертація файлу у Base64
  function fileToBase64(file) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.onload = () => resolve(reader.result);
      reader.onerror = reject;
      reader.readAsDataURL(file);
    });
  }

  async function handleFiles(event) {
    const files = Array.from(event.target.files);
    form.files = await Promise.all(
      files.map(async f => ({
        name: f.name,
        type: f.type,
        content: await fileToBase64(f),
      }))
    );
  }

  function previewFile(file) {
    const blob = fetch(file.content).then(res => res.blob());
    blob.then(b => {
      const fileURL = URL.createObjectURL(b);
      if (file.type === 'application/pdf') window.open(fileURL, '_blank');
      else alert('Для DOC/DOCX використайте MS Word або Google Docs');
    });
  }

  // Ініціалізація форми
  function initForm() {
    if (props.editApp) Object.assign(form, props.editApp);
    else {
      form.number = props.nextNumber;
      form.date = new Date().toISOString().split('T')[0];
      form.reason = '';
      form.comment = '';
      form.power = null;
      form.customer = { lastName: '', firstName: '', address: '', email: '', phone: '' };
      form.files = [];
    }
  }

  onMounted(initForm);
  watch(() => props.editApp, initForm);

  function submitForm() {
    if (
      !form.number ||
      !form.date ||
      !form.power ||
      !form.customer.lastName ||
      !form.customer.firstName ||
      !form.customer.address
    ) {
      alert('Заповніть всі обов’язкові поля!');
      return;
    }
    emit('save-application', { ...form, files: [...form.files] });
    if (!editingApp.value) initForm();
  }

  function resetForm() {
    initForm();
  }
</script>

<style scoped>
  form div {
    margin-bottom: 10px;
  }
  label {
    display: block;
    font-weight: bold;
  }
  input,
  select,
  button {
    margin-top: 5px;
    padding: 5px;
    width: 100%;
    max-width: 400px;
  }
  button {
    width: auto;
    margin-right: 10px;
    margin-top: 10px;
  }
</style>
