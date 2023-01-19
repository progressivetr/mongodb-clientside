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

const collectionName = "vm-list";

const isMongoDBReady = ref<boolean>(false);
const selectedRegion = ref<string>("");

const regions = ref<string[]>([]);

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
      updateRegions();
    })
    .catch(err => {
      console.error("Failed to log in", err);
    });

function updateRegions() {
  mongoClient.collection(collectionName).aggregate([{
    "$group": {
      "_id": null,
      "regions": {
        "$addToSet": "$region"
      }
    },
  }]).then(regionsResponse => {
    regions.value = regionsResponse[0].regions;
  }).catch(reason => {
    console.error('Unable to update regions', reason);
  })
}

function addNextData() {
  pagination.page += 1;
  updateData(true);
}

function updateData(add = false) {
  loading.value = true;
  const collection = mongoClient.collection(collectionName);
  collection.aggregate([
    {$match: {region: selectedRegion.value}},
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
