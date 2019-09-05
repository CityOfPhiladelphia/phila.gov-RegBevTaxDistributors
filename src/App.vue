<template>
  <div id="bev-tax-app">
    <div class="search">
      <input
        id="search-bar"
        v-model="search"
        class="search-field"
        type="text"
        placeholder="Search by distributor name, address, or phone #"
        @keyup.enter="filter()"
      >
      <input
        ref="archive-search-bar"
        type="submit"
        class="search-submit"
        value="Search"
        @click="filter()"
      >
    </div>
  
    <div
      v-show="loading"
      class="mtm center"
    >
      <i class="fas fa-spinner fa-spin fa-3x" />
    </div>
    <div
      v-show="!loading && emptyResponse"
      class="h3 mtm center"
    >
      Sorry, there are no results.
    </div>
    <div
      v-show="failure"
      class="h3 mtm center"
    >
      Sorry, there was a problem. Please try again.
    </div>
    <div
      v-if="!loading && !emptyResponse && !failure"
      class="table-container"
    >
      <table
        v-if="filteredList.length"
       
      >
        <thead
          class="center"
          data-sticky
          data-top-anchor="filter-results:bottom"
          data-btm-anchor="page:bottom"
        >
          <tr>
            <th>
              <span>Distributor Name</span>
            </th>
            <th>
              <span>Address</span>
            </th>
            <th>
              <span>Phone</span>
            </th>
            <th>
              <span>Website</span>
            </th>
          </tr>
        </thead>
        <!-- <tbody> -->
        <paginate
          name="filteredList"
          :list="filteredList"
          class="paginate-list"
          tag="tbody"
          :per="50"
        >
          <tr
            v-for="distributor in paginated('filteredList')"
            :key="distributor.cartodb_id"
          >
            <td>
              {{ distributor.doing_business_as_name }}
            </td>
            <td>
              {{ distributor.street_address }}
              {{ distributor.city }}, {{ distributor.state }}
              {{ distributor.zip_code }}
            </td>
            <td>
              {{ distributor.phone_number }}
              <!-- check to see if there is an extension -->
               x{{ distributor.phone_extension }}
            </td>
            <td>
              {{ distributor.website }}
            </td>
          </tr>
        </paginate>
        <!-- </tbody> -->
      </table>
      <div class="app-pages">
        <p> Showing <b> {{ numberOf }} </b> distributors </p>
        <paginate-links
          v-show="!loading && !emptyResponse && !failure && displayPaginate"
          for="filteredList"
          :async="true"
          :limit="3"
          :show-step-links="true"
          :hide-single-page="false"
          :step-links="{
            next: 'Next',
            prev: 'Previous'
          }"
          @change="scrollToTop"
        />
      </div>
    </div>
  </div>
</template>
<script>

import Vue from "vue";
import axios from "axios";
import moment from "moment";
import VuePaginate from "vue-paginate";
import VueFuse from "vue-fuse";

Vue.use(VuePaginate);
Vue.use(VueFuse);

const endpoint =
  "https://phl.carto.com/api/v2/sql?q=SELECT%20*%20FROM%20beverage_tax_registration_data";

export default {
  name: "PhillyBevTaxDistributors",
  components: { 

  },
  filters: {
   
  },
  data: function() {
    return {
      distributorsList : [],
      filteredList: [],
      loading : true,
      emptyResponse : false,
      failure : false,
      displayPaginate: true,
      search: '',
      searchOptions: {
        threshold: 0.3,
        keys: [
          "doing_business_as_name",
          "street_address",
          "city",
          "state",
          "zip_code",
          "phone_number",
        ],
      },
      paginate: [ "filteredList" ],
    };
  },
  computed: { 
    numberOf: function() {
      return this.filteredList.length;
    },
  },

  watch: {
    search: function() {
      this.filter();
    },
  },

  created:  function() {
    this.getDistributors();
  },

  methods: {
    getDistributors: function() {
      {
        axios.get(endpoint)
          .then(response => {
            this.distributorsList = response.data.rows;
            this.filteredList = this.distributorsList;
            this.loading = false;
          }).catch(e => {
            window.console.log(e);
            this.failure = true;
            this.loading = false;
          });
      }
    },

    filter: async function() {

      if (this.search !== '') { // there is nothing in the search bar -> return everything in filteredPosts
        this.filteredList = [];
        
        this.$search(this.search, this.distributorsList, this.searchOptions).then(posts => {
          this.filteredList = posts;
        });
    
      } else {
        this.filteredList = this.distributorsList;
    
      }

      // await 
    },

    checkEmpty: function() {
      this.emptyResponse = (this.filteredList.length === 0 ? true : false);
      this.displayPaginate = (this.filteredList.length >= 50) ? true : false;
    },
    
    scrollToTop : function () {
      window.scrollTo({
        top: 0,
        behavior: 'smooth',
      });
    },
  },
};
</script>

<style lang="scss">

#bev-tax-app {
  width: 80%;
  margin: 10px auto;
  max-width: 1024px;

  .app-pages {
    display: flex;
    justify-content: space-between;
  }
}
</style>