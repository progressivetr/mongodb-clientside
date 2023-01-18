
<template>
  <select v-model="selectedRegion">
    <option v-for="(region, key) of regions" :key="key" :value="region">{{region}}</option>
  </select>
</template>

<script setup lang="ts">
import {onMounted, ref} from "vue";
import {App, Credentials} from "realm-web";

const regions = ref<string[]>([]);
const selectedRegion = ref<string>("");

const app = new App({id: 'data-vstdn'});
const credentials = Credentials.anonymous();
const user = app.logIn(credentials)
    .then(value => {
      const mongo = value.mongoClient('Cluster0');
      const collection = mongo.db('test').collection('documents');
      collection.aggregate([
        {
          "$group": {
            "_id": null,
            "regions": {
              "$addToSet": "$region"
            }
          }
        }
      ]).then(value1 => {
        regions.value = value1[0].regions;
      })


    })
    .catch(err => {
  console.error("Failed to log in", err);
});

</script>


