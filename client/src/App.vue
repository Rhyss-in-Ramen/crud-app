<script setup>
import { ref, onMounted } from 'vue';

const API = 'http://localhost:4000/api/items';

const items = ref([]);

const form = ref({
  name: '',
  description: '',
  price: ''
});

const editId = ref(null);

async function load() {
  items.value = await fetch(API).then(r => r.json());
}

async function save() {
  const payload = {
    name: form.value.name,
    description: form.value.description,
    price: Number(form.value.price)
  };
  console.log('editId:', editId.value);
  console.log('payload:', payload);

  if (editId.value) {
    const res = await fetch(`${API}/${editId.value}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    });
    console.log(await res.json());

    editId.value = null;
  } else {
    await fetch(API, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    });
  }

  form.value = {
    name: '',
    description: '',
    price: ''
  };
  await load();
}
function startEdit(item) {
  form.value = {
    name: item.name,
    description: item.description,
    price: item.price
  };

  editId.value = item.id;
}

async function remove(id) {
  await fetch(`${API}/${id}`, {
    method: 'DELETE'
  });
  load();
}

onMounted(load);
</script>

<template>
  <main class="app-container">
    <h1 class="app-title">📦 Item List</h1>

    <form @submit.prevent="save" class="crud-form">
      <div class="form-inputs">
        <input v-model="form.name" placeholder="Item Name" required class="form-control"/>
        <input v-model="form.description" placeholder="Description" required class="form-control"/>
        <input v-model="form.price" type="number" step="0.01" placeholder="Price" required class="form-control"/>
      </div>

      <button type="submit" class="btn btn-submit">
        {{ editId ? 'Update Item' : 'Add Item' }}
      </button>
    </form>

    <ul class="item-list">
      <li v-for="item in items" :key="item.id" class="item-row">
        <div class="item-details">
          <strong class="item-name">{{ item.name }}</strong>
          <span class="item-desc">{{ item.description }}</span>
          <span class="item-price">₱{{ Number(item.price).toFixed(2) }}</span>
        </div>

        <div class="item-actions">
          <button @click="startEdit(item)" class="btn btn-edit">
            Edit
          </button>

          <button @click="remove(item.id)" class="btn btn-delete">
            Delete
          </button>
        </div>
      </li>
    </ul>
  </main>
</template>