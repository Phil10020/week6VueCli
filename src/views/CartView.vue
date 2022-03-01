<template>
  <div class="container">
    <h2>購物車</h2>
    <table class="table align-middle">
      <thead>
        <tr>
          <th>圖片</th>
          <th>商品名稱</th>
          <th>價格</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in products" :key="product.id">
          <td style="width: 200px">
            <div
              :style="{ backgroundImage: `url(${product.imageUrl})` }"
              style="
                height: 100px;
                background-size: cover;
                background-position: center;
              "
            ></div>
          </td>
          <td>
            {{ product.title }}
          </td>
          <td>
            <div class="h5" v-if="product.origin_price === product.price">
              {{ product.price }} 元
            </div>
            <div v-else>
              <del class="h6">原價 {{ product.origin_price }} 元</del>
              <div class="h5">現在只要 {{ product.price }} 元</div>
            </div>
          </td>
          <td>
            <div class="btn-group btn-group-sm">
              <button
                type="button"
                class="btn btn-outline-secondary"
                :disabled="isLoadingItem === product.id"
                @click="openProductModal(product.id)"
              >
                查看更多
              </button>
              <button
                :disabled="isLoadingItem === product.id"
                @click="addToCart(product.id)"
                type="button"
                class="btn btn-outline-danger"
              >
                <span
                  class="spinner-grow spinner-grow-sm"
                  v-show="isLoadingItem === product.id"
                ></span>
                加到購物車
              </button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="text-end">
      <button
        class="btn btn-outline-danger"
        type="button"
        @click="removeCartAll"
      >
        清空購物車
      </button>
    </div>
    <table class="table align-middle">
      <thead>
        <tr>
          <th></th>
          <th>品名</th>
          <th style="width: 150px">數量/單位</th>
          <th>單價</th>
        </tr>
      </thead>
      <tbody>
        <!-- 購物車資料是否存在 -->
        <template v-if="cartData.carts">
          <tr v-for="item in cartData.carts" :key="item.id">
            <td>
              <button
                type="button"
                class="btn btn-outline-danger btn-sm"
                @click="removeCartItem(item.id)"
              >
                x
              </button>
            </td>
            <td>
              {{ item.product.title }}
            </td>
            <td>
              <div class="input-group input-group-sm">
                <div class="input-group mb-3">
                  <!-- <input min="1" type="number" class="form-control" v-model="item.qty"> -->
                  <select
                    id=""
                    class="form-select"
                    v-model="item.qty"
                    @change="updateCartItem(item)"
                    :disabled="isLoadingItem === item.id"
                  >
                    <option
                      :value="num"
                      v-for="num in 20"
                      :key="`${num}-${item.id}`"
                    >
                      {{ num }}
                    </option>
                  </select>
                  <span class="input-group-text" id="basic-addon2">{{
                    item.product.unit
                  }}</span>
                </div>
              </div>
            </td>
            <td class="text-end">
              {{ item.total }}
            </td>
          </tr>
        </template>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3" class="text-end">總計</td>
          <td class="text-end">{{ cartData.total }}</td>
        </tr>
      </tfoot>
    </table>
  </div>
</template>

<script>
import emitter from '@/libs/emitter'
export default {
  data () {
    return {
      cartData: {},
      products: [],
      isLoadingItem: ''
    }
  },
  methods: {
    getProducts () {
      this.$http
        .get(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/products/all`
        )
        .then((res) => {
          // console.log(res);
          this.products = res.data.products
        })
    },
    // 參數預設值qty=1
    addToCart (id, qty = 1) {
      const data = {
        product_id: id,
        qty
      }
      this.isLoadingItem = id
      // 此api需要帶入參數格式{data}
      this.$http
        .post(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart`,
          { data }
        )
        .then((res) => {
          console.log(res)
          this.isLoadingItem = ''
          // get-cart
          emitter.emit('get-cart')
        })
    },
    getCart () {
      this.$http
        .get(`${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart`)
        .then((res) => {
          // console.log(res);
          this.cartData = res.data.data
        })
    },
    removeCartItem (id) {
      this.isLoadingItem = id
      this.$http
        .delete(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart/${id}`
        )
        .then(() => {
          // console.log(res);
          this.getCart()
          this.isLoadingItem = ''
          emitter.emit('get-cart')
        })
    },
    removeCartAll () {
      this.$http
        .delete(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/carts`
        )
        .then(() => {
          // console.log(res);
          this.getCart()
          emitter.emit('get-cart')
        })
    },
    updateCartItem (item) {
      const data = {
        product_id: item.id,
        qty: item.qty
      }
      this.isLoadingItem = item.id
      // 此api需要帶入參數格式{data}
      this.$http
        .put(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart/${item.id}`,
          { data }
        )
        .then(() => {
          // console.log(res);
          this.getCart()
          this.isLoadingItem = ''
        })
    }
  },
  mounted () {
    emitter.on('get-cart', () => {
      this.getCart()
    })
    this.getProducts()
    this.getCart()
  }
}
</script>
