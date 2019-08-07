<template>
  <div class="hello container">
    <h1>{{ msg }}</h1>

    <form>
      <div class="form-group row">
        <label for="jobTitle" class="col-sm-2 col-form-label">Job Title</label>
        <div class="col-sm-10">
          <input type="text" class="form-control" id="jobTitle" v-model="searchInput.jobTitle" />
        </div>
      </div>

      <div class="form-group row">
        <label for="companyName" class="col-sm-2 col-form-label">Company Name</label>
        <div class="col-sm-10">
          <input
            type="text"
            class="form-control"
            id="companyName"
            v-model="searchInput.companyName"
          />
        </div>
      </div>

      <div class="form-group">
        <label for="formControlRange">
          Recruit Me Score
          <span class="badge badge-primary">{{searchInput.score}}</span>
        </label>
        <input
          type="range"
          class="form-control-range"
          id="formControlRange"
          v-model="searchInput.score"
        />
      </div>

      <button type="button" class="btn btn-primary" v-on:click="mapSearch">Search</button>
    </form>

    <div class="row">
      <button type="button" class="btn btn-primary btn-lg" v-on:click="activateListView">List View</button>
      <button type="button" class="btn btn-secondary btn-lg" v-on:click="activateMapView">Map View</button>
    </div>

    <div v-if="this.mapView" class="row">
      <div class="col-sm-8">
        <GmapMap
          :center="{lat: 39.5, lng: -98.35}"
          :zoom="3"
          map-type-id="terrain"
          style="width: 750px; height: 500px"
        >
          <GmapMarker
            :key="index"
            v-for="(m, index) in this.searchResults"
            :position="{lat:parseInt(m.lat), lng:parseInt(m.lng)}"
            :clickable="true"
            @click="toggleInfoWindow(m)"
          />
        </GmapMap>
      </div>

      <div class="col-sm-4" v-if="contact">
        <div class="jumbotron jumbotron-fluid">
          <div class="container">
            <h4 class="display-10">{{contact.name}}</h4>
            <p class="lead">{{contact.jobTitle}} at {{contact.companyName}}</p>
          </div>
        </div>
      </div>
    </div>

    <div class="results" v-if="this.listView">
      <h5>
        Total Results:
        <span class="badge badge-secondary">{{this.searchResults.length}}</span>
      </h5>
      <div v-for="(item, index) in this.searchResults">
        <div
          v-bind:class="{'alert':true, 'alert-primary':(index % 2 != 0), 'alert-secondary':(index % 2 == 0) }"
          role="alert"
        >
          <h4>{{item.name}}</h4>
          <h5>{{item.jobTitle}} at {{item.companyName}}</h5>
          <h6>
            Phone: {{item.phone}}, Email: {{item.email}}
            <span
              class="badge badge-primary"
            >{{item.score}}</span>
          </h6>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data: function() {
    return {
      searchInput: {
        jobTitle: "",
        companyName: "",
        score: 50
      },
      searchResults: [],
      listView: true,
      mapView: false,
      contact: undefined
    };
  },
  methods: {
    search: function() {
      this.$http
        .post("http://localhost:8081/api/search", this.searchInput)
        .then(
          result => {
            this.searchResults = result.body;
          },
          error => {
            console.error(error);
          }
        );
    },
    mapSearch: function() {
      this.contact = undefined;
      this.$http
        .post("http://localhost:8081/api/mapSearch", this.searchInput)
        .then(
          result => {
            this.searchResults = result.body;
          },
          error => {
            console.error(error);
          }
        );
    },
    toggleInfoWindow: function(item) {
      console.log("Clicked Info Window");
      this.contact = item;
    },
    activateListView: function() {
      this.listView = true;
      this.mapView = false;
    },
    activateMapView: function() {
      this.mapView = true;
      this.listView = false;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.results {
  padding-top: 20px;
}

/* Set the size of the div element that contains the map */
#map {
  height: 400px; /* The height is 400 pixels */
  width: 100%; /* The width is the width of the web page */
}
</style>
