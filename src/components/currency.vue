<template>
  <div id="box" class="container">
    <div id="top" class="row">
      <div class="col-lg-12 col-12">
        <p>USD - United States Dollars</p>
      </div>
      <div class="col-lg-6 col-6">
        <p>USD</p>
      </div>
      <div class="col-lg-6 col-6">
        <!-- ammount input -->
        <input id="currVal" type="number" name="" placeholder="10.00" v-on:change="calculate()">
      </div>
    </div>
    <div id="body" class="row">
      <ul class="convert">
        <li v-for="(x , index) in rate" v-bind:key="x.name" v-bind:class="x.class">
          <div class="container">
            <div id="IDR" class="row">
              <div id="left" class="container col-lg-10 col-10">
                <div class="row">
                  <div class="col-lg-6 col-6">
                    <!-- currency naem -->
                    <p>{{ x.name }}</p>
                  </div>
                  <div class="col-lg-6 col-6">
                    <!-- converted currency amount -->
                    <p><span class="finalamount">{{ x.total }}</span></p>
                  </div>
                  <div class="col-lg-12 col-12">
                    <!-- currency -->
                    <p>{{ x.name }} - {{ x.currlong }}</p>
                  </div>
                  <div class="col-lg-12 col-12">
                    <!-- currency exchange rate -->
                    <p>1 USD = {{ x.name }} {{ x.exRate }}</p>
                  </div>
                </div>
              </div>
              <div id="del" class="col-lg-2 col-2">
                <!-- delete currency list -->
                <a v-on:click="removeElement(index)">(-)</a>
              </div>
            </div>
          </div>
        </li>
      </ul>
      <ul id="add" class="add">
        <li class="loading" style="display: none;">load currency rate...</li>
        <li id="plus" class="active">
          <!-- addnew currency -->
          <a v-on:click="addCurr = !addCurr">(+) Add More Currencies</a>
        </li>
        <li id="drop" class="dropdown" v-show="addCurr">
          <div class="container">
            <div class="row">
              <div class="col-lg-9 col-9">
                <!-- currency avaiable list -->
                 <select id="slccurr" v-on:change="othercurr($event)">
                  <option readonly>Choose Currency</option>
                  <option id="optCAD" value="CAD">CAD</option>
                  <option id="optIDR" value="IDR">IDR</option>
                  <option id="optGBP" value="GBP">GBP</option>
                  <option id="optCHF" value="CHF">CHF</option>
                  <option id="optSGD" value="SGD">SGD</option>
                  <option id="optINR" value="INR">INR</option>
                  <option id="optMYR" value="MYR">MYR</option>
                  <option id="optJPY" value="JPY">JPY</option>
                  <option id="optKRW" value="KRW">KRW</option>
                </select>
              </div>
              <div class="col-lg-3 col-3">
                <input id="calc" type="submit" name="Submit">
              </div>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import axios from 'axios';
  var countryarr=[];
  var currtxt;
  export default {
    data(){
      return{
        currency: '',
        addCurr : false,
        rate: [
        ]
      }
    },
    methods: {
      calculate(){
        // triggger re-calculate coversion amount
        this.rates("",1);
      },
      removeElement: function (index) {
        // remove currency from list
        var name = this.rate[index].name;
        this.$delete(this.rate, index);
        this.$delete(countryarr, index);
        
        // add deleted currency back to options
        var sel = document.getElementById('slccurr');
        var opt = document.createElement('option');
        opt.appendChild(document.createTextNode(name));
        opt.id = "opt"+name;
        opt.value = name;
        sel.appendChild(opt);
      },
      othercurr(event) {
        // calculate newly add currency from option
        var value = event.target.value;
        document.getElementById("opt"+value).remove();
        this.addCurr=false;
        this.rates(value,0);
      },
      rates: function(curr,x) {
        if (curr!="") {
          // save coverted currency name 
          countryarr.push(curr);
        }
        var amount = x;
        if (amount > 0) {
          // base amount change, recalculate converted amount
          this.result("",1);
        }else{
          if (curr == 'CAD' || curr == 'IDR' || curr == 'GBP' || curr == 'CHF' || curr == 'SGD' || curr == 'INR' || curr == 'MYR' || curr == 'JPY' || curr == 'KRW') {
            // get currency name
            if (curr == 'CAD') {
              currtxt="Canadian Dollar";
            }else if (curr == 'IDR'){
              currtxt="Indonesian Rupiah";
            }else if (curr == 'GBP'){
              currtxt="British Pound";
            }else if (curr == 'CHF'){
              currtxt="Swiss Franc";
            }else if (curr == 'SGD'){
              currtxt="Singapore Dollar";
            }else if (curr == 'INR'){
              currtxt="Indian Rupee";
            }else if (curr == 'MYR'){
              currtxt="Malaysian Ringgit";
            }else if (curr == 'JPY'){
              currtxt="Japanese Yen";
            }else if (curr == 'KRW'){
              currtxt="Sout Korean Won";
            }else {
              currtxt="No Currencies";
            }
            $('.loading').css({'display':'block'});
            // get exchange rate
            axios.get("https://api.exchangeratesapi.io/latest?base=USD&symbols="+curr)
              .then(response => {
                // sucess
                var html='';
                var rate = response.data.rates;
                var exRate = rate[curr];
                // push response to array
                this.rate.push(
                  {name: curr, exRate: exRate, currlong:currtxt, class:'li'+curr}
                );
                // calculate amount
                this.result(exRate,0);
                $('.loading').css({'display':'none'});
              })
              // catch error
              .catch((error) => console.log(error));
          }
        }
      },
      result: function (x,y) {
        // calculate amount base on exchange rate
        $('.loading').css({'display':'block'});
        var base_amount = $('#currVal').val();
        if (base_amount == "" || base_amount < 1) {
          base_amount=10;
        }
        
        if (y == 0) {// calculate amount from rates method
          var index = this.rate.length - 1;

          // remake array
          var final = base_amount*x.toFixed(4);
          var name = this.rate[index].name;
          var exRate = this.rate[index].exRate;
          var c = this.rate[index].class;
          var currlong = this.rate[index].currlong;

          this.$delete(this.rate, index);
          this.rate.push(
            {name: name, exRate: exRate.toFixed(4), currlong:currlong, class:c, total:final.toFixed(4)}
          );
        }else{// if base amount changed amount calculate here

          // get list of selected currency
          var c = countryarr.toString();
          axios.get("https://api.exchangeratesapi.io/latest?base=USD&symbols="+c)
            .then(response => {
              //loop all selected currency
              for (var i=countryarr.length-1; i>=0; i--) {
                var final="";
                var name="";
                var exRate="";
                var c="";
                var currlong="";

                // remake array
                var rate = response.data.rates;
                var exRate = rate[countryarr[i]];
                final = base_amount*exRate.toFixed(4);
                name = this.rate[i].name;
                c = this.rate[i].class;
                currlong = this.rate[i].currlong;
                this.$delete(this.rate, i);
                this.rate.push(
                  {name: name, exRate: exRate.toFixed(4), currlong:currlong, class:c, total:final.toFixed(4)}
                );
              }
          })
        }
        $('.loading').css({'display':'none'});
      }
    }
  }
</script>

<style scoped>
  #box {
    width: 450px;
    margin: 50px auto;
    border: 1px solid #555;}
  #top {
    padding: 15px;
    border-bottom: 1px solid #555;}
  #body ul.convert,
  #body ul.add {
    width: 100%;
    padding: 15px;}
  #body ul.convert li,
  #body ul.add li {
    list-style: none;
    border: 1px solid #555;}
  #body ul.convert li {
    margin-bottom: 20px;}
  #body ul.convert p,
  #body ul.add p {
    margin: 0;}
  #left {
    padding: 15px;}
  #del {
    border-left: 1px solid #555;
    text-align: center;}
  #del a {
    cursor: pointer;
    line-height: 102px;}
  #plus a {
    cursor: pointer;}
  #drop .row div{
    padding: 0;}
  #slccurr,
  #calc {
    width: 100%;
    height: 100%;
    border: none;
    cursor: pointer;}
  #currVal {
    width: 100%;}
  @media only screen and (max-width: 768px) {
    #box {
      width: auto;}
  }
</style>
