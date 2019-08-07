<template>
  <div class="container-contact100">
    <div class="wrap-contact100">
      <div class="hello container">
        <span class="contact100-form-title">{{ msg }}</span>

      <!-- <form>
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
      </form>-->

      <form class="contact100-form">
        <div
          class="wrap-input100 validate-input bg1 rs1-wrap-input100"
          data-validate="Please enter Job Title"
        >
          <span class="label-input100">Job Title</span>
          <input
            class="input100"
            type="text"
            name="jobTitle"
            id="jobTitle"
            placeholder="Enter Job Title"
            v-model="searchInput.jobTitle"
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
          />
        </div>
        <div class="wrap-contact100-form-range">
          <label for="filter-bar" class="label-input200">
            Confidence Score Range
            <span class="badge badge-primary">{{searchInput.score}}</span>
          </label>
          <div class="contact100-form-range-bar">
            <input
              type="range"
              class="form-control-range contact100-form-range-bar"
              id="filter-bar"
              v-model="searchInput.score"
            />
          </div>
        </div>
        <div class="container-contact100-form-btn">
          <button class="contact100-form-btn" type="button" v-on:click="mapSearch">
            <span>Search</span>
          </button>
        </div>
      </form>

      <div class="row">
        <button type="button" class="contact100-form-btn width-50" v-on:click="activateListView">List View</button>
        <button type="button" class="contact100-form-btn width-50" v-on:click="activateMapView">Map View</button>
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

.badge {
  padding: 3px;
  margin-left: 3px;
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

.contact100-form-btn:hover {
  background-color: #00ad5f;
}

.contact100-form-btn:hover i {
  -webkit-transform: translateX(10px);
  -moz-transform: translateX(10px);
  -ms-transform: translateX(10px);
  -o-transform: translateX(10px);
  transform: translateX(10px);
}

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

.contact100-form-btn:hover {
  background-color: #00ad5f;
}

.contact100-form-btn:hover i {
  -webkit-transform: translateX(10px);
  -moz-transform: translateX(10px);
  -ms-transform: translateX(10px);
  -o-transform: translateX(10px);
  transform: translateX(10px);
}

.width-50 {
  width: 15%;
  background-color: #00ad5f;
  margin-right: 5px;
}

.width-50:hover i {
  -webkit-transform: translateX(10px);
  -moz-transform: translateX(10px);
  -ms-transform: translateX(10px);
  -o-transform: translateX(10px);
  transform: translateX(10px);
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

/*------------------------------------------------------------------
[ in select ]*/
.select2-container .select2-selection--single .select2-selection__rendered {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  font-size: 18px;
  color: #555555;
  line-height: 1.2;
  padding-left: 0px ;
  background-color: transparent;
}

.select2-container--default .select2-selection--single .select2-selection__arrow {
  height: 100%;
  top: 50%;
  transform: translateY(-50%);
  right: 0px;
  display: -webkit-box;
  display: -webkit-flex;
  display: -moz-box;
  display: -ms-flexbox;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

.select2-selection__arrow b {
  display: none;
}

.select2-selection__arrow::before {
  content: '\f312';
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  font-size: 18px;
  color: #555555;
}


/*------------------------------------------------------------------
[ Dropdown option ]*/
.select2-container--open .select2-dropdown {
  z-index: 1251;
  width: calc(100% + 2px);
  border: 0px solid transparent;
  border-radius: 10px;
  overflow: hidden;
  background-color: white;
  left: -24px;

  box-shadow: 0 3px 10px 0px rgba(0, 0, 0, 0.2);
  -moz-box-shadow: 0 3px 10px 0px rgba(0, 0, 0, 0.2);
  -webkit-box-shadow: 0 3px 10px 0px rgba(0, 0, 0, 0.2);
  -o-box-shadow: 0 3px 10px 0px rgba(0, 0, 0, 0.2);
  -ms-box-shadow: 0 3px 10px 0px rgba(0, 0, 0, 0.2);
}

@media (max-width: 576px) {
  .select2-container--open .select2-dropdown {
    left: -12px;
  }
}

.select2-dropdown--above {top: -38px;}
.select2-dropdown--below {top: 10px;}

.select2-container .select2-results__option[aria-selected] {
  padding-top: 10px;
  padding-bottom: 10px;
  padding-left: 24px;
}

@media (max-width: 576px) {
  .select2-container .select2-results__option[aria-selected] {
    padding-left: 12px;
  }
}

.select2-container .select2-results__option[aria-selected="true"] {
  background: #00ad5f;
  color: white;
}

.select2-container .select2-results__option--highlighted[aria-selected] {
  background: #00ad5f;
  color: white;
}

.select2-results__options {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  font-size: 14px;
  color: #555555;
  line-height: 1.2;
}

.select2-search--dropdown .select2-search__field {
  border: 1px solid #aaa;
  outline: none;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  font-size: 15px;
  color: #333333;
  line-height: 1.2;
}

.wrap-input100 .dropDownSelect2 .select2-container--open {
  width: 100% !important;
}

.wrap-input100 .dropDownSelect2 .select2-dropdown {
  width: calc(100% + 2px) !important;
}

/*==================================================================

.label-radio100::after {
  content: "";
  display: block;
  position: absolute;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 6px solid transparent;
  background: #00ad5f;
  -moz-background-clip: padding;
  -webkit-background-clip: padding;
  background-clip: padding-box;
  left: 0;
  top: 50%;
  -webkit-transform: translateY(-50%);
  -moz-transform: translateY(-50%);
  -ms-transform: translateY(-50%);
  -o-transform: translateY(-50%);
  transform: translateY(-50%);
  display: none;

}

.input-radio100:checked + .label-radio100::after {
  display: block;
}


/*==================================================================
[ rs NoUI ]*/
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

h4 {
  color: #00ad5f;
}
</style>
