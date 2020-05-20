<template>
  <v-layout row wrap>
    <card header='Current Duty'>
      <v-slider
        :label="label(pin, curr)"
        v-model="curr"
        :min="range[0]"
        :max="range[1]"
        @change='setDuty()'/>
    </card>
    <card header='Config'>
      <v-text-field ref='duty' type='number' v-model.number='duty' label='Duty' :rules='[required, integer, between(range[0], range[1])]' />
      <v-text-field ref='freq' type='number' v-model.number='freq' label='Freq' :rules='[required, integer, between(0, 1000)]'/>
      <v-text-field ref='min' type='number' v-model.number='range[0]' label='Min' :rules='[required, integer, between(0, 1023)]'/>
      <v-text-field ref='max' type='number' v-model.number='range[1]' label='Max' :rules='[required, integer, between(0, 1023)]'/>
      <v-btn color='primary' @click='setConfig' :disabled='! valid()'>Save</v-btn>
    </card>
  </v-layout>
</template>

<script lang='coffee'>
{Pwm} = require('./plugins/model.coffee').default
import {required, integer, between} from 'vuelidate/lib/validators'

export default
  components:
    card: require('./card').default
  data: ->
    pin: 2
    curr: 0
    # boot settings
    duty: 0
    freq: 50
    range: [50, 100]
  methods:
    valid: ->
      _.every _.map @$refs, 'valid'
    required: (val) ->
      required(val) || 'Required'
    integer: (val) ->
      integer(val) || 'Integer only'
    between: (min, max) ->
      (val) =>
        between(min, max)(val) || "Valid range [#{@range[0]}, #{@range[1]}]"
    label: (pin, curr) ->
      "Pin #{pin} (#{curr})"
    setDuty: ->
      try
        await Pwm.update
          data:
            id: @pin
            duty: @curr
      catch err
        console.error err
    getCurr: (val) ->
      try
        {duty} = await Pwm.read
          data:
            id: @pin
        @curr = duty
      catch err
        console.error err
    setConfig: ->
      try
        await Pwm.put
          url: "#{process.env.API_URL}/pwm/init/#{@pin}"
          data:
            duty: @duty
            freq: @freq
            range: @range
      catch err
        console.error err
    getConfig: ->
      try
        {duty, freq, range} = await Pwm.get
          url: "#{process.env.API_URL}/pwm/init/#{@pin}"
        @duty = duty
        @freq = freq
        @range[0] = range[0]
        @range[1] = range[1]
      catch err
        console.error err
  mounted: ->
    await @getConfig()
    await @getCurr()
</script>
