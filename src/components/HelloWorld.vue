<template>
  <div class="hello container-fluid">
    <img class="logo" src="../assets/recruitMeLogo.png" />

    <form>
      <div class="form-group row">
        <label for="jobTitle" class="col-sm-3">Job Title</label>
        <div class="col-sm-8">
          <input type="text" class="form-control" id="jobTitle" v-model="searchInput.jobTitle"
            v-debounce:300ms="search" />
        </div>
      </div>

      <div class="form-group row"><label for="companyName" class="col-sm-3">Company Name</label>
        <div class="col-sm-8">
          <input type="text" class="form-control" id="companyName" v-model="searchInput.companyName"
            v-debounce:300ms="search" />
        </div>
      </div>

      <div class="form-group">
        <label for="formControlRange">
          <h5>
            Recruit Me Score
            <span class="badge badge-primary">{{searchInput.score}}</span>
          </h5>
        </label>
        <input type="range" class="form-control-range" id="formControlRange" v-model="searchInput.score"
          @change="search" />
      </div>

      <button type="button" class="btn btn-primary btn-lg" @click="search">Search</button>

    </form>

    <div v-if="this.searchResults.length > 0">
      <h5>
        Total Results:
        <span class="badge badge-secondary">{{this.searchResults.length}}</span>
      </h5>

      <div class="row">
        <button type="button" class="btn btn-primary btn-lg col-sm-6" v-on:click="activateListView">List View</button>
        <button type="button" class="btn btn-danger btn-lg col-sm-6" v-on:click="activateMapView">Map View</button>
      </div>
    </div>

    <div class="results">
      <div v-if="this.mapView" class="row">
        <div class="col-lg-9">
          <GmapMap :center="{lat: 39.5, lng: -98.35}" :zoom="5" map-type-id="terrain"
            style="width: 1870px; height: 700px">
            <gmap-info-window v-if="mapData.contact" :position="mapData.infoWindowPos" :opened="mapData.infoWinOpen"
              @closeclick="mapData.infoWinOpen=false">

              <h4 class="text-dark">{{mapData.contact.name}}</h4>
              <h5><span class="text-primary">{{mapData.contact.jobTitle}}</span> at <span
                  class="text-info">{{mapData.contact.companyName}}</span></h5>
              <h6 class="text-dark">
                Phone: {{mapData.contact.phone}}, Email: {{mapData.contact.email}}
                <span class="badge badge-primary">{{mapData.contact.score}}</span>
              </h6>

            </gmap-info-window>

            <GmapMarker :key="index" v-for="(m, index) in this.mapResults" :position="getPosition(m, index)"
              :clickable="true" :icon="{url: getMarkerColor(m.score)}" @click="toggleInfoWindow(m, index)"
              :label="'`'" />
          </GmapMap>
        </div>

      </div>

      <div v-if="this.listView">

        <div v-if="this.searchResults.length > 0">
          <table class="table table-striped">
            <thead class="thead-dark">
              <tr>
                <th>Name</th>
                <th>Title</th>
                <th>Company</th>
                <!-- <th>Avg time in position</th>
                <th>Changed job last year</th>
                <th>Job changes in last 10 years</th> -->
                <th>Contact Info</th>
                <th>RecuitMe Score</th>
              </tr>
            </thead>
            <tbody>
              <tr :key="index" v-for="(item, index) in this.searchResults">
                <td class="text-dark"><b>{{item.name}}</b></td>
                <td class="text-primary"><b>{{item.jobTitle}}</b></td>
                <td>{{item.companyName}}</td>
                <!-- <td>{{item.avgTimeInPosition}}</td>
                <td>{{item.changedAJobLastYear}}</td>
                <td>{{item.numJobChangesLast10Years}}</td> -->
                <td class="text-info"><b>{{item.phone}}, {{item.email}}</b></td>
                <td>
                  <h4><span class="badge badge-primary">{{item.score}}</span></h4>
                </td>
              </tr>
            </tbody>
          </table>
        </div>


        <!-- <div v-bind:class="{'alert':true, 'alert-primary':(index % 2 != 0), 'alert-danger':(index % 2 == 0) }"
          role="alert">
          <h4>{{item.name}}</h4>
          <h5>{{item.jobTitle}} at {{item.companyName}}</h5>
          <h6>
            Phone: {{item.phone}}, Email: {{item.email}}
            <span class="badge badge-primary">{{item.score}}</span>
          </h6>
        </div> -->
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
    data: function () {
      return {
        searchInput: {
          jobTitle: "",
          companyName: "",
          score: 50
        },
        searchResults: [],
        mapResults: [],
        listView: true,
        mapView: false,
        mapData: {
          infoContent: "",
          infoWindowPos: null,
          infoWinOpen: false,
          currentMidx: null,
          contact: null
        }
      };
    },
    methods: {
      search: function () {
        this.$http
          .post("http://localhost:8081/api/search", this.searchInput)
          .then(
            result => {
              this.searchResults = result.body;
              this.mapSearch();
            },
            error => {
              console.error(error);
            }
          );
      },
      mapSearch: function () {
        this.contact = undefined;
        this.mapResults = [];
        for (var value of this.searchResults) {
          if (value.lat && value.lng) this.mapResults.push(value);
        }
        console.log(this.searchResults.length, this.mapResults.length);
      },
      toggleInfoWindow: function (marker, idx) {
        this.mapData.infoWindowPos = {
          lat: parseInt(marker.lat),
          lng: parseInt(marker.lng)
        };
        this.mapData.contact = marker;

        //check if its the same marker that was selected if yes toggle
        if (this.mapData.currentMidx == idx) {
          this.mapData.infoWinOpen = !this.mapData.infoWinOpen;
        }
        //if different marker set infowindow to open and reset current marker index
        else {
          this.mapData.infoWinOpen = true;
          this.mapData.currentMidx = idx;
        }
      },
      activateListView: function () {
        this.listView = true;
        this.mapView = false;
      },
      activateMapView: function () {
        this.mapView = true;
        this.listView = false;
      },
      getMarkerColor: function (score) {
        if (score <= 70)
          return "http://maps.google.com/mapfiles/ms/icons/red-dot.png";
        if (score > 70 && score <= 90)
          return "http://maps.google.com/mapfiles/ms/icons/yellow-dot.png";
        if (score > 90)
          return "http://maps.google.com/mapfiles/ms/icons/green-dot.png";
      },
      getPosition: function (m, index) {
        // console.log("getPosition", index, m.score);
        return { lat: parseInt(m.lat), lng: parseInt(m.lng) };
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
    height: 100%;
    width: 100%;
  }

  .badge-primary {
    padding: 3px;
    margin-left: 3px;
  }

  .logo {
    width: 25%;
  }
</style>