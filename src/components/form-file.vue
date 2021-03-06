<template>
  <div v-show="visibility === false" :ref="containerId" :id="containerId" :class="{ 'form__element--input control-group' : defaultClass, 'error' : hasErrors }">
    <label class="control-group" :for="id" v-if="label"><span v-text="label"></span><span v-show="required" class="form--required">*</span></label>
    <div :class="{ 'controls': true }">
      <input
        type="file"
        :id="id"
        :name="name"
        :autocomplete="id"
        :data-vv-name="plainTitle"
        v-validate="validator"
        :class="{'input': true, 'is-danger': errors.first(plainTitle)}"
        v-on="inputListeners" />
    </div>
    <div class="help-block">
      <span v-show="errors.first(plainTitle)" class="help is-danger" >{{ errors.first(plainTitle) }}</span>
    </div>
    <div class="file__list">
        <div class="file__details" v-for="file in files" :key="file.index">
            {{ file.name }} <button @click="deleteFile(file.index)">X</button>
        </div>
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
    expression: {
      type: String,
      required: false
    }
  },
  data: function () {
    return {
      key: '',
      value: '',
      files: [],
      selected: '',
      options: [],
      multiple: false,
      hasErrors: false,
      message: String,
      defaultClass: true,
      title: '',
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
      visibility: this.$props.hidden,
      index: 0
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
            vm.files.push({'file': event.target.files[0], 'name': event.target.files[0].name, 'index': vm.index})

            if (vm.files.length > 0 || vm.files === undefined) {
              console.log('add files to form')
              vm.$emit('update', {'file': event.target.files[0], 'filename': event.target.files[0].name, 'key': vm.id, 'label': vm.label})
            }
            if (vm.controller.target.length > 0) {
              vm.emitControllerAction(vm.controller.target, event.target.files[0])
            }
            vm.index++
          },
          blur: function (event) {}

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
    },
    deleteFile: function (index) {
      this.files.splice(index, 1)
      this.index--
    }
  }
}

</script>
