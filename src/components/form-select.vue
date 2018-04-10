<template>
  <div v-show="visibility === false" :ref="containerId" :id="containerId" :class="{ 'form__element--select control-group' : defaultClass, 'error' : hasErrors }">
    <label class="control-group" :for="id" v-if="label"><span v-text="label"></span><span v-show="required" class="form--required">*</span></label>
      <div :class="{ 'controls': true }">
        <select
          v-model="selected"
          :id="id"
          :name="name"
          v-validate="expression"
          :data-vv-name="altTitle"
          :class="{'input': true, 'is-danger': errors.has(title) }"
          :value="value"
          v-on="inputListeners"
        >
        <option v-for="option in options" :value="option.value" :key="option.index" >
          {{ option.text }}
        </option>
      </select>
      </div>
      <div class="help-block">
        <span v-show="errors.has(title)" class="help is-danger" >{{ errors.first(title) }}</span>
      </div>
    </div>
</template>

<script>
import bus from '@/bus'
import EventBus from '@/EventBus'

export default {
  inject: ['$validator'],
  name: 'formSelect',
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
    active: {
      type: String,
      required: false
    },
    items: {
      type: String,
      required: true
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
    validate: {
      type: String,
      required: false,
      default: ''
    },
    validator: {
      type: String,
      required: false,
      default: ''
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
    }
  },
  data: function () {
    return {
      key: '',
      value: '',
      selected: '',
      options: [],
      hasErrors: false,
      message: String,
      defaultClass: true,
      title: String,
      expression: Object,
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
    if (this.validator != null) {
      if (this.required) {
        this.expression = 'required|' + this.validator
      } else {
        this.expression = this.validator
      }
      if (this.altTitle !== null) {
        this.title = this.altTitle
      } else {
        this.title = this.name
      }
    }
  },
  created () {
    bus.$on('validate', this.onValidate)

    // Populate dropdown
    if (this.$props.active !== '') {
      this.selected = this.$props.active
    }

    if (this.$props.items !== null) {
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
    if (this.dependency != null) {
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
      if (val !== oldV) {

      }
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
          input: function (event) {},
          change: function (event) {
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
