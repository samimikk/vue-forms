<template>
  <div v-show="visibility === false" :ref="containerId" :id="containerId" :class="{ 'form__element--date control-group' : defaultClass, 'error' : hasErrors }">
    <label class="control-group" :for="id" v-if="label"><span v-text="label"></span><span v-show="required" class="form--required">*</span></label>
    <div class="input-append">
      <flat-pickr
        v-model="date"
        :config="config"
        class="form-control"
        :placeholder="placeholder"
        name="date"
        :data-vv-name="plainTitle"
        :class="{'input': true, 'is-danger': errors.first(plainTitle)}"
        v-validate="validator"
        data-vv-value-path="innerValue"
        :has-error="errors.has(plainTitle)">
      ></flat-pickr>
      <span class="add-on"><i class="icon-calendar" aria-hidden="true"></i></span>
    </div>
    <div class="help-block">
      <span v-show="errors.first(plainTitle)" class="help is-danger" >{{ errors.first(plainTitle) }}</span>
    </div>
  </div>
</template>

<script>
import flatPickr from 'vue-flatpickr-component'
import 'flatpickr/dist/flatpickr.css'
import bus from '@/bus'
import EventBus from '@/EventBus'

export default {
  inject: ['$validator'],
  name: 'formInput',
  components: { flatPickr },
  props: {
    id: {
      type: String,
      required: true
    },
    containerId: {
      type: String,
      default: function () {
        return 'form__element--' + this.$props.id
      }
    },
    name: {
      type: String,
      required: true,
      default: null
    },
    plainTitle: {
      type: String,
      required: false,
      default: null
    },
    placeholder: {
      type: String,
      required: false,
      default: null
    },
    label: {
      type: String,
      required: false
    },
    items: {
      type: String,
      required: false,
      default: null
    },
    validate: {
      type: String,
      required: false,
      default: ''
    },
    validator: {
      type: String,
      required: false,
      default: null
    },
    dependency: {
      type: String,
      required: false
    },
    readOnly: {
      type: Boolean,
      default: false,
      required: false
    },
    hidden: {
      type: Boolean,
      required: false,
      default: false
    },
    errorText: {
      type: String,
      required: false
    },
    isChecked: {
      type: Boolean,
      required: false,
      default: false
    },
    expression: {
      type: String,
      required: false
    }
  },
  data: function () {
    return {
      key: '',
      value: '',
      date: '',
      config: {
        wrap: true, // set wrap to true only when using 'input-group'
        altFormat: 'd.m.Y',
        altInput: true,
        dateFormat: 'd.m.Y'
      },
      hasErrors: false,
      message: String,
      defaultClass: true,
      title: String,
      validatorRules: String,
      required: false,
      controlled: {
        controller: '',
        term: '',
        behaviour: 'hide'
      },
      controller: {
        target: [],
        term: ''
      },
      visibility: this.$props.hidden
    }
  },
  mounted () {

  },
  created () {
    bus.$on('validate', this.onValidate)

    // Create dependency rules
    if (this.dependency !== undefined) {
      let rule = this.dependency.split('|')
      this.controlled.controller = rule[0]
      this.controlled.term = rule[1]
      this.controlled.behaviour = rule[2]

      if (this.controlled.behaviour === 'show') {
        this.visibility = true
      }
      this.emitControllerRules(this.controlled.controller, this.controlled.term, this.id)
    }

    // Add element rules for controller
    EventBus.$on('control', (id, rule, target) => {
      if (this.id === id) {
        this.controller.target.push(target)
        this.controller.term = rule
      }
    })

    // Get control rules for controlled element
    EventBus.$on('controllable', (ids, term) => {
      for (let id of ids) {
        if (this.id === id) {
          if ((this.controlled.term === 'any' && term !== '') || term === this.controlled.term) {
            if (this.controlled.behaviour === 'show') {
              this.visibility = false
            } else {
              this.visibility = true
            }
          } else {
            this.visibility = !this.visibility
          }
        }
      }
    })

    if (this.validator !== null && this.validator.indexOf('required') !== -1) {
      this.required = true
    }
  },
  beforeDestroy () {
    bus.$off('validate', this.onValidate)
  },
  watch: {
    value: function (val, oldV) {
    },
    date: function (val, oldV) {
      this.$emit('update', {'key': this.id, 'value': val, 'label': this.label})
      this.value = val
      if (this.controller.target.length > 0) {
        this.emitControllerAction(this.controller.target, val)
      }
    }
  },
  computed: {},
  methods: {
    onValidate () {
      this.$validator.validateAll()
    },
    emitControllerRules (id, rule, target) {
      EventBus.$emit('control', id, rule, target)
    },
    emitControllerAction (id, term) {
      EventBus.$emit('controllable', id, term)
    }
  }
}
</script>
