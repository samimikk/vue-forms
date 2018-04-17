<template>
  <div id="app">
    <form id="yja-forms" @submit.prevent autocomplete="off">
      <form-input
        @update="sync"
        id="firstname"
        type="text"
        label="First name"
        name="firstname"
        validator="required|alpha"
        plainTitle="first name"
      >
      </form-input>
      <form-input
        @update="sync"
        id="lastname"
        type="text"
        label="Last name"
        name="lastname"
        plainTitle="last name"
        validator="required"
      >
      </form-input>
      <form-input
        @update="sync"
        id="title"
        type="text"
        label="Title"
        name="title"
      >
      </form-input>
      <form-input
        @update="sync"
        id="radio"
        type="radio"
        label="Choose one"
        name="radio"
        items="Yksi,two|Kaksi,Kolme,Neljä"
      ></form-input>
      <form-input
        @update="sync"
        id="checkbox"
        type="checkbox"
        label="Choose one or many"
        name="checkbox"
        items="Yksi,two|Kaksi,Kolme,Neljä"
      ></form-input>
      <form-input
        @update="sync"
        id="useselect"
        type="checkbox"
        label="Want to to select?"
        name="checkbox"
      ></form-input>

      <form-select
        @update="sync"
        id="Address"
        type="text"
        label="Your address"
        name="address"
        plainTitle="Your address"
        items="Yksi,two|Kaksi,Kolme,Neljä"
        dependency="useselect|true|show"
        active="Yksi"
      >
      </form-select>
      <form-date
        @update="sync"
        id="date"
        label="Choose date"
        name="date"
        plainTitle="päivämäärä"
        placeholder="Valitse päivämäärä"
        validator="required"
        ></form-date>

        <form-textarea
          @update="sync"
          id="feedback"
          label="Anna palautetta"
          name="feeback"
          plainTitle="palautetta"
        ></form-textarea>

      <button @click="validateBeforeSubmit">Send</button>
    </form>
  </div>
</template>

<script>
import formInput from './components/form-input'
import formSelect from './components/form-select'
import formDate from './components/form-date'
import formTextarea from './components/form-textarea'
import Vue from 'vue'
import bus from '@/bus'

export default {
  $_veeValidate: {
    validator: 'new'
  },
  name: 'App',
  components: {
    formInput,
    formSelect,
    formDate,
    formTextarea
  },
  data: function () {
    return {
      formData: [],
      hasErrors: false,
      bus: new Vue(),
      date: null
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
        this.formData.push({id: args.key, value: args.value, label: args.label})
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
