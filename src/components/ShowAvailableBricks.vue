<template>
    <div class="">
        <v-dialog width="800" scrollable>
            <template v-slot:activator="{ props: activatorProps }">
                <v-btn class="text-none" v-bind="activatorProps" size="small" text="See availability" variant="flat"
                    color="#d71313" border></v-btn>
            </template>

            <template v-slot:default="{ isActive }">
                <v-card prepend-icon="mdi-office-building-outline" :title="'Bricks to ' + property.name">
                    <v-divider class="mt-3"></v-divider>

                    <v-card-text class="px-4" style="height: 70vh;">

                        <div style="display: flex; justify-content: center;">
                            <v-select v-model="userSelected" variant="outlined" density="comfortable" label="User"
                                rounded="lg" :items="users" item-value="id" item-title="name" style="max-width: 300px;">
                            </v-select>
                        </div>

                        <v-alert v-model="alert" :type="alertType" variant="outlined" closable class="mb-3">
                            {{ alertMessage }}
                        </v-alert>

                        <v-row dense>
                            <v-col cols="4" v-for="(brick, index) in property.bricks" :key="index">
                                <v-card border flat :color="!brick.buyed ? 'green-lighten-5' : 'blue-grey-lighten-5'">
                                    <v-card-title class="subtitle-1">
                                        ID: {{ brick.id }}
                                    </v-card-title>
                                    <v-card-text>
                                        <p class="text-center subtitle-2 font-weight-bold mt-2">
                                            {{ property.name }}
                                        </p>
                                        <p class="text-center subtitle-1 font-weight-black mt-2">
                                            ${{ brick.cost }}.00
                                        </p>
                                    </v-card-text>
                                    <v-card-actions>
                                        <v-btn flat block variant="outlined" prepend-icon="mdi-cart-plus"
                                            :disabled="brick.buyed"
                                            :color="!brick?.buyed ? 'green-lighten-2' : 'blue-grey-lighten-2'" @click="validateTransaction(brick.id)">
                                            Add to cart
                                        </v-btn>
                                    </v-card-actions>
                                </v-card>
                            </v-col>
                        </v-row>
                    </v-card-text>

                    <v-divider></v-divider>

                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn text="Close" @click="isActive.value = false"></v-btn>
                    </v-card-actions>
                </v-card>
            </template>
        </v-dialog>
    </div>
</template>

<script>
import axios from 'axios'
export default {
    name: "show-available-bricks",
    props: ["property"],
    data() {
        return {
            users: [],
            userSelected: null,
            alertMessage: "",
            alertType: "error",
            alert: false
        }
    },
    async mounted() {
        this.users = await this.getUsers();
        this.userSelected = this.users[0]?.id ? this.users[0].id : null;
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

        async validateTransaction(brickId) {
            await this.addBrickInCart(brickId)
                .then((response) => {
                    if (response?.datos?.id) {
                        this.alertMessage = "Added successfully";
                        this.alertType = "success"
                        this.alert = true;
                    } else {
                        this.alertMessage = `An error occurred '${response}'`;
                        this.alertType = "error"
                        this.alert = true;
                    }
                })
                .catch((error) => {
                    this.alertMessage = error;
                        this.alertType = "error"
                        this.alert = true;
                })
        },

        addBrickInCart(brickId) {
            const params = {
                brickId: brickId,
                shoppingCartId: this.userSelected
            }

            return new Promise((res, rej) => {
                axios.post(`${process.env.VUE_APP_API_URL}/add-brick-in-shopping-cart`, params)
                    .then((response) => {
                        res(response.data.status ? response.data : response.data.message);
                    })
                    .catch((error) => {
                        rej(error)
                    })
            })
        }
    }
}
</script>