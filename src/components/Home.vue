<template>
  <div class="home debug">
    <QUX :app="hash" :config="config" :showDebug="true" v-model="viewModel" />
  </div>
</template>
<style lang="scss">
  .home {
    height: 100%;
  }
</style>

<script>
  import Vue from "vue";
  import QUX from 'vue-low-code'
  import axios from 'axios'
  Vue.use(QUX);

  export default {
    name: 'home',
    data: function () {
      return {
        hash: "a2aa10aypCe57tVhNvvWifUHQHJ8eHnFIWXv9wIfZ7f8KJhUKEod04nuf6W6",
        viewModel: {
          notifications: new Set(),
          selectedProducts: new Map(), // productid to quantity
          selectedProductsInAreaDisplay: [],
          notificationsArr: [],
          selectedDiscounts: new Map(), // disocuntid to product in offer id
          selectedProductsStringified: "",
          selectedDiscountsStringified: "",
          stringifiedTransactionDate: "",
          stringifiedOrderDate: "",
          selectedProductsInNewStore: [],
          selectedProductsInNewStoreForSend: [],
          cart: [],
          pushSocket: null
        },
        config: {
          debug: {
            resize: false,
            logLevel: 0
          },
          css: {
            grid: true,
            justifyContentInWrapper: true
          },
          router: {
            key: 'screenName',
            prefix: ''
          }
        },
        showPayload: false
      }
    },
    components: {},
    methods: {
      validateRole() {
        if(this.viewModel.role !== 'seller') {
          this.viewModel.role = 'consumer'
        }
      },
      updateProductChoise() {
        if(this.viewModel.productQuantity === 0) {
          return 
        }
        if(this.viewModel.selectedProducts[this.viewModel.selectedProduct.id]) {
          this.viewModel.selectedProducts[this.viewModel.selectedProduct.id] += this.viewModel.productQuantity
        }
        else {
          this.viewModel.selectedProducts[this.viewModel.selectedProduct.id] = this.viewModel.productQuantity
        }
        this.viewModel.selectedProductsStringified = JSON.stringify(this.viewModel.selectedProducts)
        this.viewModel.selectedDiscountsStringified = JSON.stringify(this.viewModel.selectedDiscounts)
        this.viewModel.selectedProductsInAreaDisplay = this.viewModel.allProductsInAreaResponse.allProducts
          .filter(product => this.viewModel.selectedProducts[product.id] !== undefined)
        this.viewModel.selectedProductsInAreaDisplay
          .find(product => product.id === this.viewModel.selectedProduct.id)
          .quantity = this.viewModel.productQuantity
        const objectInCart = Object.create(this.viewModel.selectedProductsInAreaDisplay
          .find(product => product.id === this.viewModel.selectedProduct.id))
        objectInCart.quantity = this.viewModel.productQuantity
        if(this.viewModel.cart.find(o => o.id === objectInCart.id)) {
            this.viewModel.cart.find(o => o.id === objectInCart.id).quantity += this.viewModel.productQuantity
        } else {
            this.viewModel.cart.push(objectInCart)
        }
        this.viewModel.productQuantity = 0
      },
      confirmDiscountAllOrNothing() {
        // check if the chosen offers for the discount list is initialized already or not
        if (!this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name] ||
          !this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name].length ||
          !(this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name] instanceof Array)) {
          this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name] = []
        }
        this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name] = this.viewModel.selectedDiscount.offers
          .map(offer => offer.productId)
        this.viewModel.selectedDiscountsStringified = JSON.stringify(this.viewModel.selectedDiscounts)
      },
      confirmDiscountOneOf() {
        console.log("hey")
        // check if the chosen offers for the discount list is initialized already or not
        if (!this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name] || 
          !this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name].length ||
          !(this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name] instanceof Array)) {
          this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name] = []
        }
        this.viewModel.selectedDiscounts[this.viewModel.selectedDiscount.name].push(this.viewModel.selectedOffer
          .productId);
        this.viewModel.selectedDiscountsStringified = JSON.stringify(this.viewModel.selectedDiscounts)
        console.log(this.viewModel.selectedDiscountsStringified)
      },
      clearOrderAndDiscounts() {
        this.viewModel.selectedDiscounts = new Map()
        this.viewModel.selectedProducts = new Map()
        this.viewModel.selectedDiscountsStringified = ''
        this.viewModel.selectedProductsStringified = ''
        this.viewModel.selectedDiscountsStringified = JSON.stringify(this.viewModel.selectedDiscounts)
        this.viewModel.cart = []
      },
      addProductToNewStore() {
        console.log('this.viewModel.chosenNewStoreProduct')
        console.log(this.viewModel.chosenNewStoreProduct)
        if (!this.viewModel.selectedProductsInNewStore.map(selected => selected.id).includes(this.viewModel
            .chosenNewStoreProduct.id)) {
          this.viewModel.selectedProductsInNewStore.push({
            id: this.viewModel.chosenNewStoreProduct.id,
            name: this.viewModel.chosenNewStoreProduct.name,
            price: this.viewModel.chosenNewStoreProduct.price
          })
          this.viewModel.selectedProductsInNewStoreForSend = JSON.stringify(this.viewModel.selectedProductsInNewStore)
        }
      },
      setNewStoreProductPrice(event) {
        this.viewModel.selectedNewProductInStoreToChangePrice.price = event.target.value
      },
      sumOffers() {
        this.viewModel.sumOfOffersAdditionalCost =
          this.viewModel.selectedDiscount.offers
          .map(offer => offer.additionalCost)
          .reduce((a, b) => a + b, 0)
      },
      stringifyDate() {
        const d = this.viewModel.transactionDate
        console.log(d)
        const dformat = [
          d.getDate(),
          d.getMonth() + 1,
          d.getFullYear()
        ] //.join('/') + ' ' + [d.getHours(),
          //d.getMinutes(),
          //d.getSeconds()
        //].join(':');
        this.viewModel.stringifiedTransactionDate = dformat
        console.log(this.viewModel.stringifiedTransactionDate)
      },
      onOrderDateChanged() {
        const d = this.viewModel.orderDate
        const dformat = [
          d.getDate(),
          d.getMonth() + 1,
          d.getFullYear()
        ] //.join('/') + ' ' + [d.getHours(),
          //d.getMinutes(),
          //d.getSeconds()
        //].join(':');
        this.viewModel.stringifiedOrderDate = dformat
      },
      connectToSocket() {
        if (!this.viewModel.loginResponse.uuid ||
          (this.viewModel.pushSocket && this.viewModel.pushSocket.readyState === WebSocket.OPEN)) {
          return;
        }
        this.viewModel.pushSocket = new WebSocket(
          `ws://localhost:8080/SDM/api/push/${this.viewModel.loginResponse.uuid}`)
        this.viewModel.pushSocket.onmessage = async () => {
          await this.getNotifications()
        }
      },
      async getNotifications() {
        this.viewModel.notifications = new Set()
        const beforePull = this.viewModel.notifications.size
        const notifications =
          await axios.get(
            `http://localhost:8080/SDM/api/users/notifications?uuid=${this.viewModel.loginResponse.uuid}`);
        notifications.data.allNotifications.forEach(notification => this.viewModel.notifications.add(notification))
        this.viewModel.notificationsArr = [...this.viewModel.notifications].map(notification => {
          return {
            'description': notification.description
          }
        })
        console.log(this.viewModel.notificationsArr)
        this.viewModel.numberOfNotifications = this.viewModel.notifications.size - beforePull
      },
      makeNotificationsZero() {
        this.viewModel.numberOfNotifications = undefined;
      },
      deleteChosenProductsForStore() {
        this.viewModel.selectedProductsInNewStore = []
      },
      removeProductFromCart() {
        console.log(this.viewModel.productToRemoveFromCart)
        delete this.viewModel.selectedDiscounts[this.viewModel.productToRemoveFromCart.id]
        this.viewModel.selectedProductsStringified = JSON.stringify(this.viewModel.selectedProducts)
        this.viewModel.cart = this.viewModel.cart.filter(product => product.id !== this.viewModel.productToRemoveFromCart.id)
      }
    }
  }
</script>