<template>
  <div v-show="visibility === false" :ref="containerId" :id="containerId" :class="{ 'form__element--input control-group' : defaultClass, 'error' : hasErrors }">
    <div v-if="type === 'radio'">
      <legend v-text="label"></legend>
      <label :for="id+'_option_'+option.index" v-for="option in options" :key="option.index">
        <input :type="type" :id="id+'_option_'+option.index" :value="option.value" v-model="radioValue" /> {{option.text}}
      </label>
    </div>
    <div v-else-if="type === 'checkbox'">
      <legend v-if="multiple" v-text="label"></legend>
      <label v-if="multiple" :for="id+'_option_'+option.index" v-for="option in options" :key="option.index">
        <input :type="type" :id="id+'_option_'+option.index" :value="option.value" v-model="checkboxValues" /> {{option.text}}
      </label>
      <label v-if="!multiple" :for="id">
        <input :checked="!isChecked" :type="type" :id="id" :value="value" v-model="checkboxValue" /> {{label}}
      </label>
    </div>
    <div v-else>
      <label class="control-group" :for="id" v-if="label"><span v-text="label"></span><span v-show="required" class="form--required">*</span></label>
      <div :class="{ 'controls': true }">
        <input
          :type="type"
          :id="id"
          :name="name"
          :placeholder="placeholder"
          :data-vv-name="altTitle"
          v-validate="validator"
          :class="{'input': true, 'is-danger': errors.first(altTitle)}"
          :value="value"
          v-on="inputListeners"/>
      </div>
    </div>
    <div class="help-block">
      <span v-show="errors.first(altTitle)" class="help is-danger" >{{ errors.first(altTitle) }}</span>
    </div>
  </div>
</template>
<script>
import bus from '@/bus'
import EventBus from '@/EventBus'

export default {
  inject: ['$validator'],
  name: 'formInput',
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
    type: {
      type: String,
      required: true,
      default: 'text'
    },
    name: {
      type: String,
      required: true,
      default: null
    },
    altTitle: {
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
    required: {
      type: Boolean,
      required: false,
      default: false
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
      selected: '',
      options: [],
      multiple: false,
      radioValue: '',
      checkboxValues: [],
      checkboxValue: Boolean,
      hasErrors: false,
      message: String,
      defaultClass: true,
      title: String,
      validatorRules: String,
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
    if (this.type === 'checkbox' && !this.multiple ) {
      document.getElementById(this.id).checked = this.isChecked
    }
  },
  created () {

    bus.$on('validate', this.onValidate)

    // Populate items list
    if (this.$props.active !== undefined) {
      this.selected = this.$props.active
    }

    if (this.$props.items !== null) {
      this.multiple = true
      this.repeatableKey = 'radio_' + this.id
      let options = this.$props.items.split(',')
      let index = 0
      let value, text
      for (let option of options) {
        if (option.indexOf('|') !== -1) {
          const values = option.split('|')
          value = values[0]
          text = values[1]
        } else {
          value = option
          text = option
        }
        this.options.push({'text': text, 'value': value, 'index': index})
        index++
      }
    }
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
  },
  beforeDestroy () {
    bus.$off('validate', this.onValidate)
  },
  watch: {
    value: function (val, oldV) {
    },
    radioValue: function (val, oldV) {
      if (this.controller.target.length > 0) {
        this.emitControllerAction(this.controller.target, val)
      }
      this.$emit('update', {'value': val, 'key': this.id})
    },
    checkboxValues: function (val, oldV) {
      if (this.controller.target.length > 0) {
        this.emitControllerAction(this.controller.target, val)
      }
      this.$emit('update', {'value': val, 'key': this.id})
    },
    checkboxValue: function (val, oldV) {
      if (this.controller.target.length > 0) {
        this.emitControllerAction(this.controller.target, val)
      }
      this.$emit('update', {'value': val, 'key': this.id})
    }
  },
  computed: {
    inputListeners: function () {
      var vm = this
      // `Object.assign` merges objects together to form a new object
      return Object.assign({},
        // We add all the listeners from the parent
        this.$listeners,
        // Then we can add custom listeners or override the
        // behavior of some listeners.
        {
          // This ensures that the component works with v-model
          input: function (event) {

          },
          change: function (event) {

          },
          blur: function (event) {
            vm.$emit('update', {'value': event.target.value, 'key': vm.id})
            vm.value = event.target.value
            if (vm.controller.target.length > 0) {
              vm.emitControllerAction(vm.controller.target, event.target.value)
            }
          }
        }
      )
    }
  },
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
