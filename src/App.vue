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

        <!-- TEXTAREA -->
        <form-textarea
          @update="sync"
          id="feedback"
          label="Anna palautetta"
          name="feeback"
          plainTitle="palautetta"
          dependency="useselect|true|show"
        ></form-textarea>

        <!-- FILE UPLOAD -->
        <form-file
          @update="sync"
          id="files"
          label="Lisää liite"
          name="attachments"
          plainTitle="liite"
          validator="required|image"
        ></form-file>
      <button @click="validateBeforeSubmit">Send</button>
    </form>
  </div>
</template>

<script>
import formInput from './components/form-input'
import formSelect from './components/form-select'
import formDate from './components/form-date'
import formTextarea from './components/form-textarea'
import formFile from './components/form-file'
import Vue from 'vue'
import bus from '@/bus'



export default {
  name: 'App',
  components: {
    formInput,
    formSelect,
    formDate,
    formTextarea,
    formFile
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
          let form = this.appendToForm(this.formData)
          console.log('formData contains: ', JSON.stringify(this.formData, null, 2))
          console.log('form contains: ', ...form)
          return
        }
        console.log('Form still contains errors')
      })
    },
    sync: function (args) {
      // Check if is new entry
      let isNew = this.checkForExisting(args.key, this.formData)
      if (isNew === false) {
        this.formData = this.removeDataInstance(args.key)
      }
      if (args.value !== undefined) {
        this.formData.push({id: args.key, value: args.value, label: args.label})
      }
      if (args.value === undefined && args.files !== null) {
        this.formData.push({id: args.key, file: args.file, filename: args.filename, label: args.label})
      }
    },
    removeDataInstance (id, value) {
      const data = this.formData
      for (var key in data) {
        if (!data.hasOwnProperty(key)) {
          continue
        } else {
          var obj = data[key]
          for (var prop in obj) {
            if (!obj.hasOwnProperty(prop)) {
              continue
            } else {
              if (prop === 'id') {
                if (obj[prop] === id) {
                  delete data[key]
                  return data
                }
              }
            }
          }
        }
      }
      return data
    },
    checkForExisting (id, data) {
      let result = Boolean(true)
      for (var key in data) {
        if (!data.hasOwnProperty(key)) {
          continue
        } else {
          var obj = data[key]
          for (var prop in obj) {
            if (!obj.hasOwnProperty(prop)) {
              continue
            } else {
              if (prop === 'id') {
                if (obj[prop] === id) {
                  result = Boolean(false)
                }
              }
            }
          }
        }
      }
      return result
    },
    appendToForm (s) {
      let t = new FormData()
      for (var key in s) {
        let pKey = ''
        let pValue = ''
        // skip loop if the property is from prototype
        if (!s.hasOwnProperty(key)) {
          continue
        } else {
          var obj = s[key]
          for (var prop in obj) {
            // skip loop if the property is from prototype
            if (!obj.hasOwnProperty(prop)) {
              continue
            } else {
              if (prop === 'label') {
                pKey = obj[prop]
              }
              if (prop === 'value' || prop === 'file') {
                pValue = obj[prop]
              }
              if (pKey !== '' && pValue !== '') {
                t.append(pKey, pValue)
              }
            }
          }
        }
      }
      return t
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
