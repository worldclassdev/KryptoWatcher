<template>

  <div>
  <intro></intro>
  <div id="body"> 
  <h2> </h2>
  <div id="current"> <current v-bind:currentCurrency="currentCurrency"> </current>  </div>
  <div id="previous"> <previous v-bind:previousCurrency="previousCurrency"> </previous>  </div>
  </div>

  </div>

</template>

<script>

import Intro from './components/Intro.vue';
import Current from './components/Current.vue';
import Previous from './components/Previous.vue';
import Pusher from 'pusher-js'

export default {

  name: 'app',

  components: {
    current: Current,
    previous: Previous,
    intro:Intro
  },

  data () {

    return {

      currentCurrency: {
      BTC : '',
      ETH : '',
      LTC : '',
      },

      previousCurrency:[]
    }
  },

  methods: {

getCurrencyHistory: function(days) {
    var self = this;
  
    for ( var i=1; i>=days ; i++ ) {

       
       var date = this.$moment().subtract(i, 'days').unix(); 
     
   

    this.axios.all([
      
       this.axios.get('https://min-api.cryptocompare.com/data/pricehistorical?fsym=BTC&tsyms=USD&ts=' + date),
       this.axios.get('https://min-api.cryptocompare.com/data/pricehistorical?fsym=ETH&tsyms=USD&ts=' + date),
       this.axios.get('https://min-api.cryptocompare.com/data/pricehistorical?fsym=LTC&tsyms=USD&ts=' + date)

    ]).then(this.axios.spread((BTC, ETH, LTC) => {

                let temp  = [... self.previousCurrency,
                {
                    DATE: this.$moment.unix(date).format("MMMM Do YYYY"),
                    BTC: BTC.data.BTC.USD,
                    ETH: ETH.data.ETH.USD,
                    LTC: LTC.data.LTC.USD
                }]

                self.previousCurrency = temp;
                localStorage.setItem(this.$moment.unix(date).format("MMMM Do YYYY"), JSON.stringify(temp));

            }))
      
  }
  console.log(self.previousCurrency);

  },


    sendPricePusher (data) {
        this.axios.post('/api/prices/update', {
            update: data
        })
            .then(response => {
                console.log(response)
            })
            .catch(error => {
                console.log(error)
            })
    }

  },

 created(){

       if (!navigator.onLine) {

        this.currentCurrency.BTC = localStorage.getItem('BTC');
        this.currentCurrency.ETH = localStorage.getItem('ETH');
        this.currentCurrency.LTC = localStorage.getItem('LTC');
        
        for (var i=1; i<=30;i++){
        var date = this.$moment().subtract(30, 'days').unix(); 
        offlineData = [...this.previousCurrency,
         JSON.parse(localStorage.getItem(this.$moment.unix(date).format("MMMM Do YYYY")))]
        }

        
    }

    this.pusher = new Pusher('92328e0f11b44cfe765f', {
      cluster: 'eu',
      encrypted: true
    });

   this.prices = this.pusher.subscribe('price-updates');

    this.getCurrencyHistory(30);

    this.axios.get('https://min-api.cryptocompare.com/data/pricemulti?fsyms=BTC,ETH,LTC&tsyms=USD').then((response) => {

    this.currentCurrency.BTC = response.data.BTC.USD,
    localStorage.setItem('BTC', response.data.BTC.USD),

    this.currentCurrency.ETH = response.data.ETH.USD,
    localStorage.setItem('ETH', response.data.ETH.USD),

    this.currentCurrency.LTC = response.data.LTC.USD,
    localStorage.setItem('LTC', response.data.LTC.USD)
})

  },

  mounted(){
    
       setInterval(() => {
            this.axios.get('https://min-api.cryptocompare.com/data/pricemulti?fsyms=BTC,ETH,LTC&tsyms=USD')
                .then(response => {
                    this.sendPricePusher (response.data)
                })
                .catch(error => {
                    console.log(error)
                })
        }, 5000);


  this.prices.bind('coin-updates', update => {
            this.currentCurrency.BTC =  update.update.BTC.USD
            this.currentCurrency.ETH =  update.update.ETH.USD
            this.currentCurrency.LTC =  update.update.LTC.USD
        });

  }

}


</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Lato');

/* ==================== General Styles ========================== */

* {
  margin : 0px;
  padding : 0px;
 /* box-sizing: border-box; */
  font-family: 'Lato', sans-serif;
  color: #ffffff;
}
h1,h2,h3,h3,h5,h6{

}

body{
  height: 100vh;
  width: 100%;
}
.row{
	display: flex;
	flex-wrap: wrap;
}
h1{
	font-size: 48px;
}
a{
	color: #FFFFFF;
	text-decoration: none;
}
a:hover{
	color: #FFFFFF;
}
a:visited{
	color: #000000;
}
.button{
margin: auto;
width: 200px;
height: 60px;
border: 2px solid #E36F55;
box-sizing: border-box;
border-radius: 30px;

}

#body {
  max-width: 90%;
  margin: 0 auto;
  padding: 1.5em;
  text-align: center;
  color:rgb(0, 193, 131);
}


#current {
  padding: 2em 0em;
}

#previous {
  padding: 2em 0em;
}
</style>
