<template>
  <input type="text" v-model="value">
</template>
<script>
  import $ from 'jquery';
  import 'suggestions-jquery';
  import axios from 'axios';

  export default {
    props: {
      model: {
        required: true
      },
      location: {},
      oldLocation:{},
      kladr_id: 0,
      options: {
        type: Object,
        default: {
          type: 'ADDRESS',
          addon: 'none'
        }
      }
    },
    data() {
      return {
        value: '',
        location: {
          lat: '',
          lon: ''
        },
        kladr_id: 0,
      }
    },
    mounted() {
      this.callbacks = $.Callbacks();
      this.value = this.model;
      this.initSuggestion();
    },
    destroyed() {
      this.destroySuggestion();
    },
    watch: {
      location: {
        handler() {
          this.$emit('update:location', this.location);
        },
        deep: true
      },
      kladr_id() {
        this.$emit('update:kladr_id', this.kladr_id);
      },
      value() {
        this.$emit('update:model', this.value);
      },
      model() {
        this.value = this.model;
      },
      oldLocation: function(newVal) {
          this.location = newVal
          this.geolocate()
      }
    },
    methods: {
      initSuggestion() {
        this.callbacks.add(this.onSelect);
        this.callbacks.add(this.options.onSelect || $.noop)
        const options = Object.assign({}, this.options, {
          onSelect: suggestion => this.callbacks.fire(suggestion)
        });
        $(this.$el).suggestions(options);
      },
      destroySuggestion() {
        const plugin = $(this.$el).suggestions();
        plugin.dispose();
      },
      onSelect(suggestion) {
        this.value = suggestion.value;
        const { geo_lat, geo_lon, kladr_id } = suggestion.data;
        this.location.lat = geo_lat;
        this.location.lon = geo_lon;
        this.kladr_id = kladr_id;
      },
      geolocate() {
        var url = "https://suggestions.dadata.ru/suggestions/api/4_1/rs/geolocate/address";
        var token = this.options.token;
        var query = this.location;

        var options = {
            method: "POST",
            mode: "cors",
            headers: {
                "Content-Type": "application/json",
                "Accept": "application/json",
                "Authorization": "Token " + token
            },
            body: JSON.stringify(query)
        }

        fetch(url, options)
        .then(response => response.text())
        .then(result => this.onSelect(JSON.parse(result).suggestions[0]))
        .catch(error => console.log("error", error));
      }
    }
  };
</script>
