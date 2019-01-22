<template>
    <div id="app">
        <form id="form" name="pickupLocation">
            <h2>Let’s find your ideal car</h2>
            <fieldset id="location-fieldset">
                <div class="location-input">
                    <label for="location">Pick-up Location</label>
                    <input
                        type="text"
                        name="location"
                        autocomplete="off"
                        :value="location"
                        @input="retrieveLocations"
                        @focus="searchActive = true"
                        placeholder="city, airport, station, region, district…"
                        class="ui-autocomplete-input"
                        role="textbox"
                        aria-required="true"
                        aria-autocomplete="list"
                        aria-haspopup="true"
                    >
                </div>
                <div class="location-list" v-show="searchActive">
                    <ul>
                        <li
                        v-for="specificLocation in listOfResults"
                        :key="specificLocation.bookingId"
                        role="menuitem"
                        tabindex="0"
                        :class="{'location-no-results': numberInList > 0 &&  specificLocation.bookingId === undefined}"
                        @keyup.enter="selectLocation(specificLocation.name,hasIata(specificLocation.iata), specificLocation.region, specificLocation.country)"
                        >
                            <a v-if="numberInList > 0 &&  specificLocation.bookingId === undefined" @click="searchActive = false">
                                <div class="location-name">
                                    <p>{{specificLocation.name}}</p>
                                </div>
                            </a>
                            <a v-else @click="selectLocation(specificLocation.name,hasIata(specificLocation.iata), specificLocation.region, specificLocation.country)">
                                <span :class="`location-type location-type${specificLocation.placeType ? `-${specificLocation.placeType.toLowerCase()}` : ''}`"></span>
                                <div class="location-name">
                                    <p v-html="highlight(specificLocation.name,hasIata(specificLocation.iata))"></p>
                                    <span class="location-region-country">
                                        {{specificLocation.region}}, {{specificLocation.country}}
                                    </span>
                                    <!-- couldn't find any result with isPopular = true.
                                    for testing it please change to v-if="!specificLocation.isPopular"-->
                                    <span class="location-popular" v-if="specificLocation.isPopular">Popular</span>
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
import axios from "axios";

export default {
    data() {
        return {
            location: "", //used as model for input value
            numberOfResults: 6, //number of max results for the API call
            listOfResults: null, //storing the result of the API
            debounceTimer: null, //used for debouncing the API call
            numberInList: 0, //used for checking if the API returns more than 1 result
            searchActive: false //opening and closing the result panel
        };
    },
    methods: {
        //method associated with @input
        retrieveLocations(event) {
            this.location = event.target.value;
            const url = `https://cors.io/?https://www.rentalcars.com/FTSAutocomplete.do?solrIndex=fts_en&solrRows=${this.numberOfResults}&solrTerm=${this.location}`;

            //calling the API only in case the search is greater than 1 char
            if (this.location.length > 1) {
                this.searchActive = true;

                //debouncing the search for avoiding eccessive numbers of calls to the API
                clearTimeout(this.debounceTimer);
                this.debounceTimer = setTimeout(() => {
                    axios.get(url).then(res => {
                        this.listOfResults = res.data.results.docs;

                        this.numberInList = res.data.results.docs.length;
                    });
                }, 300);
            } else {
                //resetting the search and close the result panel
                this.listOfResults = null;
                this.searchActive = false;
            }
        },
        //method used for matching user search and result with a bold text and appending the IATA if applicable
        highlight(string, append) {
            if (!this.location) return;
            //string replace location with result checking all matches with insensitive case
            const locationFirstPart = string.replace(
                new RegExp(this.location, "gi"),
                match => "<em>" + match + "</em>"
            );
            return locationFirstPart + " " + append;
        },
        //checking if result has IATA or not
        hasIata(string) {
            return string !== undefined ? `(${string})` : "";
        },
        //on location selected, filling the input with correct data
        selectLocation(name, iata, region, country) {
            this.location = iata === "" ? `${name}, ${region}, ${country}` : `${name} ${iata}, ${region}, ${country}`;
            this.searchActive = false; //closing the result panel
        }
    }
};
</script>

<style lang="scss">
    @import "./assets/scss/main.scss";
</style>
