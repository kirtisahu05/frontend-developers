<template>
  <button-full class="btn-spinner" @click.native="addToCart(product)" :disabled="isProductDisabled" data-testid="addToCart">
    <span>{{ addButtonText }}
      <spinner v-if="isProductDisabled" />
    </span>
  </button-full>
</template>

<script>
import { formatProductMessages } from '@vue-storefront/core/filters/product-messages'
import { notifications } from '@vue-storefront/core/modules/cart/helpers'
import focusClean from 'theme/components/theme/directives/focusClean'
import ButtonFull from 'theme/components/theme/ButtonFull.vue'
import { mapGetters, mapActions } from 'vuex'
import Spinner from 'theme/components/core/Spinner'

export default {
  directives: { focusClean },
  components: { ButtonFull, Spinner },
  props: {
    product: {
      required: true,
      type: Object
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      isAdded: false
    }
  },
  methods: {
    onAfterRemovedVariant () {
      this.$forceUpdate()
    },
    async addToCart (product) {
      try {
        const diffLog = await this.$store.dispatch('cart/addItem', { productToAdd: product })
        this.isAdded = true;
        this.openMicrocart();
      } catch (message) {
        this.notifyUser(notifications.createNotification({ type: 'error', message }))
      }
    },
    notifyUser (notificationData) {
      this.$store.dispatch('notification/spawnNotification', notificationData, { root: true })
    },
    ...mapActions({
      openMicrocart: 'ui/toggleMicrocart'
    })
  },
  computed: {
    ...mapGetters({
      isAddingToCart: 'cart/getIsAdding'
    }),
    isProductDisabled () {
      return this.disabled || formatProductMessages(this.product.errors) !== '' || this.isAddingToCart
    },
    addButtonText () {
      return this.isAdded ? 'Added' : 'Add to cart';
    }
  },
  beforeMount () {
    this.$bus.$on('product-after-removevariant', this.onAfterRemovedVariant)
  },
  beforeDestroy () {
    this.$bus.$off('product-after-removevariant')
  }
}
</script>

<style lang="scss" scoped>
.btn-spinner {
  background-color: black;
  span {
    display: inline-flex !important;
  }
}
</style>
