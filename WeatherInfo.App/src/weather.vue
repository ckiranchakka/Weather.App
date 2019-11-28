<template>
<div id="weather">
    <form @submit.prevent="getWeather(inputValue)" class="form">
        <div class="radioinline">
            <label for="City">City</label>
            <input type="radio" id="cityid" value="City" v-model="selecttype" />
            <span class="checkmark"></span>
        </div>
        <div class="radioinline">
        <label for="ZipCode">ZipCode</label>
        <input type="radio" id="zipcodeid" value="ZipCode" v-model="selecttype" />
        <span class="checkmark"></span>
        </div>
        <br />
        <input type="text" class="autosearchfield" @keyup="filterAutoComplete" v-model="inputValue" placeholder="Enter city/ zipcode,country" />
        <ul class="autoComplete" :class="isShow? 'showDiv' : ''">
            <li v-for="(inputItem, i) in filterData" :key="i">
                <span @click="setItem(inputItem)">{{inputItem}}</span>
            </li>
        </ul>
        <button type="submit" class="autoformbtn">Search</button>
    </form>

    <div v-if="!errorText" id="forecast" class="text-center">
        <div class="container align-center">
            <h3 class="selectedlocation" centered>{{city}} {{ country}}</h3>
            <br />
            <div class="row" style="centered">
                <div v-for="item in weatherData" :key="item.date">
                    <div v-if="item.temperature" class="card bg-light text-dark col-sm" style="height: 300px;width:150px">
                        <h5 class="card-text">{{item.day}}</h5>
                        <h5 class="card-text">{{item.date}}</h5>
                        <h5 class="card-text">{{item.description}}</h5>
                        <img :src="item.icon" v-bind:alt="item.icon" />
                        <h5 class="card-text">{{item.temperature.celsiusCurrent}} &#8451;</h5>
                        <h5 class="card-text">{{item.pressure}} hpa</h5>
                        <h5 class="card-text">{{item.humdity}}%</h5>
                        <h5 class="card-text">{{item.speed}} kn</h5>
                        <!-- <h5 class="card-text">{{item.weatherType}}</h5> -->
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div v-if="!errorText" id="chart">
        <div class="container">
            <div class="my-5">
                <div v-show="chart != null">
                    <canvas id="myChart"></canvas>
                </div>
            </div>
        </div>
    </div>
    <div v-if="displaygrid" id="grid">
        <div class="table-responsive">
            <table class="table table-bordered">
                <thead>
                    <tr>
                        <th>DAY</th>
                        <th>DESCRIPTION</th>
                        <th>HIGH/LOW</th>
                        <th>WIND</th>
                        <th>HUMIDITY</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in weatherData" :key="item.date">
                        <td>
                            <div class="d_info">
                                <span>{{item.day}}</span><br>
                                <span>{{item.date}}</span>
                            </div>
                            <div class="w_icon">
                                <span>
                                    <img :src="item.icon" />
                                </span>
                                
                            </div>
                        </td>
                        <td>
                            <div>
                                <span>{{item.description}}</span>
                            </div>
                        </td>
                        <td>
                            <div>
                                <span>{{item.temperature.celsiusCurrent}}</span>
                            </div>
                        </td>
                        <td>
                            <div>
                                <span>{{item.humdity}}%</span>
                            </div>
                        </td>
                        <td>
                            <div>
                                <span>{{item.speed}} km/h</span>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</div>
</template>

