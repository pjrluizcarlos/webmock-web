<template>
  <div id="container" class="d-flex bg-grey flex-grow">
    <div id="form" class="bg-white flex-grow-1">
      <v-form ref="form">
        <v-text-field v-model="model.id" label="ID" disabled v-show="model.id" />
        <v-text-field v-model="model.uri" label="URI" placeholder="/v1/ivoices/123" />

        <v-row>
          <v-col>
            <v-text-field v-model="model.method" label="Method" placeholder="GET" />
          </v-col>
          <v-col>
            <v-text-field v-model="model.status" autocomplete="null" type="number" label="Status" placeholder="200" />
          </v-col>
        </v-row>

        <v-textarea class="align-self-stretch" v-model="model.response" label="Response" placeholder="{}" />

        <v-btn color="primary" class="mt-4" block :disabled="!isValid" @click="save">
          Save
        </v-btn>

        <v-btn class="mt-4" block @click="resetItem">
          Reset
        </v-btn>

        <v-btn class="mt-4" block @click="remove" v-show="canDelete" :disabled="canDelete">
          Remove
        </v-btn>
      </v-form>
    </div>


    <div id="items" class="bg-red flex-grow-1">
      <div v-for="(item, index) in models" :key="item.id" @click="loadItem(item)" :class="(index % 2 === 0) ? 'bg-blue' : 'bg-white'">
        <p>[{{ item.method }}] {{ item.uri }} > {{ item.status }}</p>
        <pre>{{ JSON.parse(item.response, null, 2) }}</pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios'
import { onBeforeMount } from 'vue';
import { reactive, computed, ref } from 'vue'

const model = reactive({})
const models = ref([])

onBeforeMount(() => {
  resetItem()
  fillModels()
})

const canDelete = computed(() => {
  return !!model.id
})

const isValid = computed(() => {
  return !!model.uri && !!model.method && !!model.status
})

const save = () => {
  const data = {
    id: model.id,
    uri: model.uri,
    method: model.method,
    status: model.status,
    response: model.response
  }

  axios.post('http://localhost:8080/mocks-configuration', data)
    .then(fillModels)
    .then(resetItem)
}

const resetItem = () => {
  model.id = null
  model.uri = null
  model.method = null
  model.response = null
  model.status = null
}

const loadItem = (item) => {
  model.id = item.id
  model.uri = item.uri
  model.method = item.method
  model.response = item.response
  model.status = item.status
}

const fillModels = () => {
  return findAll()
    .then(response => models.value = response.data)
}

const findAll = () => {
  return axios.get('http://localhost:8080/mocks-configuration')
}

const remove = () => {
  return axios.delete(`http://localhost:8080/mocks-configuration/${model.id}`)
}
</script>

<style>
#form {
  padding: 20px;
  margin: 20px;
  max-width: 500px;
  min-width: 300px;
}

#items {
  margin: 20px;
}
</style>