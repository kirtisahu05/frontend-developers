<template>
  <div class="product-quantity-mobile">
    <base-select
      class="col-xs-12 col-sm-6 mb10"
      name="name"
      :options="getOptions"
      :selected="value"
      :placeholder="$t('Quantity *')"
      :validations="[
        {
          condition: value.$error && !value.required,
          text: $t('Field is required')
        }
      ]"
      :value="value"
      @input="$emit('input', $event)"
      autocomplete="name"
      @blur="$v.value.$touch()"
      @change.native="$v.value.$touch();"
    />
    <spinner v-if="loading" />
  </div>
</template>

<script>
import { minValue, maxValue, numeric, required } from 'vuelidate/lib/validators'
import { onlineHelper } from '@vue-storefront/core/helpers'
import Spinner from 'theme/components/core/Spinner'
import BaseSelect from 'theme/components/core/blocks/Form/BaseSelect'

export default {
  components: {
    Spinner,
    BaseSelect
  },
  props: {
    value: {
      type: [Number, String],
      required: true
    },
    showQuantity: {
      type: Boolean,
      default: false
    },
    maxQuantity: {
      type: Number,
      default: undefined
    },
    checkMaxQuantity: {
      type: Boolean,
      default: true
    },
    loading: {
      type: Boolean,
      default: false
    },
    isSimpleOrConfigurable: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    isOnline (value) {
      return onlineHelper.isOnline
    },
    max () {
      if (!this.isOnline || !this.isSimpleOrConfigurable) {
        return null
      }

      return this.maxQuantity
    },
    disabled () {
      if (!this.isOnline) {
        return false
      }
      return !this.maxQuantity && this.checkMaxQuantity && this.isSimpleOrConfigurable
    },
    name () {
      if (this.isSimpleOrConfigurable && !this.loading && this.showQuantity) {
        return this.$i18n.t(this.isOnline ? 'Quantity available' : 'Quantity available offline', { qty: this.maxQuantity })
      }
      return this.$i18n.t('Quantity')
    },
    getOptions () {
      const value = [];
      if (this.max) {
        for (let index = 1; index < this.max; index++) {
          value.push(index);
        }
      }

      return value;
    }
  },
  validations () {
    return {
      value: {
        minValue: minValue(1),
        maxValue: maxValue(this.max) && !this.isSimpleOrConfigurable,
        numeric,
        required
      }
    }
  },
  watch: {
    '$v.$invalid' (error) {
      this.$emit('error', error)
    }
  }
}
</script>
<style lang="scss" scoped>
.product-quantity-mobile {
  position: relative;
  /deep/ .spinner {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
  }
}
</style>
