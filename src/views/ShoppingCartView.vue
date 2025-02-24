<template>
  <div class="about">
    <v-container>
      <div style="display: flex; justify-content: center;">
        <v-select v-model="userSelected" variant="outlined" density="comfortable" label="User" rounded="lg"
          :items="users" item-value="id" item-title="name" style="max-width: 300px;"
          @update:modelValue="refreshShoppingCart()">
        </v-select>
      </div>
      <v-divider class="mx-15 my-3"></v-divider>
      <v-card flat variant="outlined" rounded="lg" class="pa-3 mb-2" v-for="property in shoppingCart"
        :key="property.id">
        <p class="subtitle-1 font-weight-bold text-left">{{ property.name }}</p>
        <v-row dense>
          <v-col cols="12" v-for="(brick, index) in property.bricks" :key="index">
            <v-card color="blue-grey-lighten-5" variant="flat" flat>
            <div class="d-flex flex-no-wrap justify-start">
              <v-avatar class="ma-3" rounded="0" size="125">
                <v-img
                  src="https://images.adsttc.com/media/images/5429/eea1/c07a/809a/0e00/0245/newsletter/01-_MG_5842.jpg?1412034196"></v-img>
              </v-avatar>
              <div>

                <v-card-title class="text-h5">
                  Brick ID:{{ brick.id }}
                  <div v-show="brick.buyed" style="width: 100%;">
                    <v-alert density="compact" variant="outlined" type="error">
                      <p class="text-body-1">This brick was already purchased for someone else!</p>
                    </v-alert>
                  </div>
                </v-card-title>

                <v-card-text>
                  Cost ${{ brick.cost }}.00
                </v-card-text>
              </div>
            </div>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn class="ms-2" prepend-icon="mdi-close" size="small" text="remove" color="pink-darken-3"
                variant="tonal" @click="removeBrick(brick['bricks-in-carts'][0].id )"></v-btn>
              <v-btn class="ms-2" prepend-icon="mdi-cart-check" size="small" text="buy" color="teal-darken-3"
                variant="tonal" v-show="!brick.buyed"
                @click="inPurchaseBrick = { brick: brick.id, user: userSelected }, showTermsAndConditions = true"></v-btn>
            </v-card-actions>
          </v-card>
          </v-col>
        </v-row>
      </v-card>
    </v-container>

    <TermsAndConditions v-if="showTermsAndConditions" :termsAndConditions="showTermsAndConditions"
      @validate="setTermsResponse" />

    <BrickDetails v-if="dialogCompletePurchase" :dialogCompletePurchase="dialogCompletePurchase"
      :inPurchaseBrick="inPurchaseBrick" @buyBrick="setCompletePurchase" />

    <v-snackbar v-model="snackbar" :color="colorSnackBar">

      <template v-slot:actions>
        <v-btn color="white" variant="text" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script>
import axios from 'axios'

import TermsAndConditions from "@/components/TermsAndConditionsComponent.vue"
import BrickDetails from "@/components/BrickDetails.vue"

export default {
  components: { TermsAndConditions, BrickDetails },
  data() {
    return {
      users: [],
      userSelected: null,
      shoppingCart: [],
      showTermsAndConditions: false,
      snackbar: false,
      dialogCompletePurchase: false,
      inPurchaseBrick: {},
      textSnackBar: '',
      colorSnackBar: '',
    }
  },
  async mounted() {
    this.users = await this.getUsers();
    this.userSelected = this.users[0]?.id ? this.users[0].id : null;
    this.shoppingCart = await this.getShoppingCartByUser();
  },
  methods: {
    showSnackBar(text, color = 'error') {
      this.textSnackBar = text;
      this.colorSnackBar = color;
      this.snackbar = true;
    },
    async refreshShoppingCart() {
      this.shoppingCart = await this.getShoppingCartByUser();
    },
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
            res(response.data.status ? response.data.datos : []);
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
    async setTermsResponse(response) {
      this.showTermsAndConditions = false;
      const transaction = await this.registerPurchase(response)
        .catch((error) => {
          this.showSnackBar(error)
        });

      if (transaction && response) {
        this.dialogCompletePurchase = true;
      } else if (!transaction && response) {
        this.showSnackBar("An error ocurred. The transaction was not completed.")
      }
    },
    registerPurchase(userResponse) {
      const params = {
        brickId: this.inPurchaseBrick.brick,
        userId: this.inPurchaseBrick.user,
        isTersAccepted: userResponse,
        isBrickPurchased: false
      }
      return new Promise((res, rej) => {
        axios.post(`${process.env.VUE_APP_API_URL}/add-purchase`, params)
          .then((response) => {
            res(response.data.status);
          })
          .catch((error) => {
            rej(error)
          })
      })
    },
    async setCompletePurchase(response) {
      this.dialogCompletePurchase = false;
      const transaction = await this.completePurchase(this.inPurchaseBrick)
        .catch((error) => {
          this.showSnackBar(error)
        });

      if (transaction && response) {
        this.showSnackBar("Purchase successfully completed.", "success");

        await this.refreshShoppingCart();
      } else if (!transaction && response) {
        this.showSnackBar("An error ocurred. The transaction was not completed.");
      }
    },
    completePurchase(data) {
      const params = {
        userId: data.user,
        brickId: data.brick
      }
      return new Promise((res, rej) => {
        axios.patch(`${process.env.VUE_APP_API_URL}/buy-brick`, params)
          .then((response) => {
            res(response.data.status);
          })
          .catch((error) => {
            rej(error)
          })
      })
    }

  }
}
</script>
