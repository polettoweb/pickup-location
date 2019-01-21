<template>
  <div id="app">
    <form id="form" name="pickupLocation">
      <h2>Let’s find your ideal car</h2>
      <fieldset id="location-fieldset">
          <div class="location-input">
            <label for="ftsAutocomplete">Pick-up Location</label>
            <input type="text" name="location" autocomplete="off" :value="location" @input="retrieveLocations" placeholder="city, airport, station, region, district…" aria-required="true" class="ui-autocomplete-input" role="textbox" aria-autocomplete="list" aria-haspopup="true">
          </div>
          <div class="location-list">
              <ul>
                  <li v-for="specificLocation in listOfResults" :key="specificLocation.bookingId" role="menuitem">
                      <a>
                          <span :class="`location-type-${specificLocation.placeType.toLowerCase()}`"></span>
                          <div class="location-name">
                              <p>
                              {{specificLocation.name}}
                              </p>
                              <span>{{specificLocation.region}}, {{specificLocation.country}}</span>
                          </div>
                    </a>
                    </li>
              </ul>
          </div>
      </fieldset>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      location: '',
      numberOfResults: 6,
      listOfResults: null,
      boldName: null,
      normalName: null
    }
  },
  methods: {
    retrieveLocations(event) {
      this.location = event.target.value;
      if (this.location.length > 2) {
        let url = `https://cors.io/?https://www.rentalcars.com/FTSAutocomplete.do?solrIndex=fts_en&solrRows=${this.numberOfResults}&solrTerm=${this.location}`;
        axios.get(url)
        .then(res => {
          this.listOfResults = res.data.results.docs;
        })
      }
      
    },
    nameSplit(value) {
          if (!value) return ''
          return value.replace(this.location.charAt(0).toUpperCase() + this.location.substr(1), "").trim();
      }
  }
}
</script>

<style lang="scss">
* {
  padding: 0;
  margin: 0;
}
html, body {
  font-family: Ubuntu, Helvetica, Arial, sans-serif;
}
#form {
    margin: 26px auto;
    padding: 26px;
    width: 50%;
    background: #F3CE56;
    background: linear-gradient(to bottom,  #f5d361 0%,#e7bf3b 100%);
    border-bottom: 2px solid #DAB129;

    fieldset {
      border: none;
    }
    .location-input {
          line-height: 30px;
          padding: 0 0 5px;

          label {
                color: #444;
                text-shadow: none;
                font-size: 0.875;
          }

          input {
                padding: 12px 8% 12px 2%;
                width: 90%;
                margin: 0;
                background: #FEFEFE;
                border-color: #a9a9a9;
                border-style: solid;
                border-width: 1px;
                font-size: 0.813em;
          }
    }
}
em {
    font-weight: 700;
    font-style: normal;
}
</style>
