<template>
  <div id="app">
    <form id="yja-forms" @submit.prevent autocomplete="off">

      <form-input
        @update="sync"
        id="firstname"
        type="text"
        label="First name"
        placeholder="First name"
        name="first_name"
        validator="alpha"
        required
      >
      </form-input>
      <form-input
        @update="sync"
        id="lastname"
        type="text"
        label="Last name"
        name="first_name"
        placeholder="Your last name"
        validator="alpha"
        altTitle="Last name"
        dependency="firstname|Sami|show"
        required
      >
      </form-input>
      <form-input
        @update="sync"
        id="title"
        type="text"
        label="Title"
        name="title"
        placeholder="Your title"
        dependency="lastname|any|show"
      >
      </form-input>
      <button @click="validateBeforeSubmit">Send</button>
    </form>
  </div>
</template>

<script>
import formInput from './components/form-input'
import Vue from 'vue'
import bus from '@/bus'

export default {
  $_veeValidate: {
    validator: 'new'
  },
  name: 'App',
  components: {
    formInput
  },
  data: function () {
    return {
      formData: {
        key: '',
        value: ''
      },
      hasErrors: false,
      bus: new Vue()
    }
  },
  methods: {
    validateBeforeSubmit () {
      // On button pressed run validation
      bus.$emit('validate')
      this.$validator.validateAll().then((result) => {
        if (result) {
          // eslint-disable-next-line
          console.log('Sending form: ' + this.formData)
          return
        }
        console.log('Form still contains errors')
      })
    },
    sync: function (args) {
      if (args.value !== undefined) {
        this.formData[args.key] = args.value
      }
    }
  },
  mounted () {
  },
  created () {
    bus.$on('errors-changed', (errors) => {
      this.errors.clear()
      errors.forEach((e) => {
        this.errors.add(e.field, e.msg, e.rule, e.scope)
      })
    })
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
