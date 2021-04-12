<template>
  <div>
    <Search v-model="input" :games="games" v-on:keyup="filtered" />
    <div class="data">
      <div class="games-wrapper" v-for="item in filtered" v-bind:key="item.dealID">
        <div class="rating">
          <i class="fas fa-star-half-alt"> {{ item.dealRating }}</i>
        </div>
        <div class="play-buy"> <i @click="playMessage" class="fas fa-play-circle"></i> 
          <button @click="addCart(item)">Add To Cart</button>
        </div>
        <div class="text-rating">
          <h5>
            <span>Steam Rating</span>: <br />
            {{
              item.steamRatingText == null ? "Unknown" : item.steamRatingText
            }}
          </h5>
          <h5>
            <span>Rating Percent </span> <br />
            {{
              item.steamRatingPercent.includes("0")
                ? "Unknown"
                : item.steamRatingPercent + "%"
            }}
          </h5>
        </div>

        <span class="storeID"> Store ID: {{ item.storeID }}</span>

        <ul>
            <img class="thumb" :src="item.thumb" />
          <li>{{ item.title }}</li>
          <li>
            <ul class="bottom-info">
              <li>Price: {{ item.normalPrice }}$</li>
              <li>Sale: {{ item.salePrice }}$</li>
            </ul>
          </li>
        </ul>
      </div>
      <!-- cart  -->
        <button @click="openCart" class="cart-btn">Cart <sup>{{this.cart.length}}</sup></button>
        <!-- cart popup -->
        <transition  name="slide-fade"> 
          <div class="cart-popup" v-if="showModal" >
            <div class="cart-content-wrapper">
              <div class="cart-games" v-for="cartGames in cart" :key="cartGames.dealID">
                <table>
                  <tbody>
                    <tr>
                      <td><img :src="cartGames.thumb"></td>
                      <td> Game Title: <br> {{cartGames.title}}</td>
                      <td> Price: {{cartGames.normalPrice}}$</td>
                      <td> <button @click="delCartGame(cartGames.gameID)" class="del-cart-game">Delete</button>  </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <div class="total">
                  <h2> Total Price : {{cartTotal}}$</h2>
                  <button> Buy </button> 
              </div>
              <button class="cart-close" @click="showModal = false">Close</button>
            </div>
          </div>
        </transition> 
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
import Search from "./Search";
import loading from 'vuejs-loading-screen';
import VueSimpleAlert from "vue-simple-alert";
import Toast from "vue-toastification";
import "vue-toastification/dist/index.css";

Vue.use(Toast, {
  transition: "Vue-Toastification__bounce",
  maxToasts: 20,
  newestOnTop: true
});

Vue.use(VueSimpleAlert);
Vue.use(loading)
Vue.use(VueAxios, axios);

export default {
  name: "Data",
  components: {
    Search,
  },
  data() {
    return {
      games: [],
      cart: [],
      input: '',
      showModal: false,
      cartTotal: 0,
    };
  },

  methods: {
      loading() {
          this.$isLoading(true)
           Vue.axios.get("https://www.cheapshark.com/api/1.0/deals").then((res) => {
              this.$isLoading(false)
              console.log(res) 
          })
      },
     playMessage(){
       this.$fire({
         title: "This Game Is Not Yet Available To Play",
         type: "warning",
         background: "#fff",
       })
     },
     addCart(item){
       let exist = this.cart.some((game) => {
          return game.title === item.title
        });
        if(!exist){
          document.querySelector('.cart-btn').classList.add('active')
          this.cart.push(item)
          console.log("Games added in cart ",this.cart);
          setTimeout(() => {
            document.querySelector('.cart-btn').classList.remove('active')
          }, 300);

        }else{
          this.$toast.error("This Game Is Already In Your Cart", {
            position: "bottom-right",
            timeout: 2000,
            closeOnClick: true,
            pauseOnFocusLoss: true,
            pauseOnHover: true,
            draggable: true,
            draggablePercent: 2,
            showCloseButtonOnHover: false,
            hideProgressBar: true,
            closeButton: "button",
            icon: true,
            rtl: false
          });
        }
     },
     openCart(){
        if(this.cart.length == 0){
          this.$fire({
            title: "Empty",
            text: "There are no Games in your Cart",
            type: "error",
          })
        }else{
          this.showModal = true;
        }
     },
     delCartGame(id){
       this.cart = this.cart.filter(e => e.gameID !== id)
       if(this.cart.length < 1){
         this.showModal = false
         this.$fire({
           text: 'Cart Is Empty',
           type: 'info'
         })
       }
     }
  },
  mounted() {
    Vue.axios.get("https://www.cheapshark.com/api/1.0/deals").then((res) => {
      const result = res.data;
      this.games = result;
    });
    this.loading();  //Loading
    this.cart = JSON.parse(localStorage.getItem('cartGames')) || [];

  },
   computed: {
    filtered() {
          const trimVal = this.input.toUpperCase().split(' ').join('');
          return this.games.filter((e) => e.internalName.includes(trimVal));
    },
  },

  watch:{
    cart(newGame){
      localStorage.setItem("cartGames", JSON.stringify(newGame));
      this.cartTotal = this.cart.map(e=> Math.round(Number(e.normalPrice))).reduce((a, b) => a + b, 0)
    }
  }
};
</script>


