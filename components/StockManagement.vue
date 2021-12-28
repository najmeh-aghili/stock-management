<template>
<b-container>
    <b-col cols="12">
        <b-skeleton-table v-if="showSkeleton" :rows="10" :columns="3" :table-props="{ bordered: true, striped: true }">
        </b-skeleton-table>
        <b-card v-else header-tag="header" footer-tag="footer">
            <div class="card-header">
                <i class="fa fa-align-justify" />
                Product List
            </div>
            <div class="app-btn">
                <div v-if="!$fetchState.pending || !$fetchState.error">
                    <b-table show-empty stacked="md" :items="productData" :fields="fields" :current-page="currentPage" striped hover bordered :per-page="perPage">
                        <template v-slot:cell(actions)="row">
                            <b-button class="mr-3" v-b-modal.increaseStock size="sm" variant="info" @click="ProductStock(row.item.id,row.item.name)">
                                Fill
                            </b-button>
                            <b-button class="mr-3" v-b-modal.decreasStock size="sm" variant="info" @click="ProductStock(row.item.id,row.item.name)">
                                Buy
                            </b-button>
                            <b-button class="mr-3" v-b-modal.showAmountModal size="sm" variant="info" @click="showProductStock(row.item.id,row.item.name)">
                                Show Stock
                            </b-button>
                        </template>
                    </b-table>
                </div>
                <b-pagination v-model="currentPage" :total-rows="totalRows" class="my-0" :per-page="perPage" align="fill" size="sm" first-text="First" prev-text="Previous" next-text="Next" last-text="Last" />
            </div>
        </b-card>
    </b-col>
    <b-modal id="increaseStock" centered hide-footer title="Increse Stock">
        <b-form-group>
            <pre>{{productName}}</pre>
            <label class="mb-1">
                Please enter amount
            </label>
            <b-form-input v-model="amount" type="number" class="mb-3" />
        </b-form-group>

        <b-button @click="fillProduct" variant="info">
            Increase
        </b-button>
        <b-button @click="closefillModal" variant="info">
            cancel
        </b-button>
    </b-modal>

    <b-modal id="decreasStock" centered hide-footer title="Decreas Stock">
        <b-form-group>
            <div>
                {{productName}}
            </div>
            <label class="mb-1">
                Please enter amount
            </label>
            <b-form-input v-model="amount" type="number" class="mb-3" />
        </b-form-group>

        <b-button @click="buyProduct" variant="info" :disabled="btnDisable">
            Decreas
        </b-button>
        <b-button @click="closeBuyModal" variant="info">
            Cancel
        </b-button>
    </b-modal>
    <b-modal id="showAmountModal" centered hide-footer title="Stock">
        <b-form-group>
            <label class="mb-1">
                {{productName}} : {{productStock}}
            </label>
        </b-form-group>
    </b-modal>
</b-container>
</template>

<script>
import '~/node_modules/vue-snotify/styles/material.css'

export default {
    data() {
        return {
            productData: [],
            fields: [{
                    key: 'id',
                    label: 'Id ',
                },
                {
                    key: 'name',
                    label: 'Name ',
                },
                {
                    key: 'actions',
                    label: 'Actions'
                }
            ],
            currentPage: 1,
            perPage: 10,
            pages: 1,
            totalRows: 1,
            increaseStock: false,
            amount: '',
            productId: '',
            productStock: '',
            productName: '',
            btnDisable: false,
            showSkeleton: true
        }
    },
    async fetch() {
        this.productData = await this.$axios.$get('/product')
        if (this.productData) {
            this.totalRows = this.productData.length
            this.showSkeleton = false
        }
    },

    methods: {
        ProductStock(id, name) {
            this.productId = id
            this.productName = name
            this.amount = ''

        },
        buyProduct() {
            if (this.amount) {
                this.$axios
                    .$put(`/product/${this.productId}/buy?amount=${this.amount}`)
                    .then(res => {
                        if (res) {
                            this.$toast.success('Successful purchase', {
                                duration: 1000
                            })
                            const index = this.productData.findIndex(element => element.id === this.productId)
                            this.productData[index].stock -= this.amount
                        }
                    })
                    .catch(err => {
                        console.log(err)
                        this.$toast.error('Inventory shortage', {
                            duration: 1000
                        })
                    })
            }
            this.$bvModal.hide('decreasStock')
        },
        fillProduct() {
            if (this.amount) {
                this.$axios
                    .$put(`/product/${this.productId}/refill?amount=${this.amount}`)
                    .then(res => {
                        if (res) {
                            this.$toast.success('Successful inventory increase', {
                                duration: 1000
                            })
                        }
                    })
                    .catch(err => {
                        console.log(err)
                    })
            }
            this.$bvModal.hide('increaseStock')
        },
        showProductStock(id, name) {
            this.productName = name
            this.$axios
                .$get(`/product/${id}/stock`)
                .then(res => {
                    if (res) {
                        this.productStock = res
                    }
                })
                .catch(err => {
                    console.log(err)
                })
        },
        closefillModal() {
            this.$bvModal.hide('increaseStock')
        },
        closeBuyModal() {
            this.$bvModal.hide('decreasStock')
        },
        closShowStockModal() {
            this.$bvModal.hide('showAmountModal')
        }

    }
}
</script>

<style scoped>
.card-header {
    padding: 0.75rem 1.25rem;
    margin-bottom: 0;
    background-color: #f0f3f5;
    border: 1px solid #c8ced3;
}

.card {
    border: none;
}

.btn-check:focus+.btn,
.btn:focus {
    box-shadow: none !important;
}
</style>
