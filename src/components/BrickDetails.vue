<template>
    <v-dialog v-model="localDialogOpen" transition="dialog-top-transition" width="auto" persistent>
        <v-card>
            <v-toolbar title="Review"></v-toolbar>

            <v-card-text class="pa-12">
                <div v-if="brickDetails?.id">
                    <p class="body-1"><strong>ID:</strong>{{ brickDetails.id }}</p>
                    <p class="body-1"><strong>Property:</strong>{{ brickDetails.propertyId }}</p>
                    <p class="body-1"><strong>Cost:</strong>${{ brickDetails.cost }}.00</p>
                    <p class="body-1"><strong>Creation date:</strong>{{ brickDetails.createdAt }}</p>
                </div>
                <div v-else>
                    <p class="text-center text-caption">
                        An error ocurred. No information to display.
                    </p>
                </div>
            </v-card-text>

            <v-card-actions class="justify-end">
                <v-btn text="Cancel" prepend-icon="mdi-close" color="pink-darken-3" variant="outlined" @click="localDialogOpen = false; $emit('buyBrick', false)"></v-btn>
                <v-btn text="Complete" prepend-icon="mdi-check" color="teal" variant="flat" @click="localDialogOpen = false; $emit('buyBrick', true)"></v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import axios from 'axios'

export default {
    props: ["dialogCompletePurchase", "inPurchaseBrick"],
    data() {
        return {
            localDialogOpen: false,
            brickDetails: {}
        }
    },
    async mounted() {
        this.localDialogOpen = this.dialogCompletePurchase;
        this.brickDetails = await this.getBrickDetails();
    },
    methods: {
        getBrickDetails() {
            return new Promise((res, rej) => {
                axios.get(`${process.env.VUE_APP_API_URL}/get-brick-details?id=${this.inPurchaseBrick.brick}`)
                    .then((response) => {
                        res(response.data.status ? response.data.datos : {});
                    })
                    .catch((error) => {
                        rej(error)
                    })
            })
        }
    }
}
</script>