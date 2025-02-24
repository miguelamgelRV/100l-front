<template>
  <div class="about">
    <v-container>
      <div style="display: flex; justify-content: center;">
        <v-select v-model="userSelected" variant="outlined" density="comfortable" label="User" rounded="lg"
          :items="users" item-value="id" item-title="name" style="max-width: 300px;">
        </v-select>
      </div>
      <v-divider class="mx-15 my-3"></v-divider>
      <v-row dense>
        <v-col cols="12" v-for="(brick, index) in shoppingCart" :key="index">
          <v-card color="blue-grey-darken-1" variant="outlined" flat>
            <div class="d-flex flex-no-wrap justify-start">
              <v-avatar class="ma-3" rounded="0" size="125">
                <v-img
                  src="https://images.adsttc.com/media/images/5429/eea1/c07a/809a/0e00/0245/newsletter/01-_MG_5842.jpg?1412034196"></v-img>
              </v-avatar>
              <div>

                <v-card-title class="text-h5">
                  Brick ID:{{ brick.id }}
                  <div v-show="brick.bricks.buyed" style="width: 100%;">
                    <v-alert density="compact" variant="outlined" type="error">
                      <p class="text-body-1">This brick was already purchased for someone else!</p>
                    </v-alert>
                  </div>
                </v-card-title>

                <v-card-text>
                  Cost ${{ brick.bricks.cost }}.00
                </v-card-text>
              </div>
            </div>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn class="ms-2" prepend-icon="mdi-close" size="small" text="remove" color="pink-darken-3"
                variant="tonal" @click="removeBrick(brick.id)"></v-btn>
              <v-btn class="ms-2" prepend-icon="mdi-cart-check" size="small" text="buy" color="teal-darken-3"
                variant="tonal" v-show="!brick.bricks.buyed" @click="showTermsAndConditions = true"></v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>

    <TermsAndConditions v-if="showTermsAndConditions" :termsAndConditions="showTermsAndConditions" />
  </div>
</template>

<script>
import axios from 'axios'

import TermsAndConditions from "@/components/TermsAndConditionsComponent.vue"

export default {
  components: { TermsAndConditions },
  data() {
    return {
      users: [],
      userSelected: null,
      shoppingCart: [],
      showTermsAndConditions: false
    }
  },
  async mounted() {
    this.users = await this.getUsers();
    this.userSelected = this.users[0]?.id ? this.users[0].id : null;
    this.shoppingCart = await this.getShoppingCartByUser();
  },
  methods: {
    getUsers() {
      return new Promise((res, rej) => {
        axios.get(`${process.env.VUE_APP_API_URL}/get-users`)
          .then((response) => {
            res(response.data.status ? response.data.datos : []);
          })
          .catch((error) => {
            rej(error)
          })
      })
    },
    getShoppingCartByUser() {
      return new Promise((res, rej) => {
        axios.get(`${process.env.VUE_APP_API_URL}/get-shopping-cart-by-user?id=${this.userSelected}`)
          .then((response) => {
            res(response.data.status ? response.data.datos[0]["bricks-in-carts"] : []);
          })
          .catch((error) => {
            rej(error)
          })
      })
    },
    async removeBrick(id) {
      const response = confirm(`Do you want to remove the brick ${id} from your shopping cart?`);

      if (response) {
        const transaction = await this.deleteBrickFromCartTransaction(id);

        if (transaction) {
          this.shoppingCart = await this.getShoppingCartByUser();
        }
      }
    },
    deleteBrickFromCartTransaction(id) {
      return new Promise((res, rej) => {
        axios.delete(`${process.env.VUE_APP_API_URL}/remove-brick-in-shopping-cart?id=${id}`)
          .then((response) => {
            res(response.data.status);
          })
          .catch((error) => {
            rej(error)
          })
      })
    },
  }
}
</script>
