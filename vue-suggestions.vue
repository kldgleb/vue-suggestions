<template>
  <input type="text" v-model="value">
</template>
<script>
  import $ from 'jquery';
  import 'suggestions-jquery';

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
          console.log("bd location: ", newVal)
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
        this.getCoords(suggestion.value)
      },
      getCoords(address) {
        var url = "https://suggestions.dadata.ru/suggestions/api/4_1/rs/suggest/address";
        var token = this.options.token;
        var body = {
          query: address
        };

        var options = {
          method: "POST",
          mode: "cors",
          headers: {
              "Content-Type": "application/json",
              "Accept": "application/json",
              "Authorization": "Token " + token
          },
          body: JSON.stringify(body)
        }
        
        fetch(url, options)
        .then( response => response.text())
        .then( result => {
          let resLat = JSON.parse(result).suggestions[0].data.geo_lat; 
          let resLon = JSON.parse(result).suggestions[0].data.geo_lon; 
          let resKladr = JSON.parse(result).suggestions[0].data.kladr_id; 
          let resArray = JSON.parse(result).suggestions;
          resArray.forEach(element => {
            if(element.data.value === address) {
              let resLat = element.data.geo_lat; 
              let resLon = element.data.geo_lon; 
              let resKladr = element.data.kladr_id;   
            }
          });
          this.location.lat = resLat; 
          this.location.lon =resLon; 
          this.kladr_id = resKladr; 
        })
        .catch(error => console.log("error", error));

      },
      geolocate() {
        var url = "https://suggestions.dadata.ru/suggestions/api/4_1/rs/geolocate/address";
        var token = this.options.token;
        let body = JSON.stringify(this.location);

        var options = {
            method: "POST",
            mode: "cors",
            headers: {
                "Content-Type": "application/json",
                "Accept": "application/json",
                "Authorization": "Token " + token
            },
            body: body 
        }
        fetch(url, options)
        .then(response => response.text())
        .then(result => {
          let resArray = JSON.parse(result).suggestions;
          let toOnSelect = JSON.parse(result).suggestions[0];
          resArray.forEach(element => {
            if(element.data.geo_lat === this.location.lat && element.data.geo_lon === this.location.lon) {
              toOnSelect = element
            }
          });
          this.value = toOnSelect.value;          
          this.location.lat = toOnSelect.data.geo_lat;          
          this.location.lon = toOnSelect.data.geo_lon;          
          this.kladr_id = toOnSelect.data.kladr_id; 
          }
        )
        .catch(error => console.log("error", error));
      }
    }
  };
</script>
