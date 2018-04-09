<template>
  <div :ref="containerId" :id="containerId" :class="{ 'form__element--input control-group' : defaultClass, 'error' : hasErrors }">
<!--
    <input v-validate="'required|email'" :class="{'input': true, 'is-danger': errors.has('email') }" name="email" type="text" placeholder="Email">
    <span v-show="errors.has('email')" class="help is-danger">{{ errors.first('email') }}</span>
-->
    <label class="control-group" :for="id"><span v:if="label" v-text="label"></span></label>
      <div :class="{ 'controls': true }">
        <input
          :type="type"
          :id="id"
          :name="name"
          :placeholder="placeholder"
          v-validate="expression"
          :data-vv-name="altTitle"
          :class="{'input': true, 'is-danger': errors.has(title) }"
          :value="value"
          v-on="inputListeners"
          />
          <span v-show="errors.has(title)" class="help is-danger" >{{ errors.first(title) }}</span>
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
    visibility: {
      type: Boolean,
      default: true,
      required: false
    },
    label: {
      type: String,
      required: false
    },
    labelVisibility: {
      type: Boolean,
      default: true,
      required: false
    },
    validate: {
      type: String,
      required: false
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
    }
  },
  data: function () {
    return {
      key: '',
      value: '',
      hasErrors: false,
      message: String,
      defaultClass: true,
      title: String,
      expression: Object,
      controllerId: String,
      controllerTerm: String
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
    this.$watch(() => this.errors.errors, (value) => {
      bus.$emit('errors-changed', value)
    })
  },
  beforeDestroy () {
    bus.$off('validate', this.onValidate)
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
            vm.$emit('update', {'value': event.target.value, 'key': vm.id})
            if (vm.dependency != null) {
              let rule = vm.dependency.split('|')
              vm.controllerId = rule[0]
              vm.controllerTerm = rule[1]
              vm.emitMethod(vm.id, event.target.value, vm.controllerId, vm.controllerTerm)
            }

            EventBus.$on('controller', toggle => {
              console.log('Toggle is: ' + toggle)
            })
          }
        }
      )
    }
  },
  methods: {
    onValidate () {
      this.$validator.validateAll()
    },
    emitMethod (component, id, term, value) {
      console.log(component, id, term, value)
      let toggle = Boolean(false)
      if (component === id) {
        if ((term === 'any' && value !== '') || term === value) {
          toggle = Boolean(true)
        }

        return toggle
      }
      EventBus.$emit('controller', toggle)
    }
  }
}
</script>