<style lang="scss" scoped>
@mixin shadow() {
  box-shadow: 0px 0px 20px rgba(#ffffff, 0.2);
}

$transition: ease-in-out 0.2s;

.data {
  width: calc(100% - 200px);
  margin: 120px auto;
  line-height: 30px;
  font-weight: 500;
  display: grid;
  grid-template-columns: auto auto auto;
  align-items: center;
  grid-gap: 15px;
  justify-content: center;


  .cart-btn{
    position: fixed;  //cart
    top: 60px;
    right: 0;
    padding: 10px 50px;
    border: none;
    color: #fff;
    background: rgba($color: #fff, $alpha: .3);
    font-size: 20px;
    cursor: pointer;
    transition: ease .2s;
    outline: none;
    &:hover{
      background: palegreen;
      color: rgba(black, .7);
    }
  }

  .cart-btn.active{
    animation: addCartAnimation .3s;
       @keyframes addCartAnimation {
      from{
        transform: scale(1);
      }
      to{
        transform: scale(1.1);
        background: palegreen;
      }
    }
  }

  .slide-fade-enter-active {
    transition: all .3s ease;
  }
  .slide-fade-leave-active {
    transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }
  .slide-fade-enter{
    transform: translateY(-100%);
    opacity: 0;
  }
  .slide-fade-leave-to{
    transform: translateX(-20px);
    opacity: 0;
  }



  .cart-popup{
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    width: 100%;
    height: 100vh;
    background: rgba(black,.6);
    display: flex;
    align-items: center;
    justify-content: center;
    
    & > .cart-content-wrapper{
      width: 900px;
      max-height: 670px;
      background: #fff;
      border-radius: 2px;
      box-shadow: 0px 0px 10px rgba( #000000, .2);
      position: relative;
      overflow: auto;
      &::-webkit-scrollbar{
        background: #fff;
        width: 5px;
      }
      &::-webkit-scrollbar-thumb{
        background: rgba(black,.2);
      }

      .total{
        height: 120px;
        display: flex;
        justify-content: center;
        align-items: center;
        color: rgba(#000000, .6);
        letter-spacing: 1px;
        line-height: 50px;

        & > button{
          border: 2px rgba(black, .1) solid;
          padding: 7px 20px;
          margin-left: 50px;
          text-transform: uppercase;
          color: #fff;
          background: palegreen;
          outline: none;
          cursor: pointer;

        }
      }

      .cart-close{
        padding: 10px 20px;
        color: #fff;
        background: indianred;
        outline: none;
        border: none;
        position: fixed;
        bottom: 100px;
        left: 50%;
        transform: translate(-50%);
        cursor: pointer;
      }

      table{
        width: 100%;
        border-collapse: collapse;

        tbody{
          overflow: auto;

          & > tr,td{
            width: 150px;
            padding: 15px;
            position: relative;
            text-align: center;
            border: 1px rgba(black, .1) solid;
            & > img{
              width: 100px;
            }
          }

          td:last-child{

            & > .del-cart-game{
              outline: none;
              border: none;
              padding: 10px 15px;
              font-size: 15px;
              color: #fff;
              background: lighten($color: red, $amount: 20%);
              cursor: pointer;
            }
          }
        }
      }
    }
  }

  .games-wrapper {
    width: 500px;
    height: 300px;
    padding: 20px 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    @include shadow();
    position: relative;
    transition: $transition;
    background-image: linear-gradient(
      to top,
      #d5d4d0 0%,
      #d5d4d0 1%,
      #eeeeec 40%,
      #efeeec 95%,
      #e9e9e7 100%
    );
    cursor: pointer;
    overflow: hidden;

    &:hover > .play-buy {
      transform: translate(0);
    }

    &:hover > .play-buy i{
        animation: playBtn .8s;
    }

    .rating {
      position: absolute;
      top: 15px;
      left: 15px;
      padding: 10px;
      background: rgba(palegreen, 0.5);
    }
    .play-buy {
      width: 100%;
      height: 100%;
      font-size: 55px;
      position: absolute;
      top: 0;
      left: 0;
      transform: translateX(100%);
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: rgba(black, .5);
      transition: $transition;
      color: #fff;

      & > i{
        @keyframes playBtn {
          from{
            transform: scale(0) translate(50px);
            opacity: 0;
          }
          to{
            transform: scale(1) translate(0);
            opacity: 1;
          }
        }
      }

      & > button{
        outline: none;
        border: none;
        padding: 10px 20px;
        background-color: rgb(138, 236, 138);
        color: #ffffff;
        position: absolute;
        top: 0;
        right: 0;
        cursor: pointer;
        font-size: 15px;
      }
    }

    .text-rating {
      position: absolute;
      bottom: 100px;
      left: 15px;
      h5 {
        font-weight: 400;
        span {
          font-weight: 600;
        }
      }
    }

    .storeID {
      position: absolute;
      bottom: 0;
      right: 0px;
      padding: 3px 10px;
      background: rgb(44, 41, 41);
      color: #fff;
      font-size: 12px;
    }

    ul {
      text-align: center;

      li {
        list-style: none;
        color: rgba($color: #000000, $alpha: 0.6);

        & > .bottom-info {
          li {
            display: inline-block;
            margin-left: 15px;
            padding: 10px;
            font-weight: 600;
          }
        }
      }

      .thumb {
        width: 120px;
      }
    }
  }
}
</style>