<script>
import axios from "axios"
import "chart.min.js"
export default {
    name: "weather",
    data: function () {
        return {
            weatherData: [],
            dates: [],
            temps: [],
            inputValue: "",
            errorText: "",
            country: "",
            displaygrid: false,
            selecttype: "City",
            selected: "",
            city: "",
            chart: null,
            searchedData: [],
            inputString: "",
            isShow: false,
            searchOptions: [{
                    SearchName: "City",
                    SearchId: "1"
                },
                {
                    SearchName: "Zipcode ",
                    SearchId: "2"
                }
            ]
        }
    },
    computed: {
        filterData() {
            if (this.inputString != "") {
                this.isShow = true
                return this.searchedData.filter(item => {
                    return item.indexOf(this.inputString) !== -1
                })
            } else {
                this.isShow = false
                return this.searchedData
            }
        }
    },
    methods: {
        setItem(inputItem) {
            this.inputValue = inputItem
            this.isShow = false
        },
        getWeather: function (input) {
            this.inputValue = ''
            if (this.searchedData.indexOf(input) == -1) this.searchedData.push(input)

            var url = ""
            if (this.selecttype == "City")
                url =
                "http://localhost:49178/api/weatherforecast/search?City=" +
                input +
                "&isZipCode=false"
            else if (this.selecttype == "ZipCode")
                url =
                "http://localhost:49178/api/weatherforecast/search?ZipCode=" +
                input +
                "&isZipCode=true"
            axios
                .get(url)
                .then(response => {
                    this.responseCode = undefined
                    if (this.response === null) {
                        this.errorText = "Not Found"
                    } else {
                        this.displaygrid = true
                        this.weatherData = response.data.forecasts
                        this.city = response.data.city
                        this.country = " - " + response.data.country

                        this.dates = response.data.forecasts.map(list => {
                            return list.Date_time
                        })
                        this.temps = response.data.forecasts.map(list => {
                            return list.temperature.celsiusCurrent
                        })
                        //Chart Start

                        //Chart End
                    }
                })
                .catch(error => {
                    if (error) {
                        this.errorText = "Error in finding this city!"
                    }
                })
        },
        filterAutoComplete($event) {
            this.inputString = $event.srcElement.value
            /*if (this.searchedData.length == 0) {
                            this.displaySearchedInputArr = this.searchedData
                        } else {
                            this.displaySearchedInputArr = this.searchedData.filter(item => {
                                return item.indexOf(val) !== -1
                            })
                        }*/
        }
    }
}
</script>

<style scoped>
.searchbar {
    width: 500px;
}

@media (min-width: 1200px) {
    .container {
        max-width: 900px;
    }
}

form.form {
    position: relative;
    background-color: #e2e2e2;
    padding: 30px 10px;
}

.autosearchfield {
    width: 500px;
    padding: 9px 10px;
    border: 1px solid #b5b5b5;
}

.autoformbtn {
    background-color: #ff4700;
    color: #fff;
    padding: 10px 17px;
    border: 0;
    text-transform: uppercase;
    box-shadow: none;
}

.autoComplete {
    list-style-type: none;
    padding: 0;
    margin: 0;
    position: absolute;
    z-index: 9;
    background-color: #fff;
    border: 1px solid #ccc;
    width: 599px;
    top: 120px;
    left: 0;
    right: 0;
    margin: 0 auto;
    display: none;
}

.autoComplete li {
    text-align: left;
}

.autoComplete li span {
    padding: 10px;
    display: block;
    border-bottom: 1px solid #ccc;
}

.autoComplete li span:hover {
    background-color: #f4f4f4;
}

.showDiv {
    display: block !important;
}

#grid {
    width: 900px;
    position: absolute;
    margin: 0 auto;
    left: 0;
    right: 0;
}

.bg-light {
    background-color: #c0cdec !important;
}

.card {
    border: 1px solid #8ea7c1;
    border-radius: 0px;
    border-left: 0px;
    padding: 10px;
}

h5 {
    font-size: 15px;
}

.selectedlocation {
    display: inline-block;
    background: #585858;
    color: #fff;
    padding: 5px 15px;
    font-size: 20px;
    margin-bottom: 20px;
}


.radioinline {
    display: inline-block;
    position: relative;
    width: 96px;
    text-align: left;
    margin-bottom: 20px;
    padding-left: 35px;
    cursor: pointer;
}
.radioinline label {
    /* margin-left: 32px; */
    font-size: 18px;
    margin-bottom: 0;
}
/* Hide the browser's default radio button */
.radioinline input {
    position: absolute;
    opacity: 0;
    cursor: pointer;
    left: 0;
    width: 100%;
    height: 25px;
    top: 0;
    z-index: 1;
}

/* Create a custom radio button */
.checkmark {
    position: absolute;
    top: 0;
    left: 0;
    height: 25px;
    width: 25px;
    background-color: #adadad;
    border-radius: 50%;
}

/* On mouse-over, add a grey background color */
.radioinline:hover input~.checkmark {
    background-color: #ccc;
}

/* When the radio button is checked, add a blue background */
.radioinline input:checked~.checkmark {
    background-color: #2196F3;
}

/* Create the indicator (the dot/circle - hidden when not checked) */
.checkmark:after {
    content: "";
    position: absolute;
    display: none;
}

/* Show the indicator (dot/circle) when checked */
.radioinline input:checked~.checkmark:after {
    display: block;
}

/* Style the indicator (dot/circle) */
.radioinline .checkmark:after {
    top: 9px;
    left: 9px;
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: white;
}

.table-bordered th, .table-bordered td
{
    vertical-align: middle;
}
.d_info,
.w_icon{
    display: inline-block;
    vertical-align: middle;
}
</style>
