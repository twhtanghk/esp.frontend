<template>
  <card :header='"Internet " + status'>
    <v-select v-model='essid' label='ESSID' :items='list' filled :rules='[required($v.essid)]'/>
    <v-text-field v-model='password' label='Password' type='password' :rules='[required($v.password), minLength($v.password)]' required />
    <v-btn color="primary" @click='connect(essid, password)'>Connect</v-btn>
    <v-btn color="secondary" @click='getList()'>Scan</v-btn>
  </card>
</template>

<script lang='coffee'>
{Sta} = require('./plugins/model.coffee').default
import {required, minLength} from 'vuelidate/lib/validators'

export default
  components:
    card: require('./card').default
  data: ->
    config: {}
    essid: ''
    list: []
    password: ''
  validations:
    essid:
      required: required
    password:
      required: required
      minLength: minLength(8) 
  computed:
    status: ->
      if @config.isconnected then JSON.stringify(@config.curr) else ''
  methods:
    required: (attr) -> ->
      attr.required || 'Required'
    minLength: (attr, msg='At least #{attr.$params.minLength.min} characters') -> ->
      attr.minLength || msg
    getStatus: ->
      Sta.get()
        .then (res) =>
          @config = res
          @essid = res.essid
        .catch console.error
    connect: (essid, passwd) ->
      Sta
        .put 
          data:
            ssid: essid
            password: passwd
        .then =>
          @getStatus()
        .catch console.error
    getList: ->
      try
        @list = []
        for await i from Sta.iterAll url: "#{Sta.baseUrl}/scan"
          @list.push
            value: i
            text: i
      catch e
        console.error e
  mounted: ->
    @getStatus()
    @getList()
</script>
