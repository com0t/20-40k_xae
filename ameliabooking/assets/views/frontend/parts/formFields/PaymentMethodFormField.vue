<template>
    <el-form-item
      v-if="totalPrice > 0 && !this.$root.settings.payments.wc.enabled && paymentOptions.length > 1"
      class="am-payment-buttons-wrapper"
      :class="$root.settings.customization.forms ? `am-select-${classIdentifier}`: ''"
    >
      <template slot="label">
        <span :style="{fontWeight: 700}">
          {{ labelPaymentMethod || $root.labels.payment_method_colon }}
        </span>
      </template>
      <el-button
          v-for="item in paymentOptions"
          :key="item.value"
          :value="item.value"
          @click="changeItem(item.value)"
          class="am-payment-button"
          :class="appointment.payment.gateway === item.value ? 'am-payment-button-selected':''"
      >
        <payment-button-inner
          :value="item.value.toLowerCase()"
          :color="appointment.payment.gateway === item.value ? $root.settings.customization.primaryColor : ''"
        >
        </payment-button-inner>
      </el-button>
    </el-form-item>
</template>

<script>
import paymentButtonInner from '../../../parts/PaymentButtonInner'

export default {
  name: 'paymentMethodFormField',

  components: {
    paymentButtonInner
  },

  props: {
    totalPrice: {
      type: String
    },
    bookableColor: {
      type: String
    },
    paymentOptions: {
      type: Array,
      default: () => []
    },
    appointment: {
      type: Object,
      default: () => {}
    },
    classIdentifier: {
      type: String,
      default: ''
    },
    formField: {
      type: Object,
      default: () => {}
    }
  },

  data () {
    return {
      labelPaymentMethod: this.formField[this.$options.name].labels.payment_method_colon.value
    }
  },

  methods: {
    changeItem (value) {
      this.appointment.payment.gateway = value
      this.$emit('changeItem')
    }
  }
}
</script>