<script setup>
import { ref, onMounted } from 'vue';
const API = 'http://localhost:4000/api/items';
const items = ref([]);
const form = ref({ name: '', description: '' });
const editId = ref(null);

async function load(){
  items.value = await fetch(API).then(r => r.json());
}

async function save(){
  if(editId.value){
    await fetch(`${API}/${editId.value}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
    editId.value = null;
  } else {
    await fetch(API, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    });
  }
  form.value = { name: '', description: '' };
  load();
}

function startEdit(item){
  form.value = { name: item.name, description: item.description };
  editId.value = item.id;
}
async function remove(id){
  await fetch(`${API}/${id}`, { method: 'DELETE' });
  load();
}
onMounted(load);
</script> 

<template>
  <main class="app-container">
    <h1 class="app-title">Items Manager</h1>
    
    <form @submit.prevent="save" class="crud-form">
      <div class="form-inputs">
        <input v-model="form.name" placeholder="Name" required class="form-control">
        <input v-model="form.description" placeholder="Description" required class="form-control">
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
        </div>
        <div class="item-actions">
          <button @click="startEdit(item)" class="btn btn-edit">Edit</button>
          <button @click="remove(item.id)" class="btn btn-delete">Delete</button>
        </div>
      </li>
    </ul>
  </main>
</template>