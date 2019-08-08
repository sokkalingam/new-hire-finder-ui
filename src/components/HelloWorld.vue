<template>
  <div class="container-contact100">
    <div class="wrap-contact100">

      <div class="hello container-fluid">
        <img class="logo" src="../assets/recruitMeLogo.png" />

        <form class="contact100-form">
          <div class="wrap-input100 validate-input bg1 rs1-wrap-input100" data-validate="Please enter Job Title">
            <span class="label-input100">Job Title</span>
            <input
                    class="input100"
                    type="text"
                    name="jobTitle"
                    id="jobTitle"
                    placeholder="Enter Job Title"
                    v-model="searchInput.jobTitle"
                    v-debounce:300ms="search"
            />
          </div>

          <div class="wrap-input100 bg1 rs1-wrap-input100" data-validate="Please enter Company Name">
            <span class="label-input100">Company Name</span>
            <input
                    class="input100"
                    type="text"
                    name="companyName"
                    id="companyName"
                    placeholder="Enter Company Name"
                    v-model="searchInput.companyName"
                    v-debounce:300ms="search"
            />
          </div>

          <div class="wrap-contact100-form-range">
            <label for="filter-bar" class="label-input200">
              Recruit Me Score
              <span class="badge badge-primary">{{searchInput.score}}</span>
            </label>
            <div class="contact100-form-range-bar">
              <input
                      type="range"
                      class="form-control-range contact100-form-range-bar"
                      id="filter-bar"
                      v-model="searchInput.score"
                      @change="search"
              />
            </div>
          </div>

          <div class="container-contact100-form-btn">
            <button class="contact100-form-btn" type="button" @click="mapSearch">
              <span>Search</span>
            </button>
          </div>

      </form>

        <hr class="seperator">
        <div v-if="this.searchResults.length > 0">
        <h5>
          Total Results:
          <span class="badge badge-primary">{{this.searchResults.length}}</span>
        </h5>

        <div class="row">
          <button type="button" class="width-50" v-on:click="activateListView">List View</button>
          <button type="button" class="width-50" v-on:click="activateMapView">Map View</button>
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
            <div class="alert alert-light" :key="index" v-for="(item, index) in this.searchResults">
              <h4 class="display-name">{{item.name}} <span class="title"> - {{item.jobTitle}} at {{item.companyName}}</span> <span class="badge badge-primary float-right">{{item.score}}</span></h4>
              <hr>
              <dl class="row">
                <dd class="col-sm-6 font-weight">{{item.phone}}</dd>
                <dd class="col-sm-6 font-weight">{{item.email}}</dd>

                <dt class="col-sm-4 smaller"> Average time in Position: </dt>
                <dd class="col-sm-8 smaller">{{item.avgTimeInPosition}}</dd>
                <dt class="col-sm-4 smaller"> Changed a job last year: </dt>
                <dd class="col-sm-8 smaller">{{item.changedAJobLastYear}}</dd>
                <dt class="col-sm-4 smaller"> # of jobs changed in last 10 years: </dt>
                <dd class="col-sm-8 smaller">{{item.numJobChangesLast10Years}}</dd>
              </dl>
            </div>

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
  }

  .badge {
    padding: 3px;
    margin-left: 3px;
    background-color: #FF7A59;
  }

  /*//////////////////////////////////////////////////////////////////
  [ RESTYLE TAG ]*/

  * {
    margin: 0px;
    padding: 0px;
    box-sizing: border-box;
  }

  body, html {
    height: 100%;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
  }

  /*---------------------------------------------*/
  a {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 14px;
    line-height: 1.7;
    color: #666666;
    margin: 0px;
    transition: all 0.4s;
    -webkit-transition: all 0.4s;
    -o-transition: all 0.4s;
    -moz-transition: all 0.4s;
  }

  a:focus {
    outline: none !important;
  }

  a:hover {
    text-decoration: none;
  }

  /*---------------------------------------------*/
  h1,h2,h3,h4,h5,h6 {
    margin: 0px;
  }

  p {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 14px;
    line-height: 1.7;
    color: #666666;
    margin: 0px;
  }

  ul, li {
    margin: 0px;
    list-style-type: none;
  }


  /*---------------------------------------------*/
  input {
    outline: none;
    border: none;
  }

  input[type="number"] {
    -moz-appearance: textfield;
    appearance: none;
    -webkit-appearance: none;
  }

  input[type="number"]::-webkit-outer-spin-button,
  input[type="number"]::-webkit-inner-spin-button {
    -webkit-appearance: none;
  }

  textarea {
    outline: none;
    border: none;
  }

  textarea:focus, input:focus {
    border-color: transparent !important;
  }

  input:focus::-webkit-input-placeholder { color:transparent; }
  input:focus:-moz-placeholder { color:transparent; }
  input:focus::-moz-placeholder { color:transparent; }
  input:focus:-ms-input-placeholder { color:transparent; }

  textarea:focus::-webkit-input-placeholder { color:transparent; }
  textarea:focus:-moz-placeholder { color:transparent; }
  textarea:focus::-moz-placeholder { color:transparent; }
  textarea:focus:-ms-input-placeholder { color:transparent; }

  input::-webkit-input-placeholder { color: #adadad;}
  input:-moz-placeholder { color: #adadad;}
  input::-moz-placeholder { color: #adadad;}
  input:-ms-input-placeholder { color: #adadad;}

  textarea::-webkit-input-placeholder { color: #adadad;}
  textarea:-moz-placeholder { color: #adadad;}
  textarea::-moz-placeholder { color: #adadad;}
  textarea:-ms-input-placeholder { color: #adadad;}

  /*---------------------------------------------*/
  button {
    outline: none !important;
    border: none;
    background: transparent;
  }

  button:hover {
    cursor: pointer;
  }

  iframe {
    border: none !important;
  }


  /*---------------------------------------------*/
  .container {
    max-width: 1200px;
  }


  /*//////////////////////////////////////////////////////////////////
  [ Utility ]*/

  .bg0 {background-color: #fff;}
  .bg1 {background-color: #f7f7f7;}


  /*//////////////////////////////////////////////////////////////////
  [ Contact ]*/

  .container-contact100 {
    width: 100%;
    min-height: 100vh;
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    padding: 15px;
    background: #e6e6e6;

  }

  .wrap-contact100 {
    width: 920px;
    background: #fff;
    border-radius: 10px;
    overflow: hidden;
    padding: 62px 55px 90px 55px;
  }


  /*------------------------------------------------------------------
  [  ]*/

  .contact100-form {
    width: 100%;
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  .contact100-form-title {
    display: block;
    width: 100%;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 39px;
    color: #333333;
    line-height: 1.2;
    text-align: center;
    padding-bottom: 59px;
  }



  /*------------------------------------------------------------------
  [  ]*/

  .wrap-input100 {
    width: 100%;
    position: relative;
    border: 1px solid #e6e6e6;
    border-radius: 13px;
    padding: 5px 30px 9px 22px;
    margin-bottom: 20px;
  }

  .rs1-wrap-input100 {
    width: calc((100% - 30px) / 2);
  }

  .label-input100 {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 10px;
    color: #393939;
    line-height: 1.5;
    text-transform: uppercase;
  }

  .label-input200 {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 16px;
    color: #393939;
    line-height: 1.5;
    text-transform: uppercase;
  }

  .input100 {
    display: block;
    width: 100%;
    background: transparent;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 18px;
    color: #555555;
    line-height: 1.2;
    padding-right: 15px;
  }


  /*---------------------------------------------*/
  input.input100 {
    height: 40px;
  }


  textarea.input100 {
    min-height: 120px;
    padding-top: 9px;
    padding-bottom: 13px;
  }


  .input100:focus + .focus-input100::before {
    width: 100%;
  }

  .has-val.input100 + .focus-input100::before {
    width: 100%;
  }


  /*------------------------------------------------------------------
  [ Button ]*/
  .container-contact100-form-btn {
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    padding-top: 20px;
    width: 100%;
  }

  .contact100-form-btn {
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0 20px;
    width: 100%;
    height: 50px;
    background-color: #333333;
    border-radius: 25px;

    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 16px;
    color: #fff;
    line-height: 1.2;

    -webkit-transition: all 0.4s;
    -o-transition: all 0.4s;
    -moz-transition: all 0.4s;
    transition: all 0.4s;
  }

  .contact100-form-btn i {
    -webkit-transition: all 0.4s;
    -o-transition: all 0.4s;
    -moz-transition: all 0.4s;
    transition: all 0.4s;
  }

  /*.contact100-form-btn:hover {*/
  /*  background-color: #00ad5f;*/
  /*}*/

  /*.contact100-form-btn:hover i {*/
  /*  -webkit-transform: translateX(10px);*/
  /*  -moz-transform: translateX(10px);*/
  /*  -ms-transform: translateX(10px);*/
  /*  -o-transform: translateX(10px);*/
  /*  transform: translateX(10px);*/
  /*}*/

  /*---------------------*/

  .container-contact100-form-btn {
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    padding-top: 20px;
    width: 100%;
    margin-bottom: 10px;
  }

  .contact100-form-btn {
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0 20px;
    width: 100%;
    height: 50px;
    background-color: #FF7A59;
    border-radius: 25px;

    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 16px;
    color: #fff;
    line-height: 1.2;

    -webkit-transition: all 0.4s;
    -o-transition: all 0.4s;
    -moz-transition: all 0.4s;
    transition: all 0.4s;
  }

  .contact100-form-btn i {
    -webkit-transition: all 0.4s;
    -o-transition: all 0.4s;
    -moz-transition: all 0.4s;
    transition: all 0.4s;
  }

  /*.contact100-form-btn:hover {*/
  /*  background-color: #00ad5f;*/
  /*}*/

  .width-50 {
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0 20px;
    height: 40px;
    border-radius: 25px;

    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    font-size: 16px;
    color: #fff;
    line-height: 1.2;

    -webkit-transition: all 0.4s;
    -o-transition: all 0.4s;
    -moz-transition: all 0.4s;
    transition: all 0.4s;
    width: 14%;
    /*background-color: #FF7A59;*/
    background-color: #818182;
    margin-right: 5px;
  }


  /*------------------------------------------------------------------
  [ Responsive ]*/

  @media (max-width: 768px) {
    .rs1-wrap-input100 {
      width: 100%;
    }

  }

  @media (max-width: 576px) {
    .wrap-contact100 {
      padding: 62px 15px 90px 15px;
    }

    .wrap-input100 {
      padding: 5px 10px 9px 10px;
    }
  }



  /*------------------------------------------------------------------
  [ Alert validate ]*/

  .validate-input {
    position: relative;
  }

  /*---------------------------------------------*/
  @media (max-width: 576px) {
    .alert-validate::before {
      padding: 0 10px 0 10px;
    }

    .true-validate::after,
    .btn-hide-validate {
      right: 0px;
      width: 30px;
    }
  }


  /*==================================================================
  [ Restyle Select2 ]*/

  .select2-container {
    display: block;
    max-width: 100% !important;
    width: auto !important;
  }

  .select2-container .select2-selection--single {
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    align-items: center;
    background-color: transparent;
    border: none;
    height: 40px;
    outline: none;
    position: relative;
  }

  .wrap-contact100-form-range {
    width: 100%;
    padding: 20px 25px 25px 25px;
  }

  .contact100-form-range-value input {
    display: none;
  }

  #filter-bar {
    height: 20px;
    border: 1px solid #e6e6e6;
    border-radius: 9px;
    background-color: #f7f7f7;
  }
  #filter-bar .noUi-connect {
    border: 1px solid #e6e6e6;
    border-radius: 9px;
    background-color: #00ad5f;
    box-shadow: none;
  }
  #filter-bar .noUi-handle {
    width: 40px;
    height: 36px;
    border: 1px solid #cccccc;
    border-radius: 9px;
    background: #f5f5f5;
    cursor: pointer;
    box-shadow: none;
    outline: none;
    top: -8px;
    display: -webkit-box;
    display: -webkit-flex;
    display: -moz-box;
    display: -ms-flexbox;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #filter-bar .noUi-handle.noUi-handle-lower {
    left: -1px;
  }

  #filter-bar .noUi-handle.noUi-handle-upper {
    left: -39px;
  }

  #filter-bar .noUi-handle:before {
    content: "";
    display: block;
    position: unset;
    height: 12px;
    width: 9px;
    background-color: transparent;
    border-left: 2px solid #cccccc;
    border-right: 2px solid #cccccc;
  }
  #filter-bar .noUi-handle:after {
    display: none;
  }

  @media (max-width: 576px) {
    .wrap-contact100-form-range {
      padding: 20px 0px 57px 0px;
    }

    .wrap-contact100-form-radio {
      padding: 15px 0px 0 0px;
    }
  }

  .alert {
    border: 2px solid #e6e6e6;
    margin-bottom: 15px;
    padding: 7px;
  }

  .seperator {
    margin-top: 1rem;
    margin-bottom: 1rem;
    border: 0;
    border-top: 3px solid rgba(0, 0, 0, 0.1);
  }

  .font-weight {
    font-weight: 600;
  }

  dt {
    font-weight: 600;
  }

  .smaller {
    font-size: 13px
  }

  .display-name {
    margin-bottom: 2px;
    margin-top: 2px;
  }

  h4 {
    font-weight: bold;
    color: #FF7A59;
    /*color: #c52f00;*/
  }

  .title {
    color: #818182;
    /*color: #c52f00;*/
    font-size: 1.2rem;
  }

  .logo {
    width: 25%;
  }
</style>
