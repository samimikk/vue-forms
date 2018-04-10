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
      >
      </form-input>
      <form-input
        @update="sync"
        id="radio"
        type="radio"
        label="Choose one"
        name="radio"
        items="Yksi,two|Kaksi,Kolme,Neljä"
        required
      >
      </form-input>
      <form-select
        @update="sync"
        id="Address"
        type="text"
        label="Your address"
        name="address"
        placeholder="Your address"
        altTitle="Your address"
        items="Yksi,two|Kaksi,Kolme,Neljä"
        active="Yksi"
      >
      </form-select>
      <button @click="validateBeforeSubmit">Send</button>
    </form>
  </div>
</template>

<script>
import formInput from './components/form-input'
import formSelect from './components/form-select'
import Vue from 'vue'
import bus from '@/bus'

export default {
  $_veeValidate: {
    validator: 'new'
  },
  name: 'App',
  components: {
    formInput,
    formSelect
  },
  data: function () {
    return {
      formData: {},
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
          let form = JSON.stringify(this.formData, null, 2)
          console.log('Sending form: ' + form)
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
  mounted () {},
  created () {}
}
</script>

<style>
#app {
  border: 1px dashed #ddd;
  padding: 2em;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
