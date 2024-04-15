<template>
  <div class="text-center py-4   w-75 mx-auto">
    <v-data-table :headers="headers" :items="users">
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>CADASTRO</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <template v-slot:append>
            <v-btn class="mb-2" color="primary" dark @click="openDialog()">
              Novo item
            </v-btn>
          </template>
          <div class="text-center pa-4">
            <v-dialog v-model="dialog" width="auto">
              <v-card width="400">
                <v-card-title>{{ tempID ? 'Editar Usuario' : 'Criar Usuario' }}</v-card-title>
                <v-card-text>
                  <v-text-field v-model="tempNome" :rules="rules" label="Nome"></v-text-field>
                  <v-text-field v-model="tempEmail" :rules="rulesEmail" label="Email"></v-text-field>
                  <v-text-field v-model="tempID" label="ID" disabled></v-text-field>
                </v-card-text>
                <template v-slot:actions>
                  <v-btn class="ms-auto" @click="closeDialog()">Cancelar</v-btn>
                    <v-btn @click="tempID ? editUser() : createUser()" :disabled="disableNovoUser">{{ tempID ? 'Editar' : 'Criar' }}</v-btn>
                </template>
              </v-card>
            </v-dialog>
          </div>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon class="me-2" size="small" @click="editDialog(item)">
          mdi-pencil
        </v-icon>
        <v-icon size="small" @click="deleteUser(item)">
          mdi-delete
        </v-icon>
      </template>
    </v-data-table>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
const urlApi = 'http://localhost:3333/users'

const users = ref([])
const tempNome = ref('')
const tempEmail = ref('')
const tempID = ref('')
const rules = [
  (v) => !!v || 'Campo obrigatório',
  (v) => v && v.length >= 3 || 'Nome muito curto',
]
const rulesEmail = [
  (v) => !!v || 'Campo obrigatório',
  (v) => v && v.match(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/g) || 'Email inválido',
]

const dialog = ref(false)

const disableNovoUser = computed(() => {
  return !tempNome.value || !tempEmail.value || tempNome.value.length < 3 || !tempEmail.value.match(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/g);
})

const headers = [
  {
    title: 'Nome',
    align: 'start',
    sortable: true,
    key: 'name',
  },
  { title: 'Email', key: 'email' },
  { title: 'ID', key: 'id' },
  { title: 'Actions', key: 'actions', sortable: false },
]

const closeDialog = () => {
  dialog.value = false
  //tempNome.value = ''
  //tempEmail.value = ''
  //tempID.value = ''
}
const openDialog = () => {
  tempEmail.value = ''
  tempNome.value = ''
  tempID.value = ''
  dialog.value = true
}
const editDialog = (item) => {
  tempNome.value = item.name
  tempEmail.value = item.email
  tempID.value = item.id
  dialog.value = true
}

const carregarUsuarios = () => {
  fetch(urlApi, {
    method: 'GET'
  })
    .then(response => response.json())
    .then(data => {
      //console.log(data)
      users.value = data
    })
    .catch(error => console.error(error));
}
carregarUsuarios()

const deleteUser = (item) => {
  fetch(`${urlApi}/${item.id}`, {
    method: 'DELETE'
  })
    .then(() => {
      carregarUsuarios()
    })
    .catch(error => console.error(error));
}

const createUser = () => {
  fetch(urlApi, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: tempNome.value,
      email: tempEmail.value
    })
  })
    .then(() => {
      carregarUsuarios()
      closeDialog()
    })
    .catch(error => console.error(error));
}

const editUser = () => {
  fetch(`${urlApi}/${tempID.value}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: tempNome.value,
      email: tempEmail.value
    })
  })
    .then(() => {
      carregarUsuarios()
      closeDialog()
    })
    .catch(error => console.error(error));
}
</script>