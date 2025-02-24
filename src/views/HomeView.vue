<template>
  <v-container>
    <v-card border flat>
      
  <div class="home">
    <v-data-iterator
      :items="properties"
      :items-per-page="6"
      :search="search"
    >
      <template v-slot:header>
        <v-toolbar class="px-2">
          <v-text-field
            v-model="search"
            density="comfortable"
            placeholder="Search"
            prepend-inner-icon="mdi-magnify"
            style="max-width: 300px;"
            variant="solo"
            clearable
            hide-details
          ></v-text-field>
          <p class="ml-4 text-h6 mb-0">Propiedades disponibles</p>
        </v-toolbar>
      </template>

      <template v-slot:default="{ items }">
        <v-container class="pa-2" fluid>
          <v-row dense>
            <v-col
              v-for="item in items"
              :key="item.id"
              cols="auto"
              md="4"
            >
              <v-card class="pb-3" border flat>
                <v-img src="https://images.adsttc.com/media/images/5429/eea1/c07a/809a/0e00/0245/newsletter/01-_MG_5842.jpg?1412034196"></v-img>

                <v-list-item :subtitle="item.raw.description" class="mb-2 text-left">
                  <template v-slot:title>
                    <strong class="text-h6 mb-2">{{ item.raw.name }}</strong>
                  </template>
                </v-list-item>

                <div class="d-flex justify-space-between px-2">
                  <div class="d-flex align-center text-caption text-medium-emphasis me-1">
                    <v-icon icon="mdi-office-building-outline" start></v-icon>

                    <div class="text-truncate">{{ item.raw.type }}</div>
                  </div>

                  <ShowAvailableBricks :property="item.raw"/>
                </div>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </template>

      <template v-slot:footer="{ page, pageCount, prevPage, nextPage }">
        <div class="d-flex align-center justify-center pa-4">
          <v-btn
            :disabled="page === 1"
            density="comfortable"
            icon="mdi-arrow-left"
            variant="tonal"
            rounded
            @click="prevPage"
          ></v-btn>

          <div class="mx-2 text-caption">
            Page {{ page }} of {{ pageCount }}
          </div>

          <v-btn
            :disabled="page >= pageCount"
            density="comfortable"
            icon="mdi-arrow-right"
            variant="tonal"
            rounded
            @click="nextPage"
          ></v-btn>
        </div>
      </template>
    </v-data-iterator>
  </div>
    </v-card>
  </v-container>
</template>

<script>
import axios from 'axios';
import ShowAvailableBricks from "@/components/ShowAvailableBricks.vue"

export default {
  name: 'HomeView',
  components: {ShowAvailableBricks},
  data() {
    return {
      properties: [],
      search: ''
    }
  },
  async mounted(){
    this.properties = await this.getProperties()
  },
  methods: {
    getProperties(){
      return new Promise((res, rej) => {
        axios.get(`${process.env.VUE_APP_API_URL}/get-properties`)
        .then((response) => {
          res(response.data.status ? response.data.datos : []);
        })
        .catch((error) => {
          rej(error)
        })
      })
    }
  }
}
</script>
