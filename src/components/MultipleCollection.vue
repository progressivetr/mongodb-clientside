<template>
  <div v-if="isMongoDBReady">
    <el-select v-model="selectedRegion" @change="updateData(false)">
      <el-option v-for="(region, key) of regions" :key="key" :value="region">{{ region }}</el-option>
    </el-select>
    <el-table v-loading="loading" :data="data">
      <el-table-column prop="name" label="Name"/>
      <el-table-column prop="gpu" label="GPU"/>
      <el-table-column prop="os" label="OS"/>
      <el-table-column prop="sap" label="SAP"/>
      <el-table-column prop="ssd" label="SSD"/>
      <el-table-column prop="burstable" label="Burstable"/>
    </el-table>
    <el-button v-if="selectedRegion" :disabled="data.length >= pagination.total" @click="addNextData">Load More
    </el-button>
  </div>

</template>

<script setup lang="ts">
import {reactive, ref} from "vue";
import {App, Credentials} from "realm-web";

import MongoDBDatabase = Realm.Services.MongoDBDatabase;

const isMongoDBReady = ref<boolean>(false);
const selectedRegion = ref<string>("");

const regions = ['germany-north', 'germany-west-central', 'south-africa-north', 'south-africa-west', 'switzerland-north', 'switzerland-west', 'uae-central', 'uae-north', 'asia-pacific-east', 'asia-pacific-southeast', 'australia-central', 'australia-central-2', 'australia-east', 'australia-southeast', 'brazil-south', 'canada-central', 'canada-east', 'central-india', 'europe-north', 'europe-west', 'france-central', 'france-south', 'japan-east', 'japan-west', 'korea-central', 'korea-south', 'south-india', 'united-kingdom-south', 'united-kingdom-west', 'us-central', 'us-east', 'us-east-2', 'usgov-arizona', 'usgov-texas', 'usgov-virginia', 'us-north-central', 'us-south-central', 'us-west', 'us-west-2', 'us-west-central', 'west-india', 'norway-east', 'norway-west']

const data = ref<any[]>([]);
const pagination = reactive({
  page: 1,
  total: 0,
  limit: 100,
})
const loading = ref<boolean>(false);


const app = new App({id: 'data-gbqrc'});
const credentials = Credentials.anonymous();

let mongoClient!: MongoDBDatabase;
app.logIn(credentials)
    .then(value => {
      mongoClient = value.mongoClient('Cluster0').db('azure-pricing');
      isMongoDBReady.value = true;
    })
    .catch(err => {
      console.error("Failed to log in", err);
    });

function addNextData() {
  pagination.page += 1;
  updateData(true);
}

function updateData(add = false) {
  loading.value = true;
  const collection = mongoClient.collection(selectedRegion.value);
  collection.aggregate([
    {
      "$facet": {
        "data": [
          {"$skip": pagination.limit * (pagination.page - 1)},
          {"$limit": pagination.limit}
        ],
        "count": [
          {"$count": "count"}
        ]
      }
    },
  ]).then(value => {
    pagination.total = value[0].count[0].count;
    if (add) {
      data.value.push(...value[0].data);
    } else {
      data.value = value[0].data;
    }
  }).finally(() => {
    loading.value = false;
  })
}

</script>
